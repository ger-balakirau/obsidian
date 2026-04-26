# Оснастки Windows - msc команды

`*.msc` - это оснастки Microsoft Management Console. Их удобно запускать через `Win + R`, командную строку или PowerShell.

Главная идея: администратор должен быстро понимать, какую консоль открыть под конкретную задачу.

## Active Directory

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `dsa.msc` | Active Directory Users and Computers | Пользователи, группы, OU, компьютеры |
| `gpmc.msc` | Group Policy Management | Создание, привязка и проверка GPO |
| `domain.msc` | Active Directory Domains and Trusts | Домены, доверительные отношения |
| `dssite.msc` | Active Directory Sites and Services | Сайты AD, репликация, контроллеры домена |
| `adsiedit.msc` | ADSI Edit | Низкоуровневое редактирование AD, использовать осторожно |

## DNS, DHCP и сеть

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `dnsmgmt.msc` | DNS Manager | Зоны DNS, записи A/CNAME/PTR, диагностика AD DNS |
| `dhcpmgmt.msc` | DHCP Manager | Scope, lease, reservation, DHCP options |
| `wf.msc` | Windows Defender Firewall with Advanced Security | Правила firewall, входящие/исходящие подключения |
| `ncpa.cpl` | Network Connections | Сетевые адаптеры, IP, DNS, gateway |

`ncpa.cpl` не `.msc`, но его тоже нужно помнить: это самый быстрый вход в сетевые адаптеры.

## Система и диагностика

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `eventvwr.msc` | Event Viewer | Ошибки, события служб, AD, DNS, DHCP, входы |
| `services.msc` | Services | Запуск, остановка и автозагрузка служб |
| `taskschd.msc` | Task Scheduler | Планировщик задач |
| `devmgmt.msc` | Device Manager | Драйверы, устройства, сетевые карты |
| `diskmgmt.msc` | Disk Management | Диски, разделы, буквы, тома |
| `compmgmt.msc` | Computer Management | Общая консоль: пользователи, диски, службы, события |
| `perfmon.msc` | Performance Monitor | Счётчики производительности |
| `resmon` | Resource Monitor | CPU, RAM, Disk, Network по процессам |

## Локальная безопасность и политики

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `secpol.msc` | Local Security Policy | Локальные политики безопасности |
| `gpedit.msc` | Local Group Policy Editor | Локальные политики на одной машине |
| `lusrmgr.msc` | Local Users and Groups | Локальные пользователи и группы |
| `certlm.msc` | Certificates - Local Computer | Сертификаты компьютера |
| `certmgr.msc` | Certificates - Current User | Сертификаты текущего пользователя |

Важно: на доменных машинах основная логика обычно должна жить в `gpmc.msc`, а не в `gpedit.msc`.

## Роли и серверные компоненты

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `servermanager` | Server Manager | Роли, фичи, состояние сервера |
| `fsmgmt.msc` | Shared Folders | Общие папки, открытые файлы, сессии |
| `printmanagement.msc` | Print Management | Принтеры, очереди, драйверы печати |
| `iis.msc` | IIS Manager | Сайты IIS, app pools, bindings |
| `tsconfig.msc` | Remote Desktop Session Host Configuration | Старые версии Windows Server |
| `tsadmin.msc` | Remote Desktop Services Manager | Старые версии Windows Server |

В новых версиях Windows Server часть RDS-инструментов переехала в Server Manager и PowerShell.

## Hyper-V

| Команда | Что открывает | Когда использовать |
|---|---|---|
| `virtmgmt.msc` | Hyper-V Manager | Виртуальные машины, virtual switch, состояние VM |
| `vmconnect.exe` | Virtual Machine Connection | Подключиться к консоли VM |

## Быстрый выбор по задаче

| Задача | Команда |
|---|---|
| Создать пользователя в домене | `dsa.msc` |
| Настроить GPO | `gpmc.msc` |
| Проверить DNS-запись | `dnsmgmt.msc` |
| Проверить DHCP-аренду | `dhcpmgmt.msc` |
| Посмотреть ошибки | `eventvwr.msc` |
| Перезапустить службу | `services.msc` |
| Проверить сетевой адаптер | `ncpa.cpl` |
| Посмотреть firewall | `wf.msc` |
| Открыть шары и сессии | `fsmgmt.msc` |
| Открыть Hyper-V | `virtmgmt.msc` |

## Что выучить первым

```text
dsa.msc
gpmc.msc
dnsmgmt.msc
dhcpmgmt.msc
eventvwr.msc
services.msc
ncpa.cpl
wf.msc
compmgmt.msc
diskmgmt.msc
virtmgmt.msc
```

## Связанные заметки

- [[10 - Команды Windows Server - шпаргалка]]
- [[15 - Команды Windows админа наизусть]]
- [[02 - Active Directory - базовая модель]]
- [[05 - GPO - групповые политики]]
- [[09 - Event Viewer и диагностика]]
