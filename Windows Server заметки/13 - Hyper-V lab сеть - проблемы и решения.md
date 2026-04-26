# Hyper-V lab сеть - проблемы и решения

Эта памятка по реальной проблеме с lab-сетью Hyper-V: виртуальные машины должны жить во внутренней сети, получать адреса от DHCP/DC и выходить в интернет через NAT на основной машине.

## Правильная схема

```text
Windows-хост Hyper-V
  vEthernet (vSwitch-LAB): 10.10.0.1/23
  NAT: LabNAT -> 10.10.0.0/23

DC / Windows Server
  IP:      10.10.0.10
  Mask:    255.255.254.0
  Gateway: 10.10.0.1
  DNS:     10.10.0.10

Клиенты
  IP:      10.10.0.x
  Gateway: 10.10.0.1
  DNS:     10.10.0.10
```

Главная мысль:

```text
DC = AD + DNS + DHCP
Host = gateway + NAT
```

DC не должен становиться шлюзом, если NAT уже сделан на хосте Hyper-V.

## Что пошло не так

### DC назначили шлюзом

Проблемная логика:

```text
Клиент -> DC 10.10.0.1 -> тупик
```

Если сервер имеет только один интерфейс во внутренней сети и на нём нет RRAS/NAT/второго внешнего интерфейса, он не является шлюзом в интернет.

Правильно:

```text
Клиент -> Host 10.10.0.1 -> NAT -> Internet
```

### NAT был настроен, но DHCP раздавал не тот gateway

Если `Get-NetNat` показывает активный NAT:

```powershell
Get-NetNat
```

и видно:

```text
Name: LabNAT
InternalIPInterfaceAddressPrefix: 10.10.0.0/23
Active: True
```

то проблема часто уже не в NAT, а в настройках VM, DHCP scope или DNS.

В DHCP scope должно быть:

```text
Router / Gateway: 10.10.0.1
DNS:              10.10.0.10
```

## Internal Switch не раздаёт IP сам

`Internal` switch в Hyper-V даёт связь:

```text
Host <-> VM
VM   <-> VM
```

Но он не даёт DHCP автоматически. Если в VM появляется адрес вида `169.254.x.x`, это APIPA: клиент не получил адрес от DHCP.

Проверить на VM:

```cmd
ipconfig /all
```

Что проверить:

- VM подключена именно к `vSwitch-LAB`.
- На DC установлен и включен DHCP.
- DHCP scope активирован.
- DHCP авторизован в AD.
- В scope указан gateway хоста, а не IP DC.

## Ошибка с именем интерфейса

Команда может падать:

```powershell
New-NetIPAddress -InterfaceAlias "vSwitch-LAB" -IPAddress 10.10.0.1 -PrefixLength 23
```

Потому что `InterfaceAlias` - это не имя switch, а имя сетевого адаптера Windows. Обычно оно выглядит так:

```text
vEthernet (vSwitch-LAB)
```

Найти правильное имя:

```powershell
Get-NetAdapter | Where-Object Name -Like "*vSwitch-LAB*"
```

Назначить IP:

```powershell
New-NetIPAddress `
  -InterfaceAlias "vEthernet (vSwitch-LAB)" `
  -IPAddress 10.10.0.1 `
  -PrefixLength 23
```

Проверить:

```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (vSwitch-LAB)"
```

## Базовая настройка с нуля

### 1. Создать Internal switch

```powershell
New-VMSwitch -SwitchName "vSwitch-LAB" -SwitchType Internal
```

### 2. Назначить IP хосту

```powershell
New-NetIPAddress `
  -InterfaceAlias "vEthernet (vSwitch-LAB)" `
  -IPAddress 10.10.0.1 `
  -PrefixLength 23
```

На интерфейсе хоста gateway обычно не нужен:

```text
IP:      10.10.0.1
Mask:    255.255.254.0
Gateway: пусто
DNS:     пусто или нужный DNS для конкретной lab-задачи
```

### 3. Создать NAT на хосте

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/23
```

### 4. Настроить DC

```powershell
New-NetIPAddress `
  -InterfaceAlias "Ethernet" `
  -IPAddress 10.10.0.10 `
  -PrefixLength 23 `
  -DefaultGateway 10.10.0.1

Set-DnsClientServerAddress `
  -InterfaceAlias "Ethernet" `
  -ServerAddresses 10.10.0.10
```

### 5. Настроить DHCP scope

```text
Network: 10.10.0.0/23
Range:   10.10.0.100 - 10.10.1.200
Mask:    255.255.254.0
Gateway: 10.10.0.1
DNS:     10.10.0.10
```

## Проверка по цепочке

На хосте:

```powershell
Get-NetAdapter | Where-Object Name -Like "*vSwitch-LAB*"
Get-NetIPAddress -InterfaceAlias "vEthernet (vSwitch-LAB)"
Get-NetNat
```

На DC:

```cmd
ipconfig /all
route print
ping 10.10.0.1
ping 8.8.8.8
ping google.com
nslookup google.com
```

На клиенте:

```cmd
ipconfig /renew
ipconfig /all
ping 10.10.0.1
ping 10.10.0.10
ping 8.8.8.8
nslookup google.com
```

## Как читать результаты

| Симптом | Вероятная причина | Что делать |
|---|---|---|
| IP `169.254.x.x` | DHCP не ответил | Проверить подключение к switch, DHCP scope, авторизацию DHCP в AD |
| Не пингуется `10.10.0.1` | VM не в том switch или неверная маска/IP | Проверить адаптер VM и адреса |
| Пингуется `10.10.0.1`, но не `8.8.8.8` | Нет NAT или неверный gateway | Проверить `Get-NetNat` и gateway |
| Пингуется `8.8.8.8`, но не имя сайта | DNS-проблема | Проверить DNS в клиенте и на DC |
| `New-NetIPAddress` ругается на alias | Указано имя switch вместо имени адаптера | Использовать `vEthernet (vSwitch-LAB)` |
| Хост теряет сеть после External Switch | Hyper-V забрал физический адаптер | Проверить настройку `Allow management OS to share this network adapter` |

## Когда нужен RRAS

RRAS нужен, если именно Windows Server должен быть маршрутизатором/NAT.

Для простой lab-сети с Hyper-V он обычно лишний:

```text
Host уже делает NAT -> RRAS на DC не нужен
```

RRAS имеет смысл, если у DC или отдельного Windows Server есть две сетевые карты:

```text
LAN: 10.10.0.0/23
WAN: внешняя сеть
```

В учебной AD-лабе чаще чище держать роли отдельно:

```text
Host: NAT/gateway
DC: AD/DNS/DHCP
```

## Быстрый фикс

Если NAT на хосте уже есть, а интернета в VM нет:

1. Убедиться, что на хосте `vEthernet (vSwitch-LAB)` имеет `10.10.0.1/23`.
2. Убедиться, что `Get-NetNat` показывает `LabNAT` для `10.10.0.0/23`.
3. На DC поставить gateway `10.10.0.1`.
4. В DHCP scope поставить gateway `10.10.0.1`.
5. На клиентах выполнить `ipconfig /renew`.
6. Проверить `ping 8.8.8.8`, потом `nslookup google.com`.

## Связанные заметки

- [[11 - Windows NAT через PowerShell]]
- [[12 - Удаление Windows NAT]]
- [[Сети/06 - NAT]]
- [[Сети/04 - DHCP]]
- [[Сети/05 - DNS]]
