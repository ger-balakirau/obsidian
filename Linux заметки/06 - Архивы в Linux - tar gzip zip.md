# Архивы в Linux - tar gzip zip

Архивы в Linux встречаются постоянно: backup, логи, исходники, релизы приложений, перенос директорий между серверами.

Важно различать:

```text
tar  - собирает много файлов в один архив
gzip - сжимает один поток/файл
zip  - архивирует и сжимает, часто используется между Windows и Linux
```

Типовые расширения:

```text
.tar      - tar-архив без сжатия
.tar.gz   - tar + gzip
.tgz      - то же самое, что .tar.gz
.tar.xz   - tar + xz
.zip      - zip-архив
.gz       - один файл, сжатый gzip
```

## Посмотреть содержимое архива

`tar`:

```bash
tar -tf archive.tar
tar -tzf archive.tar.gz
tar -tJf archive.tar.xz
```

`zip`:

```bash
unzip -l archive.zip
```

## Распаковать архив

`.tar`:

```bash
tar -xf archive.tar
```

`.tar.gz` / `.tgz`:

```bash
tar -xzf archive.tar.gz
tar -xzf archive.tgz
```

`.tar.xz`:

```bash
tar -xJf archive.tar.xz
```

`.zip`:

```bash
unzip archive.zip
```

`.gz` с одним файлом:

```bash
gunzip file.log.gz
```

Оставить исходный `.gz` и распаковать копию:

```bash
gzip -dk file.log.gz
```

## Распаковать в конкретную папку

`tar`:

```bash
mkdir -p /tmp/app
tar -xzf archive.tar.gz -C /tmp/app
```

`zip`:

```bash
mkdir -p /tmp/app
unzip archive.zip -d /tmp/app
```

## Запаковать файл или папку

Один файл в `.tar.gz`:

```bash
tar -czf file.tar.gz file.txt
```

Папку в `.tar.gz`:

```bash
tar -czf project.tar.gz project/
```

Папку в `.tar.xz`:

```bash
tar -cJf project.tar.xz project/
```

Папку в `.zip`:

```bash
zip -r project.zip project/
```

Один файл через `gzip`:

```bash
gzip file.log
```

Получится:

```text
file.log.gz
```

Сжать, но оставить исходный файл:

```bash
gzip -k file.log
```

## Распаковать один файл из tar-архива

Сначала посмотреть точный путь внутри архива:

```bash
tar -tzf backup.tar.gz
```

Распаковать один файл:

```bash
tar -xzf backup.tar.gz path/inside/file.txt
```

Распаковать в отдельную папку:

```bash
mkdir -p /tmp/restore
tar -xzf backup.tar.gz path/inside/file.txt -C /tmp/restore
```

Важно: путь должен совпадать с тем, что показывает `tar -t`.

## Распаковать одну папку из tar-архива

Посмотреть содержимое:

```bash
tar -tzf backup.tar.gz | less
```

Распаковать папку:

```bash
tar -xzf backup.tar.gz path/inside/folder/
```

В отдельную директорию:

```bash
mkdir -p /tmp/restore
tar -xzf backup.tar.gz path/inside/folder/ -C /tmp/restore
```

## Распаковать определённый вид файлов из tar

Например, только `.conf`:

```bash
tar -xzf backup.tar.gz --wildcards '*.conf'
```

Только `.log` из папки `var/log`:

```bash
tar -xzf backup.tar.gz --wildcards 'var/log/*.log'
```

Только `.service`:

```bash
tar -xzf backup.tar.gz --wildcards '*.service'
```

Если нужно сохранить структуру путей, просто распаковывай как есть. `tar` сохранит директории из архива.

## Распаковать один файл из zip

Посмотреть содержимое:

```bash
unzip -l archive.zip
```

Распаковать один файл:

```bash
unzip archive.zip path/inside/file.txt
```

Распаковать в отдельную папку:

```bash
unzip archive.zip path/inside/file.txt -d /tmp/restore
```

## Распаковать папку из zip

```bash
unzip archive.zip "path/inside/folder/*" -d /tmp/restore
```

Кавычки нужны, чтобы shell не раскрыл `*` раньше времени.

## Распаковать определённый вид файлов из zip

Только `.txt`:

```bash
unzip archive.zip "*.txt" -d /tmp/restore
```

Только `.conf`:

```bash
unzip archive.zip "*.conf" -d /tmp/restore
```

Только файлы из конкретной папки:

```bash
unzip archive.zip "config/*.conf" -d /tmp/restore
```

## Запаковать только определённый вид файлов

Все `.log` в текущей папке:

```bash
tar -czf logs.tar.gz *.log
zip logs.zip *.log
```

Все `.conf` рекурсивно:

```bash
find . -name "*.conf" -print0 | tar --null -czf configs.tar.gz --files-from=-
```

Для zip:

```bash
find . -name "*.conf" -print | zip configs.zip -@
```

## Исключить файлы при упаковке

Исключить `node_modules` и `.git`:

```bash
tar -czf project.tar.gz project/ --exclude='project/node_modules' --exclude='project/.git'
```

Исключить логи:

```bash
tar -czf app.tar.gz app/ --exclude='*.log'
```

Для zip:

```bash
zip -r project.zip project/ -x "project/node_modules/*" "project/.git/*"
```

## Посмотреть архив без распаковки

Прочитать файл из `.tar.gz` в stdout:

```bash
tar -xOzf backup.tar.gz path/inside/file.txt
```

Прочитать файл из `.zip`:

```bash
unzip -p archive.zip path/inside/file.txt
```

Удобно для конфигов:

```bash
tar -xOzf backup.tar.gz etc/nginx/nginx.conf | less
unzip -p archive.zip config/app.conf | less
```

## Проверить размер и место

Размер архива:

```bash
ls -lh archive.tar.gz
du -h archive.tar.gz
```

Размер папки:

```bash
du -sh project/
```

Свободное место:

```bash
df -h
```

## Частые ошибки

### Не тот путь внутри архива

Если команда не распаковывает файл, сначала посмотри точный путь:

```bash
tar -tzf archive.tar.gz | grep file.txt
unzip -l archive.zip | grep file.txt
```

### Permission denied

Если распаковываешь в системную директорию:

```bash
sudo tar -xzf archive.tar.gz -C /
```

Но осторожно: распаковка в `/` может перезаписать системные файлы.

### Архив распаковался в текущую папку и всё смешалось

Перед распаковкой лучше создать отдельную директорию:

```bash
mkdir /tmp/unpack
tar -xzf archive.tar.gz -C /tmp/unpack
```

### unzip не установлен

```bash
sudo apt update
sudo apt install -y unzip zip
```

### xz не установлен

```bash
sudo apt install -y xz-utils
```

## Минимум на память

```bash
tar -czf archive.tar.gz folder/
tar -xzf archive.tar.gz
tar -xzf archive.tar.gz -C /tmp/unpack
tar -tzf archive.tar.gz
tar -xzf archive.tar.gz path/inside/file.txt
tar -xzf archive.tar.gz --wildcards '*.conf'

zip -r archive.zip folder/
unzip archive.zip
unzip archive.zip -d /tmp/unpack
unzip -l archive.zip
unzip archive.zip "*.conf" -d /tmp/unpack

gzip file.log
gzip -dk file.log.gz
```

## Связанные заметки

- [[01 - Linux - базовая навигация, поиск и файлы]]
- [[02 - Ubuntu Server - базовая настройка]]
- [[Linux заметки/ssh/Передача файлов/00 - Карта темы]]
