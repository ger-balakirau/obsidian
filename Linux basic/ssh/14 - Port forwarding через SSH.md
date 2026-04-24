## Что такое port forwarding

Port forwarding через SSH — это проброс порта через защищённое SSH-соединение.

Используется, когда нужно получить доступ к сервису, который напрямую недоступен.

Схема:

```text
локальный компьютер → SSH-сервер → нужный сервис
```

---

# 1. Локальный проброс порта `-L`

Локальный проброс открывает порт на твоём компьютере и передаёт трафик через SSH на сервер или дальше.

Формат:

```bash
ssh -L local_port:target_host:target_port user@ssh_server
```

Пример:

```bash
ssh -L 8080:localhost:80 user@server.com
```

Это значит:

```text
localhost:8080 на твоём компьютере
→ через SSH
→ localhost:80 на сервере
```

Теперь в браузере можно открыть:

```text
http://localhost:8080
```

---

## Пример: доступ к PostgreSQL на сервере

```bash
ssh -L 5432:localhost:5432 user@server.com
```

Теперь локально можно подключаться к базе так:

```text
localhost:5432
```

Хотя сама база находится на сервере.

---

## Пример: доступ к MySQL на сервере

```bash
ssh -L 3306:localhost:3306 user@server.com
```

Локальное подключение:

```text
localhost:3306
```

---

## Пример: доступ к внутреннему серверу через SSH-сервер

```bash
ssh -L 8080:10.0.0.5:80 user@gateway.com
```

Это значит:

```text
твой компьютер:8080
→ gateway.com
→ 10.0.0.5:80
```

Используется, когда `10.0.0.5` доступен с `gateway.com`, но недоступен напрямую с твоего компьютера.

---

# 2. Удалённый проброс порта `-R`

Удалённый проброс открывает порт на SSH-сервере и передаёт трафик на твой локальный компьютер.

Формат:

```bash
ssh -R remote_port:target_host:target_port user@ssh_server
```

Пример:

```bash
ssh -R 9000:localhost:3000 user@server.com
```

Это значит:

```text
server.com:9000
→ через SSH
→ твой компьютер:3000
```

---

## Пример: показать локальный сайт через сервер

На твоём компьютере запущено приложение:

```text
localhost:3000
```

Проброс:

```bash
ssh -R 9000:localhost:3000 user@server.com
```

Теперь на сервере можно открыть:

```text
localhost:9000
```

И попасть на приложение с твоего компьютера.

---

## Важно про внешний доступ к `-R`

По умолчанию удалённый порт часто доступен только как:

```text
localhost:9000
```

на самом сервере.

Чтобы порт был доступен снаружи, на сервере в `/etc/ssh/sshd_config` нужен параметр:

```sshconfig
GatewayPorts yes
```

Или более аккуратно:

```sshconfig
GatewayPorts clientspecified
```

После изменения:

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Тогда можно указать адрес:

```bash
ssh -R 0.0.0.0:9000:localhost:3000 user@server.com
```

---

# 3. Динамический SOCKS-прокси `-D`

Динамический проброс создаёт SOCKS-прокси через SSH.

Формат:

```bash
ssh -D local_port user@ssh_server
```

Пример:

```bash
ssh -D 1080 user@server.com
```

После этого SOCKS-прокси будет доступен локально:

```text
localhost:1080
```

Его можно указать в браузере, curl или другой программе.

---

## Пример с `curl`

```bash
curl --socks5 localhost:1080 https://example.com
```

Если нужно, чтобы DNS тоже шёл через прокси:

```bash
curl --socks5-hostname localhost:1080 https://example.com
```

---

# 4. Не открывать shell-сессию: `-N`

Для туннелей часто не нужен вход в shell.

Используется ключ:

```bash
-N
```

Пример:

```bash
ssh -N -L 8080:localhost:80 user@server.com
```

SSH-сессия будет висеть только для проброса порта.

---

# 5. Запустить туннель в фоне: `-f`

```bash
ssh -f -N -L 8080:localhost:80 user@server.com
```

Где:

```text
-f — отправить SSH в фон
-N — не запускать shell
-L — локальный проброс
```

---

# 6. Проброс на конкретный локальный адрес

По умолчанию локальный порт обычно слушает только `localhost`.

Пример:

```bash
ssh -L 8080:localhost:80 user@server.com
```

То же самое явно:

```bash
ssh -L 127.0.0.1:8080:localhost:80 user@server.com
```

Если нужно открыть порт для других устройств в сети:

```bash
ssh -L 0.0.0.0:8080:localhost:80 user@server.com
```

Осторожно: так порт станет доступен не только тебе.

---

# 7. Port forwarding через `~/.ssh/config`

Чтобы не писать длинную команду каждый раз:

```sshconfig
Host web-tunnel
    HostName server.com
    User user
    LocalForward 8080 localhost:80
```

Запуск:

```bash
ssh web-tunnel
```

С `-N`:

```bash
ssh -N web-tunnel
```

---

## Пример для базы данных

```sshconfig
Host db-tunnel
    HostName server.com
    User user
    LocalForward 5432 localhost:5432
```

Запуск:

```bash
ssh -N db-tunnel
```

Подключение к базе локально:

```text
localhost:5432
```

---

## Пример SOCKS-прокси через config

```sshconfig
Host socks-tunnel
    HostName server.com
    User user
    DynamicForward 1080
```

Запуск:

```bash
ssh -N socks-tunnel
```

SOCKS-прокси:

```text
localhost:1080
```

---

# 8. Полезные параметры

## Подробный вывод

```bash
ssh -v -N -L 8080:localhost:80 user@server.com
```

Ещё подробнее:

```bash
ssh -vvv -N -L 8080:localhost:80 user@server.com
```

---

## Завершить туннель

Если SSH запущен в обычном режиме — нажать:

```text
Ctrl + C
```

Если SSH запущен в фоне, найти процесс:

```bash
ps aux | grep ssh
```

Завершить:

```bash
kill PID
```

---

## Проверить, слушает ли порт

```bash
ss -tlnp | grep 8080
```

Или:

```bash
lsof -i :8080
```

---

# 9. Частые ошибки

## Порт уже занят

Ошибка:

```text
bind: Address already in use
```

Значит локальный порт уже используется.

Решение: выбрать другой порт.

```bash
ssh -L 8081:localhost:80 user@server.com
```

---

## Сервис недоступен

Туннель поднялся, но сервис не открывается.

Проверить на сервере:

```bash
curl localhost:80
```

Или:

```bash
ss -tlnp
```

---

## Удалённый проброс не доступен снаружи

Причина: сервер слушает только `localhost`.

Проверить `sshd_config`:

```sshconfig
GatewayPorts yes
```

Или:

```sshconfig
GatewayPorts clientspecified
```

---

# Главное запомнить

Локальный проброс:

```bash
ssh -L 8080:localhost:80 user@server.com
```

Удалённый проброс:

```bash
ssh -R 9000:localhost:3000 user@server.com
```

SOCKS-прокси:

```bash
ssh -D 1080 user@server.com
```

Только туннель без shell:

```bash
ssh -N -L 8080:localhost:80 user@server.com
```

Туннель в фоне:

```bash
ssh -f -N -L 8080:localhost:80 user@server.com
```