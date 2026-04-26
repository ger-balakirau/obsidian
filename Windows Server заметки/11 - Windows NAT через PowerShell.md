# Windows NAT через PowerShell

Windows умеет создавать NAT для внутренних сетей через PowerShell. Это часто используют для лабораторных стендов, Hyper-V-сетей и тестовых подсетей.

Практический разбор ошибок Hyper-V lab: [[13 - Hyper-V lab сеть - проблемы и решения]].

## Что делает `New-NetNat`

`New-NetNat` создаёт NAT-правило для внутренней подсети.

Пример:

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.0.0.0/8
```

Это значит:

```text
внутренняя сеть: 10.0.0.0/8
NAT name: LabNAT
```

Устройства из этой подсети смогут выходить наружу через Windows-хост, если правильно настроены интерфейсы и маршруты.

## Когда это нужно

- Hyper-V lab.
- Тестовая сеть виртуальных машин.
- Внутренний стенд без прямого выхода в LAN.
- Проверка сетевых сценариев.
- OpenVPN / routing lab.

## Посмотреть существующие NAT

```powershell
Get-NetNat
```

Подробно:

```powershell
Get-NetNat | Format-List *
```

Посмотреть конкретный NAT:

```powershell
Get-NetNat -Name "LabNAT"
```

## Создать NAT

Пример для большой lab-сети:

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.0.0.0/8
```

Более узкий и обычно удобный вариант:

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/24
```

Почему `/24` часто удобнее:

- проще понимать адреса;
- меньше риск пересечься с другой сетью;
- проще диагностировать маршруты.

## Удалить NAT

Подробная отдельная заметка: [[12 - Удаление Windows NAT]].

```powershell
Remove-NetNat -Name "LabNAT"
```

Без подтверждения:

```powershell
Remove-NetNat -Name "LabNAT" -Confirm:$false
```

Удалить все NAT-правила осторожно:

```powershell
Get-NetNat | Remove-NetNat
```

## Изменить NAT

Чаще всего NAT проще удалить и создать заново.

```powershell
Remove-NetNat -Name "LabNAT"
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/24
```

## Проверить IP интерфейсов

```powershell
Get-NetIPConfiguration
```

Посмотреть адаптеры:

```powershell
Get-NetAdapter
```

Посмотреть IP-адреса:

```powershell
Get-NetIPAddress
```

## Пример с Hyper-V Internal Switch

Создать внутренний switch:

```powershell
New-VMSwitch -SwitchName "LabSwitch" -SwitchType Internal
```

Найти интерфейс:

```powershell
Get-NetAdapter | Where-Object Name -Like "*LabSwitch*"
```

Назначить IP на интерфейс хоста:

```powershell
New-NetIPAddress -IPAddress 10.10.0.1 -PrefixLength 24 -InterfaceAlias "vEthernet (LabSwitch)"
```

Создать NAT:

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/24
```

Настройки VM в этой сети:

```text
IP:      10.10.0.10
Mask:    255.255.255.0
Gateway: 10.10.0.1
DNS:     8.8.8.8 или внутренний DNS
```

## Проверка из VM

```cmd
ipconfig /all
ping 10.10.0.1
ping 8.8.8.8
nslookup ya.ru
```

Если `ping 8.8.8.8` работает, а `nslookup` нет - проблема DNS.

## Типовые проблемы

### NAT уже существует

Проверить:

```powershell
Get-NetNat
```

Удалить старый:

```powershell
Remove-NetNat -Name "LabNAT"
```

### Пересечение подсетей

Если `10.0.0.0/8` пересекается с корпоративной сетью или VPN, лучше использовать более узкую подсеть:

```powershell
10.10.0.0/24
```

### Нет gateway в VM

VM должна использовать IP внутреннего интерфейса Windows-хоста как gateway.

Пример:

```text
Gateway: 10.10.0.1
```

Важно: если в lab-сети есть DC, он обычно отвечает за AD/DNS/DHCP, а gateway остаётся на Hyper-V-хосте.

### DNS не работает

Проверить:

```cmd
ping 8.8.8.8
nslookup ya.ru
```

Если IP доступен, а имя нет - укажи DNS.

## Главное запомнить

- `New-NetNat` создаёт NAT для внутренней подсети.
- `Get-NetNat` показывает NAT-правила.
- `Remove-NetNat` удаляет NAT.
- Для лабораторий чаще удобнее `/24`, чем большой `10.0.0.0/8`.
- VM должна иметь gateway на IP внутреннего интерфейса Windows-хоста.
