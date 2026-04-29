# Смена hostname в Linux

`hostname` - имя машины внутри системы. Оно видно в терминале, логах, SSH-сессиях, мониторинге и иногда в DNS/hosts.

Пример:

```text
proxy-vm
lab-app
vm-www-db
```

## Посмотреть текущий hostname

Коротко:

```bash
hostname
```

Через `systemd`:

```bash
hostnamectl
```

Только static hostname:

```bash
hostnamectl --static
```

## Сменить hostname через hostnamectl

Современный способ для Ubuntu, Debian, CentOS, AlmaLinux и других систем с `systemd`:

```bash
sudo hostnamectl set-hostname proxy-vm
```

Проверить:

```bash
hostnamectl
hostname
```

Обычно перезагрузка не нужна, но старые сессии терминала могут показывать старое имя до переподключения.

## Проверить /etc/hostname

Файл `/etc/hostname` хранит постоянное имя машины.

```bash
cat /etc/hostname
```

Если редактировать вручную:

```bash
sudo nano /etc/hostname
```

Внутри должно быть только имя:

```text
proxy-vm
```

## Проверить /etc/hosts

После смены hostname проверь `/etc/hosts`.

```bash
sudo nano /etc/hosts
```

Типовой вариант для Ubuntu:

```text
127.0.0.1 localhost
127.0.1.1 proxy-vm
```

Если имя не совпадает с `/etc/hostname`, могут появляться странные задержки при `sudo` или сообщения вида:

```text
sudo: unable to resolve host old-name
```

## Быстрый безопасный порядок

```bash
sudo hostnamectl set-hostname proxy-vm
cat /etc/hostname
sudo nano /etc/hosts
hostnamectl
```

В `/etc/hosts` привести строку `127.0.1.1` к новому имени:

```text
127.0.1.1 proxy-vm
```

## FQDN и короткое имя

Для маленькой lab-сети обычно достаточно короткого имени:

```text
proxy-vm
```

Если есть доменная зона, можно использовать FQDN:

```text
proxy.lab
```

Но важно понимать:

```text
hostname машины != DNS-запись
```

Если ты назвал сервер `proxy-vm`, это не значит, что с Windows автоматически заработает:

```powershell
ssh user@proxy.lab
```

Для имени `proxy.lab` нужна DNS-запись или запись в `hosts`.

## Прописать имя на Windows

Если в lab нет DNS, можно временно прописать имя на Windows.

Файл:

```text
C:\Windows\System32\drivers\etc\hosts
```

Открывать от имени администратора.

Пример:

```text
10.10.0.10 proxy.lab
10.10.0.11 app.lab
10.10.0.12 www-db.lab
```

Проверка:

```powershell
ping proxy.lab
nslookup proxy.lab
ssh user@proxy.lab
```

Важно: `ping` может работать через `hosts`, а `nslookup` обычно спрашивает DNS-сервер и может не показывать записи из `hosts`.

## Узнать IP сервера

Коротко:

```bash
hostname -I
```

Подробно:

```bash
ip -4 addr
```

Маршрут по умолчанию:

```bash
ip route
```

## Частые ошибки

### Сменил hostname, но Windows не видит proxy.lab

Причина: hostname внутри Linux не создаёт DNS-запись.

Решение:

- добавить запись в DNS;
- или прописать `proxy.lab` в Windows `hosts`;
- или подключаться по IP.

### sudo ругается unable to resolve host

Проверить:

```bash
cat /etc/hostname
cat /etc/hosts
```

Имя в `/etc/hostname` должно совпадать со строкой `127.0.1.1` в `/etc/hosts`.

### В старой SSH-сессии видно старое имя

Переподключиться:

```bash
exit
ssh user@server
```

Или открыть новый терминал.

## Минимум на память

```bash
hostname
hostnamectl
sudo hostnamectl set-hostname proxy-vm
cat /etc/hostname
sudo nano /etc/hosts
hostname -I
ip -4 addr
```

## Связанные заметки

- [[02 - Ubuntu Server - базовая настройка]]
- [[04 - Ubuntu lab proxy bastion и reverse proxy]]
- [[Linux заметки/ssh/09 - SSH client config]]
- [[Сети/05 - DNS]]
