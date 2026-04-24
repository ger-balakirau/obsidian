## Файл конфига

SSH client config хранится здесь:

```text
~/.ssh/config
```

Если файла нет, его можно создать.

---

## Базовый пример

```sshconfig
Host myserver
    HostName server.com
    User user
    Port 22
```

После этого можно подключаться короткой командой:

```bash
ssh myserver
```

---

## Подключение по ключу

```sshconfig
Host myserver
    HostName server.com
    User user
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Подключение:

```bash
ssh myserver
```

---

## Нестандартный порт

```sshconfig
Host production
    HostName 185.10.20.30
    User root
    Port 2222
```

Подключение:

```bash
ssh production
```

---

## Несколько серверов

```sshconfig
Host production
    HostName 185.10.20.30
    User root
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod

Host staging
    HostName staging.example.com
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519_staging
```

Подключение:

```bash
ssh production
```

```bash
ssh staging
```

---

## Основные параметры

```text
Host          — короткое имя подключения
HostName      — IP-адрес или домен сервера
User          — пользователь на сервере
Port          — SSH-порт
IdentityFile  — путь к приватному ключу
```

---

# Главное запомнить

Файл:

```text
~/.ssh/config
```

Пример:

```sshconfig
Host myserver
    HostName server.com
    User user
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Подключение:

```bash
ssh myserver
```