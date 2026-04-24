## Для диагностики используется `-v`.

```bash
ssh -v user@server.com
```

Более подробный вывод:

```bash
ssh -vv user@server.com
```

Максимально подробный вывод:

```bash
ssh -vvv user@server.com
```

С ключом:

```bash
ssh -v -i ~/.ssh/id_ed25519 user@server.com
```

С ключом и портом:

```bash
ssh -v -i ~/.ssh/id_ed25519 -p 2222 user@server.com
```

---

## Что смотреть в выводе

### Ключ найден и предлагается серверу

```text
Offering public key: /home/user/.ssh/id_ed25519
```

SSH-клиент нашёл ключ и пробует зайти с ним.

---

### Сервер принял ключ

```text
Server accepts key: /home/user/.ssh/id_ed25519
```

Ключ подходит. Если после этого вход не происходит — проблема может быть в shell, правах пользователя или ограничениях на сервере.

---

### Сервер отказал в ключе

```text
Permission denied (publickey)
```

Сервер не принял ключ.

Частые причины:

```text
ключ не добавлен в authorized_keys
ключ добавлен не тому пользователю
неправильные права на .ssh или authorized_keys
подключаешься не тем приватным ключом
на сервере отключён PubkeyAuthentication
```

---

### Клиент пробует не тот ключ

```text
Trying private key: /home/user/.ssh/id_rsa
Trying private key: /home/user/.ssh/id_ecdsa
Trying private key: /home/user/.ssh/id_ed25519
```

SSH перебирает ключи. Если нужного ключа нет в списке — указать явно:

```bash
ssh -i ~/.ssh/id_ed25519 user@server.com
```

---

### Слишком много ключей

```text
Too many authentication failures
```

Клиент предложил слишком много ключей, сервер оборвал подключение.

Использовать только указанный ключ:

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 user@server.com
```

С портом:

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 -p 2222 user@server.com
```

---

### Неверные права на сервере

В выводе может быть:

```text
Authentication refused: bad ownership or modes
```

Исправить на сервере:

```bash
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys
```

Также проверить домашнюю папку:

```bash
chown deploy:deploy /home/deploy
chmod 755 /home/deploy
```

---

### Не тот порт

```text
Connection refused
```

На этом порту SSH не слушает или порт закрыт.

Проверить другой порт:

```bash
ssh -p 2222 user@server.com
```

---

### Сервер недоступен

```text
Connection timed out
```

Сервер не отвечает.

Причины:

```text
неверный IP или домен
сервер выключен
порт закрыт firewall
порт закрыт у хостинга
SSH слушает другой порт
```

---

### Изменился fingerprint сервера

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

SSH считает, что сервер изменился. Часто бывает после переустановки сервера.

Удалить старую запись:

```bash
ssh-keygen -R server.com
```

Если порт нестандартный:

```bash
ssh-keygen -R "[server.com]:2222"
```

Потом подключиться заново:

```bash
ssh -p 2222 user@server.com
```

---

## Самая полезная команда диагностики

```bash
ssh -vvv -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 -p 2222 deploy@server.com
```

Она показывает:

```text
какой ключ используется
предлагается ли ключ серверу
принимает ли сервер ключ
на какой порт идёт подключение
на каком этапе происходит отказ
```