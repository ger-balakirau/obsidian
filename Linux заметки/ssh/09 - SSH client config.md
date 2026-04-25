# SSH client config

`~/.ssh/config` — это конфиг SSH-клиента. Он нужен, чтобы не писать каждый раз длинные команды с пользователем, адресом, портом и ключом.

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

## Готовое имя подключения

Можно настроить подключение один раз, а потом заходить короткой командой:

```bash
ssh work
```

Для этого в `~/.ssh/config` нужен блок:

```sshconfig
Host work
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/work_ed25519
    IdentitiesOnly yes
```

Что здесь важно:

- `Host work` — короткое имя, которое пишется после `ssh`;
- `HostName` — реальный IP или домен сервера;
- `User` — пользователь на сервере;
- `Port` — SSH-порт;
- `IdentityFile` — приватный ключ на локальном компьютере;
- `IdentitiesOnly yes` — использовать именно этот ключ.

После этого алиас `work` можно использовать не только с `ssh`:

```bash
scp file.txt work:/tmp/
rsync -av ./project/ work:/srv/project/
sftp work
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

## Главное запомнить

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
