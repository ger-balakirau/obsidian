# SSH agent

## Что такое SSH agent

SSH agent хранит приватные ключи в памяти.

Это нужно, чтобы не вводить пароль от ключа при каждом подключении.

---

## Запуск агента

```bash
eval "$(ssh-agent -s)"
```

Пример вывода:

```text
Agent pid 12345
```

---

## Добавить ключ в агент

```bash
ssh-add ~/.ssh/id_ed25519
```

Если у ключа есть пароль, SSH попросит его один раз.

---

## Посмотреть добавленные ключи

```bash
ssh-add -l
```

Если ключей нет:

```text
The agent has no identities.
```

---

## Удалить конкретный ключ

```bash
ssh-add -d ~/.ssh/id_ed25519
```

---

## Удалить все ключи из агента

```bash
ssh-add -D
```

---

## Проверить, работает ли агент

```bash
echo $SSH_AUTH_SOCK
```

Если агент работает, будет путь к сокету:

```text
/tmp/ssh-xxxxxx/agent.12345
```

Если вывода нет, агент не подключён к текущей сессии.

---

## Подключение с ключом из агента

После добавления ключа можно подключаться обычно:

```bash
ssh user@server.com
```

Без указания:

```bash
-i ~/.ssh/id_ed25519
```

---

## Добавить ключ на ограниченное время

Например, на 1 час:

```bash
ssh-add -t 1h ~/.ssh/id_ed25519
```

На 10 минут:

```bash
ssh-add -t 10m ~/.ssh/id_ed25519
```

---

## SSH agent forwarding

Agent forwarding позволяет использовать локальный SSH agent на удалённом сервере.

Подключение с forwarding:

```bash
ssh -A user@server.com
```

Использовать осторожно: удалённый сервер получает доступ к твоему agent socket.

Лучше включать только там, где это действительно нужно.

---

## Отключить agent forwarding

В команде:

```bash
ssh -a user@server.com
```

В `~/.ssh/config`:

```sshconfig
Host server
    HostName server.com
    User user
    ForwardAgent no
```

---

## Главное запомнить

Запустить агент:

```bash
eval "$(ssh-agent -s)"
```

Добавить ключ:

```bash
ssh-add ~/.ssh/id_ed25519
```

Посмотреть ключи:

```bash
ssh-add -l
```

Удалить все ключи:

```bash
ssh-add -D
```
