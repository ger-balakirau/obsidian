# Команды SSH - общая шпаргалка

Эта заметка — быстрый индекс команд. Подробные объяснения лежат в отдельных темах по ссылкам.

## Подключение

Подробнее: [[02 - Подключение по SSH через консоль]]

```bash
ssh user@server.com                  # обычное подключение
ssh root@192.168.1.10                # подключение по IP
ssh -p 2222 user@server.com          # нестандартный порт
ssh user@server.com "uptime"         # выполнить одну команду на сервере
ssh -o ConnectTimeout=10 user@server.com
```

## Подключение по готовому имени

Подробнее: [[09 - SSH client config]]

Если в `~/.ssh/config` есть блок:

```sshconfig
Host work
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/work_ed25519
    IdentitiesOnly yes
```

то подключение будет коротким:

```bash
ssh work
```

Тот же алиас работает и в других SSH-командах:

```bash
scp file.txt work:/tmp/
rsync -av ./project/ work:/srv/project/
sftp work
ssh -L 5432:localhost:5432 work
```

Важно: команда называется `ssh`, не `shh`.

## Генерация ключей

Подробнее: [[05 - Генерация SSH-ключа]]

```bash
ssh-keygen -t ed25519 -f ~/.ssh/work_ed25519 -C "work"
ssh-keygen -t rsa -b 4096 -f ~/.ssh/legacy_rsa -C "legacy"
cat ~/.ssh/work_ed25519.pub
```

## Добавление ключа на сервер

Подробнее: [[07 - Добавление публичного ключа на сервер]]

```bash
ssh-copy-id deploy@server.com
ssh-copy-id -p 2222 deploy@server.com
ssh-copy-id -i ~/.ssh/work_ed25519.pub deploy@server.com
```

Ручная проверка на сервере:

```bash
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys
```

## Права на SSH-файлы

Подробнее: [[08 - Права доступа на .ssh и authorized_keys]]

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

## SSH agent

Подробнее: [[12 - SSH agent]]

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-add -d ~/.ssh/id_ed25519
ssh-add -D
ssh-add -t 1h ~/.ssh/id_ed25519
```

## Проверка сервера и `known_hosts`

Подробнее: [[11 - Known hosts и проверка сервера]]

```bash
ssh-keygen -F server.com
ssh-keygen -R server.com
ssh-keygen -R "[server.com]:2222"
ssh-keyscan server.com
ssh-keyscan -p 2222 server.com
```

## Диагностика подключения

Подробнее: [[15 - Диагностика SSH-подключения]] и [[17 - Частые ошибки SSH]]

```bash
ssh -v user@server.com
ssh -vv user@server.com
ssh -vvv user@server.com
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 user@server.com
ssh -vvv -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 -p 2222 deploy@server.com
```

Проверить порт:

```bash
nc -vz server.com 22
nc -vz server.com 2222
```

## Копирование файлов

Подробнее: [[13 - Копирование файлов через scp, rsync и sftp]]

Углублённо:

- [[Linux заметки/ssh/Передача файлов/01 - SCP - быстрое копирование файлов]]
- [[Linux заметки/ssh/Передача файлов/02 - Rsync - синхронизация и деплой]]
- [[Linux заметки/ssh/Передача файлов/03 - SFTP - интерактивная передача файлов]]
- [[Linux заметки/ssh/Передача файлов/04 - Практические сценарии передачи файлов]]

```bash
scp file.txt user@server.com:/tmp/
scp user@server.com:/var/log/app.log .
scp -r ./site user@server.com:/srv/site

rsync -av ./project/ user@server.com:/srv/project/
rsync -avz --progress ./backup/ user@server.com:/backup/
rsync -av --delete ./site/ user@server.com:/srv/site/

sftp user@server.com
```

С готовым алиасом из `~/.ssh/config`:

```bash
scp file.txt work:/tmp/
rsync -av ./project/ work:/srv/project/
sftp work
```

## Port forwarding

Подробнее: [[14 - Port forwarding через SSH]]

Локальный проброс:

```bash
ssh -L 5432:localhost:5432 user@server.com
ssh -L 8080:127.0.0.1:80 user@server.com
```

Удалённый проброс:

```bash
ssh -R 8080:localhost:3000 user@server.com
```

SOCKS-прокси:

```bash
ssh -D 1080 user@server.com
```

Без shell-сессии и в фоне:

```bash
ssh -N -L 5432:localhost:5432 user@server.com
ssh -fN -L 5432:localhost:5432 user@server.com
```

## Каскадный SSH

Подробнее: [[18 - Каскадный SSH]]

```bash
ssh -J jump@jump.example.com target@10.0.0.5
ssh -J jump1@host1,jump2@host2 target@10.0.0.5
```

Через `~/.ssh/config`:

```sshconfig
Host internal
    HostName 10.0.0.5
    User deploy
    ProxyJump jump
```

```bash
ssh internal
```

## Серверный конфиг

Подробнее: [[10 - SSH server config - sshd_config]] и [[16 - Безопасная настройка SSH-сервера]]

```bash
sudo nano /etc/ssh/sshd_config
sudo sshd -t
sudo sshd -T
sudo systemctl reload ssh
sudo systemctl status ssh
sudo journalctl -u ssh -f
```

Если сервис называется `sshd`:

```bash
sudo systemctl reload sshd
sudo systemctl status sshd
sudo journalctl -u sshd -f
```

## Минимум на память

```bash
ssh work
ssh -v work
ssh-copy-id work
scp file.txt work:/tmp/
rsync -av ./dir/ work:/tmp/dir/
ssh -L 5432:localhost:5432 work
```
