# `known_hosts` и проверка сервера

## Что такое `known_hosts`

`known_hosts` — файл, где SSH-клиент хранит отпечатки серверов, к которым уже подключался.

Файл обычно находится здесь:

```text
~/.ssh/known_hosts
```

---

## Зачем нужен `known_hosts`

Он нужен, чтобы SSH мог проверить:

```text
ты подключаешься к тому же серверу, что и раньше
```

Если ключ сервера изменился, SSH покажет предупреждение.

---

## Первое подключение к серверу

При первом подключении SSH покажет сообщение:

```text
The authenticity of host 'server.com' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Если сервер правильный, вводим:

```text
yes
```

После этого отпечаток сервера будет сохранён в:

```text
~/.ssh/known_hosts
```

---

## Что хранится в `known_hosts`

Пример строки:

```text
server.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...
```

Формат:

```text
server.com      — адрес сервера
ssh-ed25519     — тип host key
AAAAC3...       — публичный ключ сервера
```

---

## Важно не путать

`known_hosts` — это **ключи серверов**.

`authorized_keys` — это **ключи пользователей**, которым разрешён вход.

```text
~/.ssh/known_hosts      — серверы, которым доверяет клиент
~/.ssh/authorized_keys  — пользователи, которым разрешён вход на сервер
```

---

## Предупреждение о смене ключа

Если ключ сервера изменился, SSH покажет ошибку:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Это может означать:

```text
сервер переустановили
ключ сервера заменили
ты подключаешься не к тому серверу
возможна атака man-in-the-middle
```

---

## Удалить старую запись

Если ключ сервера был изменён легально, старую запись можно удалить:

```bash
ssh-keygen -R server.com
```

Для IP:

```bash
ssh-keygen -R 185.10.20.30
```

Для нестандартного порта:

```bash
ssh-keygen -R "[server.com]:2222"
```

После этого при новом подключении SSH снова спросит подтверждение.

---

## Посмотреть отпечаток сервера

Получить ключ сервера:

```bash
ssh-keyscan server.com
```

Получить отпечаток:

```bash
ssh-keyscan server.com | ssh-keygen -lf -
```

Для нестандартного порта:

```bash
ssh-keyscan -p 2222 server.com | ssh-keygen -lf -
```

---

## Проверить запись в `known_hosts`

```bash
ssh-keygen -F server.com
```

Для IP:

```bash
ssh-keygen -F 185.10.20.30
```

Для нестандартного порта:

```bash
ssh-keygen -F "[server.com]:2222"
```

---

## Отключать проверку опасно

Так можно отключить проверку ключа сервера:

```bash
ssh -o StrictHostKeyChecking=no user@server.com
```

Использовать осторожно: SSH перестанет нормально защищать от подмены сервера.

---

## Главное запомнить

Файл:

```text
~/.ssh/known_hosts
```

Он хранит ключи серверов, к которым уже подключались.

Первое подключение:

```text
Are you sure you want to continue connecting?
```

Ошибка смены ключа:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Удалить старую запись:

```bash
ssh-keygen -R server.com
```
