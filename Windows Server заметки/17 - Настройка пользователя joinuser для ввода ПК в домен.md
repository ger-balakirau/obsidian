# Настройка пользователя joinuser для ввода ПК в домен

`joinuser` - отдельная доменная учётная запись, которой разрешено вводить компьютеры в домен. Её используют в скриптах вида `Add-Computer`, чтобы не вводить доменного администратора на каждом новом ПК.

Главная идея:

```text
joinuser не должен быть Domain Admin
joinuser должен иметь право только создавать computer objects в нужной OU
```

## Зачем отдельный joinuser

Плохо:

```text
вводить ПК в домен под Domain Admin
```

Почему плохо:

- пароль администратора вводится на обычных рабочих местах;
- выше риск утечки;
- сложно понять, кто и что делал;
- слишком много прав для простой задачи.

Лучше:

```text
LAB\joinuser -> может вводить ПК только в OU=Workstations
```

## Рекомендуемая структура OU

Пример:

```text
OU=Kaluga
  OU=Workstations
  OU=Servers
  OU=Users
  OU=Groups
  OU=Service Accounts
```

`joinuser` лучше хранить не среди обычных пользователей, а в отдельной OU:

```text
OU=Service Accounts,OU=Kaluga,DC=lab,DC=loc
```

Компьютеры должны попадать сюда:

```text
OU=Workstations,OU=Kaluga,DC=lab,DC=loc
```

## Создать пользователя joinuser через GUI

Открыть:

```cmd
dsa.msc
```

Дальше:

1. Перейти в `OU=Service Accounts`.
2. Создать пользователя `joinuser`.
3. Задать сложный пароль.
4. Включить:

```text
Password never expires
```

5. Отключить:

```text
User must change password at next logon
```

Для учебной лабы это удобно. В реальной организации лучше делать по политике: ротация пароля, контроль доступа, аудит.

## Создать пользователя через PowerShell

На DC или машине с RSAT:

```powershell
$Password = Read-Host "Enter password for joinuser" -AsSecureString

New-ADUser `
    -Name "joinuser" `
    -SamAccountName "joinuser" `
    -UserPrincipalName "joinuser@lab.loc" `
    -Path "OU=Service Accounts,OU=Kaluga,DC=lab,DC=loc" `
    -AccountPassword $Password `
    -Enabled $true `
    -PasswordNeverExpires $true `
    -Description "Account for joining workstations to domain"
```

Проверить:

```powershell
Get-ADUser joinuser -Properties Enabled,PasswordNeverExpires,Description
```

## Делегировать право на ввод ПК в домен через GUI

Открыть:

```cmd
dsa.msc
```

Дальше:

1. Найти OU, куда должны попадать новые ПК:

```text
OU=Workstations,OU=Kaluga,DC=lab,DC=loc
```

2. Правый клик по OU.
3. `Delegate Control`.
4. Добавить пользователя:

```text
LAB\joinuser
```

5. Выбрать:

```text
Create a custom task to delegate
```

6. Выбрать:

```text
Only the following objects in the folder
Computer objects
```

7. Отметить:

```text
Create selected objects in this folder
Delete selected objects in this folder
```

`Delete` можно не выдавать, если нужно максимально строго. Но в lab и helpdesk-сценариях иногда удобно разрешить удаление ошибочно созданного объекта.

8. В permissions отметить минимум:

```text
Read
Write
Create All Child Objects
Delete All Child Objects
```

Если мастер показывает более точные права для computer objects, ориентируйся на:

```text
Create Computer objects
Delete Computer objects
Read
Write
Reset Password
Validated write to DNS host name
Validated write to service principal name
```

## Более простой вариант через Delegate Control Wizard

В некоторых версиях ADUC есть готовая задача:

```text
Join a computer to the domain
```

Если она есть, можно выбрать её. Если её нет - используй custom delegation из предыдущего раздела.

## Важное ограничение по умолчанию

В домене обычные пользователи часто могут добавить до 10 компьютеров в домен из-за атрибута:

```text
ms-DS-MachineAccountQuota
```

Но для нормальной админской практики лучше не полагаться на это поведение, а явно делегировать права на конкретную OU.

Проверить значение:

```powershell
Get-ADDomain | Select-Object DistinguishedName,ms-DS-MachineAccountQuota
```

В более строгой среде его могут поставить в `0`, чтобы обычные пользователи не могли добавлять компьютеры куда попало.

## Использование в join-domain.ps1

В скрипте можно указывать:

```powershell
$JoinAccount = "LAB\joinuser"
```

Команда:

```powershell
Add-Computer `
    -DomainName "lab.loc" `
    -OUPath "OU=Workstations,OU=Kaluga,DC=lab,DC=loc" `
    -Credential "LAB\joinuser" `
    -Restart
```

После запуска скрипт спросит пароль от `joinuser`.

Пароль в скрипт не записывать:

```powershell
# Плохо
$Password = "P@ssw0rd"
```

Лучше пусть PowerShell спросит пароль интерактивно.

## Проверка прав joinuser

На тестовом клиенте:

```powershell
Add-Computer `
    -DomainName "lab.loc" `
    -OUPath "OU=Workstations,OU=Kaluga,DC=lab,DC=loc" `
    -Credential "LAB\joinuser" `
    -Restart
```

После перезагрузки проверить:

```cmd
whoami
nltest /dsgetdc:lab.loc
gpresult /r
```

На DC:

```cmd
dsa.msc
```

Проверить:

- компьютер появился в `OU=Workstations`;
- объект не попал в стандартный контейнер `Computers`;
- GPO для этой OU применяются.

## Типовые ошибки

### Access is denied

Причины:

- `joinuser` не делегирован на нужную OU;
- указан неправильный `OUPath`;
- пользователь заблокирован;
- пароль неверный.

Проверить:

```powershell
Get-ADUser joinuser -Properties Enabled,LockedOut
```

### The specified domain either does not exist or could not be contacted

Обычно это DNS.

На клиенте:

```cmd
ipconfig /all
nslookup lab.loc
nltest /dsgetdc:lab.loc
```

DNS должен указывать на контроллер домена.

### Компьютер попал в Computers, а не в OU

Значит join был сделан без `-OUPath` или OU указан неверно.

Правильно:

```powershell
-OUPath "OU=Workstations,OU=Kaluga,DC=lab,DC=loc"
```

### The account already exists

В AD уже есть computer object с таким именем.

Варианты:

- удалить старый объект, если он не нужен;
- сбросить account;
- выбрать другое имя компьютера;
- проверить, не используется ли старый ODJ-файл.

## Минимальные правила безопасности

- Не добавлять `joinuser` в `Domain Admins`.
- Не хранить пароль в `.ps1`, `.cmd`, `.txt` на флешке.
- Делегировать права только на нужную OU.
- Для серверов сделать отдельную OU и отдельную процедуру.
- Периодически менять пароль `joinuser`.
- Отключать `joinuser`, если он временно не нужен.
- Вести понятный шаблон имён компьютеров.

## Что сказать на собеседовании

Хороший ответ:

```text
Я бы не использовал Domain Admin для ввода рабочих станций в домен. Создал бы отдельную учётку joinuser, делегировал ей право создавать computer objects только в OU Workstations и использовал бы её в Add-Computer или в процессе подготовки ODJ-файлов.
```

## Связанные заметки

- [[04 - Пользователи, группы и OU]]
- [[06 - Join computer to domain]]
- [[16 - Автоматический ввод ПК в домен с флешки]]
- [[14 - Оснастки Windows - msc команды]]
