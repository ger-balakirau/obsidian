# Частые ошибки SSH

Эта заметка отвечает на вопрос “что делать при конкретной ошибке”. Если нужно понять подробный вывод `ssh -v`, смотри [[15 - Диагностика SSH-подключения]].

## `Permission denied`

Ошибка:

```text
Permission denied (publickey).
```

Причины:

```text
не тот пользователь
не тот ключ
публичный ключ не добавлен на сервер
неправильные права на .ssh или authorized_keys
вход по паролю отключён
```

Проверить подключение подробно:

```bash
ssh -v user@server.com
```

Подключиться с конкретным ключом:

```bash
ssh -i ~/.ssh/id_ed25519 user@server.com
```

---

## `Connection refused`

Ошибка:

```text
ssh: connect to host server.com port 22: Connection refused
```

Причины:

```text
SSH-сервер не запущен
SSH слушает другой порт
порт закрыт firewall
```

Проверить порт:

```bash
nc -vz server.com 22
```

Если порт другой:

```bash
ssh -p 2222 user@server.com
```

---

## `Connection timed out`

Ошибка:

```text
ssh: connect to host server.com port 22: Connection timed out
```

Причины:

```text
сервер недоступен
порт закрыт firewall
неправильный IP
проблема с сетью
```

Проверить доступность:

```bash
ping server.com
```

Проверить порт:

```bash
nc -vz server.com 22
```

---

## `Host key verification failed`

Ошибка:

```text
Host key verification failed.
```

Причины:

```text
ключ сервера изменился
старая запись в known_hosts
подключение идёт не к тому серверу
```

Удалить старую запись:

```bash
ssh-keygen -R server.com
```

Для нестандартного порта:

```bash
ssh-keygen -R "[server.com]:2222"
```

---

## `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED`

Ошибка:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Причины:

```text
сервер переустановили
изменился host key
возможна подмена сервера
```

Если изменение ожидаемое, удалить старую запись:

```bash
ssh-keygen -R server.com
```

Потом подключиться заново:

```bash
ssh user@server.com
```

---

## `Too many authentication failures`

Ошибка:

```text
Too many authentication failures
```

Причина: SSH-клиент перебирает слишком много ключей из agent.

Решение:

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 user@server.com
```

В `~/.ssh/config`:

```sshconfig
Host myserver
    HostName server.com
    User user
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

---

## `UNPROTECTED PRIVATE KEY FILE`

Ошибка:

```text
WARNING: UNPROTECTED PRIVATE KEY FILE!
Permissions 0644 for 'id_ed25519' are too open.
```

Причина: приватный ключ доступен слишком широко.

Исправить:

```bash
chmod 600 ~/.ssh/id_ed25519
```

Права на директорию:

```bash
chmod 700 ~/.ssh
```

---

## `Bad owner or permissions`

Ошибка:

```text
Bad owner or permissions on ~/.ssh/config
```

Причина: неправильные права на SSH-файлы.

Исправить:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Для `authorized_keys` на сервере:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## `No such identity`

Ошибка:

```text
Warning: Identity file ~/.ssh/id_ed25519 not accessible: No such file or directory.
```

Причина: указанного ключа нет.

Проверить ключи:

```bash
ls -la ~/.ssh
```

Указать правильный ключ:

```bash
ssh -i ~/.ssh/key_name user@server.com
```

---

## `Could not resolve hostname`

Ошибка:

```text
ssh: Could not resolve hostname server.com: Name or service not known
```

Причины:

```text
ошибка в домене
DNS не работает
неправильный Host в ~/.ssh/config
```

Проверить домен:

```bash
ping server.com
```

Или подключиться по IP:

```bash
ssh user@185.10.20.30
```

---

## `Connection closed by remote host`

Ошибка:

```text
Connection closed by remote host
```

Причины:

```text
сервер закрыл соединение
пользователь запрещён
ошибка в sshd_config
ограничения AllowUsers или DenyUsers
```

Проверить на сервере:

```bash
sudo sshd -t
```

Логи:

```bash
sudo journalctl -u ssh -f
```

---

## `No route to host`

Ошибка:

```text
No route to host
```

Причины:

```text
сервер недоступен по сети
неправильный IP
маршрут отсутствует
firewall блокирует подключение
```

Проверить:

```bash
ping server.com
```

```bash
traceroute server.com
```

---

## `Operation timed out`

Ошибка:

```text
Operation timed out
```

Похожа на `Connection timed out`.

Проверить:

```bash
nc -vz server.com 22
```

С нестандартным портом:

```bash
nc -vz server.com 2222
```

---

## `kex_exchange_identification`

Ошибка:

```text
kex_exchange_identification: Connection closed by remote host
```

Причины:

```text
сервер сбрасывает соединение
слишком много подключений
ограничения MaxStartups
блокировка firewall или fail2ban
```

Проверить логи:

```bash
sudo journalctl -u ssh -f
```

Проверить fail2ban:

```bash
sudo fail2ban-client status sshd
```

---

## `sign_and_send_pubkey: signing failed`

Ошибка:

```text
sign_and_send_pubkey: signing failed
```

Причины:

```text
проблема с SSH agent
ключ не добавлен в agent
ключ заблокирован
```

Проверить agent:

```bash
ssh-add -l
```

Добавить ключ:

```bash
ssh-add ~/.ssh/id_ed25519
```

---

## Отладка SSH

Подробный вывод:

```bash
ssh -v user@server.com
```

Ещё подробнее:

```bash
ssh -vv user@server.com
```

Максимально подробно:

```bash
ssh -vvv user@server.com
```

---

## Логи на сервере

Для systemd:

```bash
sudo journalctl -u ssh -f
```

Или:

```bash
sudo journalctl -u sshd -f
```

На Ubuntu/Debian часто:

```bash
sudo tail -f /var/log/auth.log
```

---

## Главное запомнить

Проверить подключение подробно:

```bash
ssh -v user@server.com
```

Проверить порт:

```bash
nc -vz server.com 22
```

Удалить старый host key:

```bash
ssh-keygen -R server.com
```

Исправить права:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/authorized_keys
```

Проверить серверный конфиг:

```bash
sudo sshd -t
```
