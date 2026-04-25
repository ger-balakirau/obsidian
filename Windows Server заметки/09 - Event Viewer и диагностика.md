# Event Viewer и диагностика

Event Viewer - основной инструмент просмотра событий Windows.

## Где смотреть

```text
Event Viewer
```

Основные журналы:

```text
Windows Logs -> System
Windows Logs -> Application
Windows Logs -> Security
Applications and Services Logs
```

## Что смотреть первым

### System

Службы, драйверы, сеть, диски, перезагрузки.

### Application

Ошибки приложений.

### Security

Логины, аудит, доступы.

### Directory Service

На контроллере домена: события AD.

### DNS Server

На DNS-сервере: ошибки DNS.

## Фильтр событий

В Event Viewer удобно фильтровать по:

- Level: Error, Warning;
- Event ID;
- Source;
- времени.

## PowerShell

Последние события System:

```powershell
Get-EventLog -LogName System -Newest 30
```

Ошибки:

```powershell
Get-EventLog -LogName System -EntryType Error -Newest 30
```

Новый вариант:

```powershell
Get-WinEvent -LogName System -MaxEvents 30
```

## Диагностика службы

```powershell
Get-Service
Get-Service -Name Spooler
Restart-Service Spooler
```

## Типовой порядок

1. Зафиксировать симптом.
2. Проверить время проблемы.
3. Открыть Event Viewer.
4. Отфильтровать Error/Warning за нужное время.
5. Проверить связанный сервис.
6. Проверить сеть/DNS, если ошибка доменная.

## Главное запомнить

- Event Viewer - первое место для Windows-диагностики.
- Важно смотреть время события.
- Для AD/DNS/WSUS есть отдельные журналы в Applications and Services Logs.
