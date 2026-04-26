# Команды Windows админа наизусть

Эта памятка - минимум команд, которые системному администратору Windows желательно помнить без поиска. Не для красоты, а чтобы быстро понять: сеть, DNS, домен, политики, службы или права.

## Сеть

### `ipconfig /all`

Показывает IP, маску, gateway, DNS, DHCP, имя адаптера.

```cmd
ipconfig /all
```

Ответ на вопрос:

```text
Машина вообще получила правильные сетевые настройки?
```

### `ipconfig /flushdns`

Очищает локальный DNS-кэш.

```cmd
ipconfig /flushdns
```

Использовать, когда DNS-запись поменяли, а клиент всё ещё ходит на старый адрес.

### `ping`

Проверяет базовую доступность.

```cmd
ping 8.8.8.8
ping server.domain.local
```

Логика:

```text
ping IP работает, имя нет -> вероятно DNS
ping gateway не работает -> проблема локальной сети/IP/mask
```

### `tracert`

Показывает маршрут до узла.

```cmd
tracert ya.ru
```

Полезно, когда нужно понять, где обрывается путь.

### `route print`

Показывает таблицу маршрутизации.

```cmd
route print
```

Смотреть default route:

```text
0.0.0.0/0 -> gateway
```

### `Test-NetConnection`

PowerShell-проверка доступности порта.

```powershell
Test-NetConnection server01 -Port 445
Test-NetConnection dc01.domain.local -Port 53
```

Ответ на вопрос:

```text
Сервер доступен именно по нужному порту?
```

## DNS

### `nslookup`

Проверяет разрешение имени.

```cmd
nslookup domain.local
nslookup dc01.domain.local
nslookup ya.ru
```

Для AD особенно важно проверять SRV-записи:

```cmd
nslookup -type=SRV _ldap._tcp.dc._msdcs.domain.local
```

Если AD “странно работает”, DNS проверяется одним из первых.

## Домен и пользователь

### `whoami`

Показывает текущего пользователя.

```cmd
whoami
```

### `whoami /groups`

Показывает группы и токен безопасности.

```cmd
whoami /groups
```

Использовать, когда пользователь говорит: “мне дали права, но доступа нет”.

### `gpupdate /force`

Принудительно обновляет групповые политики.

```cmd
gpupdate /force
```

После изменения GPO это одна из первых команд на клиенте.

### `gpresult /r`

Показывает применённые политики в консоли.

```cmd
gpresult /r
```

### `gpresult /h`

Создаёт HTML-отчёт по политикам.

```cmd
gpresult /h C:\Temp\gpresult.html
```

Полезно, если нужно спокойно посмотреть, какие GPO применились и почему.

### `nltest /dsgetdc`

Показывает, какой контроллер домена найден.

```cmd
nltest /dsgetdc:domain.local
```

Ответ на вопрос:

```text
Клиент вообще видит домен и DC?
```

## Службы

### `Get-Service`

Показывает службы.

```powershell
Get-Service
Get-Service -Name Spooler
```

### `Restart-Service`

Перезапускает службу.

```powershell
Restart-Service Spooler
Restart-Service DNS
```

### `sc query`

Старая, но полезная команда для проверки службы.

```cmd
sc query spooler
```

## События

### `eventvwr.msc`

Открывает Event Viewer.

```cmd
eventvwr.msc
```

Смотреть:

```text
Windows Logs -> System
Windows Logs -> Application
Applications and Services Logs
```

### `Get-WinEvent`

Быстро посмотреть события через PowerShell.

```powershell
Get-WinEvent -LogName System -MaxEvents 30
Get-WinEvent -LogName System | Where-Object LevelDisplayName -eq "Error"
```

## Файловые шары

### `net use`

Показывает и подключает сетевые диски.

```cmd
net use
net use Z: \\server\share
net use Z: /delete
```

### `Get-SmbShare`

Показывает SMB-шары на сервере.

```powershell
Get-SmbShare
```

## Active Directory PowerShell

Эти команды требуют модуль Active Directory.

```powershell
Get-ADUser username
Get-ADUser username -Properties *
Get-ADGroup "Group Name"
Get-ADGroupMember "Group Name"
Add-ADGroupMember -Identity "Group Name" -Members username
Disable-ADAccount -Identity username
Unlock-ADAccount -Identity username
```

Минимум на память:

```powershell
Get-ADUser
Get-ADGroup
Get-ADGroupMember
Add-ADGroupMember
Disable-ADAccount
Unlock-ADAccount
```

## Оснастки, которые нужно помнить

Подробно: [[14 - Оснастки Windows - msc команды]].

```text
dsa.msc       - пользователи, группы, OU
gpmc.msc      - групповые политики
dnsmgmt.msc   - DNS
dhcpmgmt.msc  - DHCP
eventvwr.msc  - события
services.msc  - службы
ncpa.cpl      - сетевые адаптеры
wf.msc        - firewall
diskmgmt.msc  - диски
virtmgmt.msc  - Hyper-V
```

## Абсолютный минимум

Если учить совсем короткий набор, начать с этого:

```cmd
ipconfig /all
ipconfig /flushdns
ping
tracert
route print
nslookup
whoami /groups
gpupdate /force
gpresult /r
nltest /dsgetdc:domain.local
net use
```

И PowerShell:

```powershell
Test-NetConnection server -Port 445
Get-Service
Restart-Service
Get-WinEvent -LogName System -MaxEvents 30
Get-ADUser username
Get-ADGroupMember "Group Name"
```

## Связанные заметки

- [[10 - Команды Windows Server - шпаргалка]]
- [[14 - Оснастки Windows - msc команды]]
- [[Сети/08 - Сетевая диагностика]]
- [[09 - Event Viewer и диагностика]]
- [[05 - GPO - групповые политики]]
