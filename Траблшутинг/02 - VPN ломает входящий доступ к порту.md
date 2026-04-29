# VPN ломает входящий доступ к порту

Кейс: на Windows-хосте настроен проброс Hyper-V NAT `0.0.0.0:2222 -> 10.10.0.2:22`. При выключенном AmneziaVPN доступ снаружи к `proxy.lab:2222` работает, при включенном VPN доступ пропадает.

## Короткий вывод

Порт `2222` на Windows открыт и принимает соединения. Проблема не в самом SSH/NAT-правиле, а в маршруте ответа.

При включенном AmneziaVPN появился интерфейс `tun2` и маршруты:

```powershell
0.0.0.0/1   -> tun2
128.0.0.0/1 -> tun2
```

Эти два маршрута забирают почти весь интернет-трафик в VPN. Входящий пакет приходит на Windows через обычный интернет/роутер, а ответ Windows пытается отправить через VPN. Из-за асимметричной маршрутизации внешнее подключение ломается.

## Что проверялось

Проверить Hyper-V NAT static mapping:

```powershell
Get-NetNatStaticMapping | Format-Table -AutoSize
```

Ожидаемый важный признак:

```text
ExternalIPAddress ExternalPort InternalIPAddress InternalPort Active
0.0.0.0           2222         10.10.0.2        22           True
```

Проверить старые `netsh portproxy` правила:

```powershell
netsh interface portproxy show all
```

Это нужно, чтобы не перепутать Hyper-V NAT с `portproxy`. В этом кейсе активный проброс на ВМ был именно через `Get-NetNatStaticMapping`.

Проверить, отвечает ли SSH внутри lab-сети:

```powershell
Test-NetConnection 10.10.0.2 -Port 22
```

Важный признак:

```text
TcpTestSucceeded : True
```

Проверить, принимает ли Windows порт `2222` локально:

```powershell
Test-NetConnection 127.0.0.1 -Port 2222
Test-NetConnection 192.168.1.30 -Port 2222
Test-NetConnection 10.33.0.2 -Port 2222
```

В этом кейсе `127.0.0.1:2222`, `192.168.1.30:2222` и `10.33.0.2:2222` отвечали успешно.

Проверить firewall-профили:

```powershell
netsh advfirewall show allprofiles state
netsh advfirewall show allprofiles firewallpolicy
```

В этом кейсе firewall был включен, политика входящих была `BlockInbound`, но существовали разрешающие правила на TCP `2222`.

Проверить разрешающие inbound-правила на порт:

```powershell
$rows = @()
$rules = Get-NetFirewallRule -Enabled True -Direction Inbound -Action Allow |
    Where-Object { $_.DisplayName -match 'Lab Proxy SSH 2222|SSH VM' }

foreach ($r in $rules) {
    $pf = Get-NetFirewallPortFilter -AssociatedNetFirewallRule $r
    $af = Get-NetFirewallAddressFilter -AssociatedNetFirewallRule $r
    $rows += [pscustomobject]@{
        DisplayName   = $r.DisplayName
        Profile       = $r.Profile
        Enabled       = $r.Enabled
        Action        = $r.Action
        Protocol      = $pf.Protocol
        LocalPort     = $pf.LocalPort
        LocalAddress  = $af.LocalAddress
        RemoteAddress = $af.RemoteAddress
    }
}

$rows | Format-List
```

Важный признак:

```text
DisplayName   : Lab Proxy SSH 2222
Profile       : Any
Enabled       : True
Action        : Allow
Protocol      : TCP
LocalPort     : 2222
LocalAddress  : Any
RemoteAddress : Any
```

Проверить IP-адреса интерфейсов:

```powershell
Get-NetIPAddress -AddressFamily IPv4 |
    Where-Object { $_.IPAddress -notlike '169.254.*' } |
    Select-Object InterfaceAlias,IPAddress,PrefixLength |
    Format-Table -AutoSize
```

В этом кейсе были важны:

```text
Ethernet                  192.168.1.30/24
vEthernet (vSwitch-LAB)   10.10.0.1/23
tun2                      10.33.0.2/32
```

Проверить метрики и маршруты:

```powershell
Get-NetIPInterface -AddressFamily IPv4 |
    Sort-Object InterfaceMetric |
    Select-Object InterfaceAlias,InterfaceMetric,ConnectionState |
    Format-Table -AutoSize

Get-NetRoute -AddressFamily IPv4 |
    Sort-Object RouteMetric,InterfaceMetric |
    Select-Object DestinationPrefix,NextHop,RouteMetric,InterfaceMetric,InterfaceAlias |
    Format-Table -AutoSize
```

Ключевая находка:

```text
DestinationPrefix NextHop RouteMetric InterfaceMetric InterfaceAlias
0.0.0.0/1         0.0.0.0 1           5               tun2
128.0.0.0/1       0.0.0.0 1           5               tun2
0.0.0.0/0         192.168.1.1 256     25              Ethernet
```

Это означает, что обычный default route через роутер остался, но более специфичные маршруты `/1` через VPN выигрывают у `0.0.0.0/0`.

## Почему исключения не помогли

Исключения `192.168.1.0/24` и `10.33.0.2/32` не решают доступ снаружи.

`192.168.1.0/24` - это локальная сеть дома/офиса.

`10.33.0.2/32` - это адрес самого VPN-интерфейса `tun2`.

А внешний клиент, который подключается к `proxy.lab:2222`, имеет другой публичный IP. Ответ до этого публичного IP уходит по маршрутам `/1` через VPN, если для него нет отдельного исключения.

## Как исправлять

Если внешний клиент имеет фиксированный IP, добавить маршрут до него через обычный Ethernet-шлюз:

```powershell
New-NetRoute -DestinationPrefix "ВНЕШНИЙ_IP_КЛИЕНТА/32" `
    -InterfaceAlias "Ethernet" `
    -NextHop "192.168.1.1" `
    -RouteMetric 1
```

Пример:

```powershell
New-NetRoute -DestinationPrefix "1.2.3.4/32" `
    -InterfaceAlias "Ethernet" `
    -NextHop "192.168.1.1" `
    -RouteMetric 1
```

Если внешний IP клиента меняется, этот способ неудобен. Тогда лучше:

- включить в AmneziaVPN split tunneling в режиме "через VPN только выбранное";
- подключаться к Windows через ту же VPN/mesh-сеть;
- использовать обратный туннель через VPS;
- использовать Tailscale, ZeroTier, Cloudflare Tunnel, FRP или похожую схему.

## Что указывать в Termius

Termius подключается ровно к тому, что указано в поле `Host`.

Если указать `proxy.lab` и порт `2222`, Termius пойдет на:

```text
proxy.lab:2222
```

Если указать IP и порт `2222`, Termius пойдет на:

```text
IP:2222
```

Важно: `proxy.lab` должен резолвиться на клиентском устройстве в адрес, доступный до Windows-хоста. На самой Windows в момент диагностики `proxy.lab` через публичные DNS не резолвился.

## Мини-чеклист

1. Проверить, что NAT mapping активен: `Get-NetNatStaticMapping`.
2. Проверить, что целевой SSH жив: `Test-NetConnection 10.10.0.2 -Port 22`.
3. Проверить, что Windows принимает `2222`: `Test-NetConnection 192.168.1.30 -Port 2222`.
4. Проверить firewall allow rule на TCP `2222`.
5. Проверить маршруты после включения VPN: `Get-NetRoute -AddressFamily IPv4`.
6. Если есть `0.0.0.0/1` и `128.0.0.0/1` через VPN, проверить маршрут ответа до внешнего IP клиента.
