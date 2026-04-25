# SCP - быстрое копирование файлов

`scp` копирует файлы через SSH. Это простой инструмент для разовой передачи: один файл, несколько файлов или директория.

## Базовый формат

```bash
scp source destination
```

Примеры направлений:

```text
локальный файл -> сервер
сервер -> локальная папка
локальная папка -> сервер
```

## С локального компьютера на сервер

```bash
scp file.txt user@server.com:/home/user/
```

С готовым алиасом из `~/.ssh/config`:

```bash
scp file.txt work:/home/deploy/
```

В конкретную директорию:

```bash
scp app.conf work:/etc/nginx/conf.d/
```

Если директория требует `sudo`, сначала копируй во временное место:

```bash
scp app.conf work:/tmp/
ssh work "sudo mv /tmp/app.conf /etc/nginx/conf.d/"
```

## С сервера на локальный компьютер

```bash
scp user@server.com:/var/log/app.log .
```

С алиасом:

```bash
scp work:/var/log/app.log .
```

Скачать в конкретную локальную папку:

```bash
scp work:/var/log/app.log ./logs/
```

## Копирование директории

Для директории нужен `-r`.

```bash
scp -r ./site work:/srv/site
```

Скачать директорию с сервера:

```bash
scp -r work:/srv/site ./site-backup
```

## Порт и ключ

У `scp` порт указывается через `-P` с большой буквы.

```bash
scp -P 2222 file.txt user@server.com:/tmp/
```

С конкретным ключом:

```bash
scp -i ~/.ssh/work_ed25519 file.txt user@server.com:/tmp/
```

С ключом и портом:

```bash
scp -i ~/.ssh/work_ed25519 -P 2222 file.txt user@server.com:/tmp/
```

Если настроен [[09 - SSH client config]], лучше использовать алиас:

```bash
scp file.txt work:/tmp/
```

## Несколько файлов

```bash
scp file1.txt file2.txt work:/tmp/
```

По маске:

```bash
scp *.log work:/tmp/logs/
```

## Частые ошибки

### `No such file or directory`

Обычно неверный путь локально или на сервере.

Проверить серверный путь:

```bash
ssh work "ls -ld /srv/site"
```

### `Permission denied`

Не хватает прав на запись в целевую директорию.

Вариант:

```bash
scp file.txt work:/tmp/
ssh work "sudo mv /tmp/file.txt /protected/path/"
```

### Неправильный порт

Для `ssh` порт пишется `-p`, для `scp` - `-P`.

```bash
ssh -p 2222 user@server.com
scp -P 2222 file.txt user@server.com:/tmp/
```

## Главное запомнить

- `scp file server:path` - загрузить на сервер.
- `scp server:path .` - скачать с сервера.
- `scp -r` - копировать директорию.
- `scp -P` - указать порт.
- Для повторяемой синхронизации лучше `rsync`.
