# Практика по Windows Server под вакансию

Цель практики - подготовиться к реальной работе системного администратора: поднять небольшой домен, настроить пользователей, DNS, DHCP, GPO, файловые шары, обновления, диагностику и базовый порядок обслуживания.

Это не теория. После каждого блока должен быть результат, который можно показать, проверить и объяснить на собеседовании.

## Итоговая схема стенда

```text
Hyper-V host
  vSwitch-LAB
  NAT/gateway: 10.10.0.1

DC1
  Windows Server
  IP: 10.10.0.10
  Roles: AD DS, DNS, DHCP

CLIENT1
  Windows 10/11
  получает IP от DHCP
  введён в домен

FS1 или тот же DC1 для упрощения
  файловые шары
  NTFS-права
```

Связанные заметки:

- [[Windows Server заметки/13 - Hyper-V lab сеть - проблемы и решения]]
- [[Windows Server заметки/02 - Active Directory - базовая модель]]
- [[Windows Server заметки/03 - DNS в Active Directory]]
- [[Windows Server заметки/04 - Пользователи, группы и OU]]
- [[Windows Server заметки/05 - GPO - групповые политики]]
- [[Windows Server заметки/08 - Файловые шары и права доступа]]
- [[Windows Server заметки/15 - Команды Windows админа наизусть]]

## Практика 1 - подготовить lab-сеть

Задача: сделать стабильную внутреннюю сеть Hyper-V, где VM видят хост и могут выходить в интернет через NAT.

Сделать:

1. Создать `Internal` switch `vSwitch-LAB`.
2. Назначить хосту IP `10.10.0.1/23`.
3. Создать NAT `LabNAT` для `10.10.0.0/23`.
4. Подключить будущие VM к `vSwitch-LAB`.

Команды на хосте:

```powershell
New-VMSwitch -SwitchName "vSwitch-LAB" -SwitchType Internal

New-NetIPAddress `
  -InterfaceAlias "vEthernet (vSwitch-LAB)" `
  -IPAddress 10.10.0.1 `
  -PrefixLength 23

New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/23
```

Проверить:

```powershell
Get-NetAdapter | Where-Object Name -Like "*vSwitch-LAB*"
Get-NetIPAddress -InterfaceAlias "vEthernet (vSwitch-LAB)"
Get-NetNat
```

Готово, если:

- `vEthernet (vSwitch-LAB)` имеет `10.10.0.1/23`;
- `LabNAT` активен;
- VM подключены к правильному switch.

## Практика 2 - установить и настроить DC1

Задача: подготовить первый Windows Server как контроллер домена.

Сделать:

1. Установить Windows Server.
2. Задать имя `DC1`.
3. Настроить статический IP.
4. Установить роль AD DS.
5. Создать новый forest, например `lab.local`.
6. Проверить, что DNS поднялся вместе с AD.

Настройки сети DC1:

```text
IP:      10.10.0.10
Mask:    255.255.254.0
Gateway: 10.10.0.1
DNS:     10.10.0.10
```

Проверить:

```cmd
ipconfig /all
nslookup lab.local
nltest /dsgetdc:lab.local
```

Готово, если:

- сервер стал DC;
- домен открывается в `dsa.msc`;
- `nslookup lab.local` отвечает через DC1;
- `nltest` находит контроллер домена.

Что уметь объяснить:

- почему DNS на DC указывает на самого себя;
- почему gateway указывает на хост Hyper-V;
- почему DC не должен быть NAT-шлюзом в этой схеме.

## Практика 3 - создать структуру OU, пользователей и группы

Задача: сделать понятную структуру AD, похожую на маленькую организацию.

Открыть:

```cmd
dsa.msc
```

Создать OU:

```text
Company
  Users
  Computers
  Servers
  Groups
  Disabled
```

Создать пользователей:

```text
ivan.petrov
anna.sidorova
helpdesk.test
```

Создать группы:

```text
GG_File_Accounting_Read
GG_File_Accounting_Modify
GG_RDP_Admins
```

Проверить PowerShell:

```powershell
Get-ADUser ivan.petrov
Get-ADGroup "GG_File_Accounting_Read"
Get-ADGroupMember "GG_File_Accounting_Read"
```

Готово, если:

- пользователи лежат в правильной OU;
- группы названы по смыслу;
- права выдаются через группы, а не напрямую пользователям.

Что уметь объяснить:

- зачем нужны OU;
- чем группа отличается от OU;
- почему права лучше выдавать через группы.

## Практика 4 - DHCP на Windows Server

Задача: настроить выдачу IP-адресов клиентам из домена.

Открыть:

```cmd
dhcpmgmt.msc
```

Создать scope:

```text
Network: 10.10.0.0/23
Range:   10.10.0.100 - 10.10.1.200
Mask:    255.255.254.0
Gateway: 10.10.0.1
DNS:     10.10.0.10
Domain:  lab.local
```

Проверить на клиенте:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

Готово, если CLIENT1 получает:

```text
IP:      10.10.0.x
Gateway: 10.10.0.1
DNS:     10.10.0.10
Suffix:  lab.local
```

Типовая ошибка:

```text
Gateway = 10.10.0.10
```

Так делать не надо, если DC не является маршрутизатором. В этой lab-схеме gateway - хост Hyper-V `10.10.0.1`.

## Практика 5 - DNS в домене

Задача: научиться проверять и чинить базовый DNS.

Открыть:

```cmd
dnsmgmt.msc
```

Сделать:

1. Проверить зону `lab.local`.
2. Создать A-запись `files.lab.local -> 10.10.0.10`.
3. Проверить резолвинг с клиента.
4. Очистить DNS-кэш и проверить снова.

Команды:

```cmd
nslookup lab.local
nslookup dc1.lab.local
nslookup files.lab.local
ipconfig /flushdns
```

Проверить SRV-записи AD:

```cmd
nslookup -type=SRV _ldap._tcp.dc._msdcs.lab.local
```

Готово, если:

- имена домена и DC резолвятся;
- клиент использует DNS DC, а не внешний DNS;
- ты можешь объяснить, почему AD ломается без правильного DNS.

## Практика 6 - ввести клиент в домен

Задача: подключить Windows 10/11 к домену и проверить вход доменным пользователем.

Сделать:

1. Убедиться, что CLIENT1 получает DNS `10.10.0.10`.
2. Ввести CLIENT1 в домен `lab.local`.
3. Перезагрузить клиент.
4. Войти пользователем `lab\ivan.petrov`.

Команды проверки:

```cmd
whoami
whoami /groups
nltest /dsgetdc:lab.local
gpresult /r
```

Готово, если:

- компьютер появился в AD;
- пользователь входит в домен;
- `nltest` на клиенте находит DC;
- `gpresult /r` показывает доменные политики.

## Практика 7 - GPO для рабочей станции

Задача: создать простую групповую политику и проверить её применение.

Открыть:

```cmd
gpmc.msc
```

Сделать GPO:

```text
GPO_Workstations_Baseline
```

Настроить что-то безопасное для тренировки:

- запретить Control Panel для тестового пользователя;
- добавить текстовый логон-баннер;
- настроить firewall rule;
- создать shortcut на рабочем столе;
- подключить сетевой диск через Group Policy Preferences.

Проверить на клиенте:

```cmd
gpupdate /force
gpresult /r
gpresult /h C:\Temp\gpresult.html
```

Готово, если:

- GPO привязана к нужной OU;
- политика применяется только к нужным объектам;
- ты можешь открыть HTML-отчёт и показать, какая GPO применилась.

Что уметь объяснить:

- порядок применения GPO;
- разницу между User Configuration и Computer Configuration;
- почему GPO может не примениться.

## Практика 8 - файловая шара и права

Задача: настроить общий ресурс с нормальной моделью доступа.

Создать папки:

```text
D:\Shares\Accounting
D:\Shares\Common
```

Создать SMB-шару:

```powershell
New-SmbShare -Name "Accounting" -Path "D:\Shares\Accounting"
New-SmbShare -Name "Common" -Path "D:\Shares\Common"
```

Права через группы:

```text
GG_File_Accounting_Read    -> чтение
GG_File_Accounting_Modify  -> изменение
```

Проверить:

```cmd
net use Z: \\dc1\Accounting
net use
whoami /groups
```

PowerShell:

```powershell
Get-SmbShare
Get-SmbSession
Get-SmbOpenFile
```

Готово, если:

- доступ работает через группу;
- пользователь без группы не имеет доступа;
- ты понимаешь разницу между Share permissions и NTFS permissions.

## Практика 9 - Event Viewer и диагностика

Задача: научиться быстро искать причину проблемы.

Открыть:

```cmd
eventvwr.msc
```

Посмотреть:

```text
Windows Logs -> System
Windows Logs -> Application
Applications and Services Logs -> Directory Service
Applications and Services Logs -> DNS Server
```

Команды:

```powershell
Get-WinEvent -LogName System -MaxEvents 30
Get-WinEvent -LogName Application -MaxEvents 30
Get-EventLog -LogName System -EntryType Error -Newest 20
```

Сделать мини-отчёт:

```text
Симптом:
Где смотрел:
Event ID:
Источник:
Что означает:
Что сделал:
Результат:
```

Готово, если ты не просто “видишь красные ошибки”, а можешь связать событие с симптомом.

## Практика 10 - WSUS на уровне понимания

Задача: понять логику обновлений Windows в организации.

Минимум сделать:

1. Изучить, где устанавливается роль WSUS.
2. Понять группы компьютеров.
3. Понять approve/decline updates.
4. Понять, как клиенты направляются на WSUS через GPO.

Что записать в конспект:

```text
Клиенты получают адрес WSUS через GPO.
WSUS скачивает обновления.
Админ одобряет обновления для групп.
Клиенты устанавливают обновления по политике.
```

Команды на клиенте:

```cmd
gpresult /r
gpupdate /force
```

PowerShell:

```powershell
Get-WindowsUpdateLog
```

Готово, если можешь объяснить, почему в компании не всегда дают клиентам обновляться напрямую из интернета.

## Практика 11 - типовые поломки

Специально сломать и починить:

| Поломка | Симптом | Чем проверить |
|---|---|---|
| Неверный DNS на клиенте | Не входит в домен, не ищет DC | `ipconfig /all`, `nslookup`, `nltest` |
| Неверный gateway | Нет интернета | `ping gateway`, `route print` |
| DHCP scope выключен | IP `169.254.x.x` | `ipconfig /all`, DHCP console |
| GPO привязана не к той OU | Политика не применяется | `gpresult /r`, `gpmc.msc` |
| Пользователь не в группе | Нет доступа к шаре | `whoami /groups`, `Get-ADGroupMember` |
| Служба остановлена | Роль не работает | `services.msc`, `Get-Service` |
| DNS-запись старая | Имя ведёт не туда | `nslookup`, `ipconfig /flushdns` |

Правило диагностики:

```text
Сначала IP -> потом gateway -> потом DNS -> потом домен -> потом GPO/права.
```

## Что сказать на собеседовании

Хороший ответ звучит так:

```text
Я поднимал lab на Hyper-V: отдельный Internal switch, NAT на хосте, Windows Server как DC/DNS/DHCP, клиент в домене. Настраивал OU, пользователей, группы, GPO, DHCP scope, DNS-записи и файловые шары. Диагностику делал через ipconfig, nslookup, nltest, gpresult, Event Viewer и PowerShell.
```

Ещё лучше добавить:

```text
Я отдельно проверял типовые поломки: неправильный DNS, неверный gateway, неприменение GPO и отсутствие доступа к SMB-шаре из-за групп.
```

## Минимальный зачёт

Практика считается закрытой, если можешь без подсказок:

- объяснить схему IP/DNS/gateway;
- создать пользователя и группу в AD;
- ввести клиент в домен;
- настроить DHCP scope;
- проверить DNS через `nslookup`;
- применить GPO и проверить через `gpresult`;
- настроить шару через группу;
- посмотреть ошибки в Event Viewer;
- объяснить, почему у клиента нет интернета или не применяется политика.

## Команды, которые нужно закрепить

```cmd
ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /flushdns
ping
tracert
route print
nslookup
whoami /groups
gpupdate /force
gpresult /r
gpresult /h C:\Temp\gpresult.html
nltest /dsgetdc:lab.local
net use
dsa.msc
gpmc.msc
dnsmgmt.msc
dhcpmgmt.msc
eventvwr.msc
services.msc
ncpa.cpl
```

PowerShell:

```powershell
Test-NetConnection server -Port 445
Get-Service
Restart-Service
Get-WinEvent -LogName System -MaxEvents 30
Get-ADUser username
Get-ADGroupMember "Group Name"
Get-SmbShare
Get-NetNat
```
