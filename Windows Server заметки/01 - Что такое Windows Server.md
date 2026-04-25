# Что такое Windows Server

Windows Server - серверная ОС Microsoft для доменов, файловых серверов, DNS/DHCP, обновлений, приложений и инфраструктурных служб.

## Где используется

- Active Directory.
- DNS / DHCP.
- Файловые шары.
- Print server.
- WSUS.
- RDS.
- 1С и другие бизнес-приложения.

## Роли и компоненты

Роль - крупная серверная функция.

Примеры:

```text
Active Directory Domain Services
DNS Server
DHCP Server
File and Storage Services
Windows Server Update Services
Remote Desktop Services
```

## Server Manager

`Server Manager` - базовая графическая консоль для установки ролей и просмотра состояния сервера.

Обычно через неё:

- добавляют роли;
- смотрят предупреждения;
- открывают оснастки;
- управляют локальным сервером.

## PowerShell

Многие действия можно делать через PowerShell.

Примеры:

```powershell
Get-ComputerInfo
Get-Service
Get-WindowsFeature
Get-EventLog -LogName System -Newest 20
```

## Что важно для администратора

- понимать роли сервера;
- знать, где смотреть события;
- уметь проверять службы;
- понимать сеть сервера;
- не менять доменную инфраструктуру вслепую.

## Главное запомнить

Windows Server - это не просто “Windows с интерфейсом”. Это набор серверных ролей, от которых зависит вход пользователей, DNS, политики, обновления и доступ к ресурсам.
