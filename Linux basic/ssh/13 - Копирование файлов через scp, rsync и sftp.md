# 1. Передача файлов через SCP

`scp` — это копирование файлов через SSH.

## С локального компьютера на сервер

```bash
scp file.txt user@server.com:/home/user/
```

С указанием ключа:

```bash
scp -i ~/.ssh/id_ed25519 file.txt user@server.com:/home/user/
```

С указанием порта:

```bash
scp -P 2222 file.txt user@server.com:/home/user/
```

Важно: у `scp` порт указывается через `-P` с большой буквы.

С ключом и портом:

```bash
scp -i ~/.ssh/id_ed25519 -P 2222 file.txt user@server.com:/home/user/
```

---

## С сервера на локальный компьютер

```bash
scp user@server.com:/home/user/file.txt .
```

С ключом:

```bash
scp -i ~/.ssh/id_ed25519 user@server.com:/home/user/file.txt .
```

С портом:

```bash
scp -P 2222 user@server.com:/home/user/file.txt .
```

---

## Копирование папки через SCP

Для папок используется `-r`.

```bash
scp -r ./my-folder user@server.com:/home/user/
```

С ключом и портом:

```bash
scp -i ~/.ssh/id_ed25519 -P 2222 -r ./my-folder user@server.com:/home/user/
```

---

# 2. Передача файлов через RSYNC

`rsync` удобен для синхронизации файлов и папок.

## С локального компьютера на сервер

```bash
rsync -av ./project/ user@server.com:/var/www/project/
```

С ключом:

```bash
rsync -av -e "ssh -i ~/.ssh/id_ed25519" ./project/ user@server.com:/var/www/project/
```

С портом:

```bash
rsync -av -e "ssh -p 2222" ./project/ user@server.com:/var/www/project/
```

С ключом и портом:

```bash
rsync -av -e "ssh -i ~/.ssh/id_ed25519 -p 2222" ./project/ user@server.com:/var/www/project/
```

---

## С сервера на локальный компьютер

```bash
rsync -av user@server.com:/var/www/project/ ./project/
```

С ключом и портом:

```bash
rsync -av -e "ssh -i ~/.ssh/id_ed25519 -p 2222" user@server.com:/var/www/project/ ./project/
```

---

## Удаление лишних файлов при синхронизации

```bash
rsync -av --delete ./project/ user@server.com:/var/www/project/
```

Осторожно: `--delete` удаляет на сервере файлы, которых нет локально.

---

# 3. SFTP через консоль

SFTP — интерактивная передача файлов через SSH.

```bash
sftp user@server.com
```

С ключом:

```bash
sftp -i ~/.ssh/id_ed25519 user@server.com
```

С портом:

```bash
sftp -P 2222 user@server.com
```

Команды внутри SFTP:

```text
ls       — список файлов на сервере
lls      — список локальных файлов
pwd      — текущая папка на сервере
lpwd     — текущая локальная папка
cd       — перейти в папку на сервере
lcd      — перейти в локальную папку
put      — загрузить файл на сервер
get      — скачать файл с сервера
mkdir    — создать папку на сервере
rm       — удалить файл на сервере
exit     — выйти
```

Пример загрузки файла:

```text
put file.txt
```

Пример скачивания файла:

```text
get file.txt
```

---
