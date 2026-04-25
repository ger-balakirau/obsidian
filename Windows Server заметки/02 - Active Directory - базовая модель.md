# Active Directory - базовая модель

Active Directory - служба каталогов Microsoft. Она хранит пользователей, компьютеры, группы и политики домена.

## Простыми словами

AD отвечает на вопросы:

- кто пользователь;
- в какие группы он входит;
- к каким ресурсам у него доступ;
- какие политики должны примениться;
- какие компьютеры входят в домен.

## Домен

Домен - логическая область управления.

Пример:

```text
company.local
```

Пользователь домена:

```text
COMPANY\ivanov
ivanov@company.local
```

## Контроллер домена

Domain Controller хранит AD и обслуживает вход пользователей.

Если контроллер домена недоступен:

- пользователи могут не войти;
- GPO не применятся;
- доменные ресурсы будут недоступны;
- DNS в домене может сломаться.

## Объекты AD

Частые объекты:

```text
User
Computer
Group
Organizational Unit
Group Policy Object
```

## OU

OU - контейнер для организации пользователей и компьютеров.

Пример:

```text
Company
├── Users
├── Workstations
├── Servers
└── Service Accounts
```

На OU удобно привязывать GPO.

## Группы

Группы используют для выдачи доступа.

Правильная логика:

```text
пользователь -> группа -> доступ к ресурсу
```

Неудобная логика:

```text
права напрямую каждому пользователю
```

## Главные консоли

```text
Active Directory Users and Computers
Active Directory Administrative Center
Group Policy Management
DNS Manager
```

## Команды проверки

На клиенте:

```cmd
whoami
whoami /groups
gpresult /r
nltest /dsgetdc:domain.local
```

PowerShell:

```powershell
Get-ADUser username
Get-ADGroupMember "Group Name"
```

## Главное запомнить

- AD хранит пользователей, компьютеры, группы и политики.
- DNS критичен для AD.
- Доступ лучше выдавать через группы.
- GPO обычно привязывают к OU.
