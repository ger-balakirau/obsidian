## Вводная 

В Linux файл — это не просто имя.

У файла есть:

```text
имя
inode
данные на диске
```

Важно понимать:

```text
имя файла — это ссылка на inode
inode — это объект файловой системы, где хранится информация о файле
```

---

## Что такое inode

`inode` — это внутренняя запись о файле.

В inode хранится:

```text
права доступа
владелец
группа
размер
время изменения
ссылки на данные файла
количество hard links
```

Имя файла хранится отдельно.

Посмотреть inode можно так:

```bash
ls -i
```

Пример:

```bash
123456 file.txt
```

Здесь:

```text
123456   inode
file.txt имя файла
```

Более подробно:

```bash
stat file.txt
```

Пример:

```bash
stat file.txt
```

Можно увидеть:

```text
Inode: 123456
Links: 1
Access: (0644/-rw-r--r--)
Uid: dima
Gid: dima
```

---

## Простая схема

Обычно мы думаем так:

```text
file.txt -> данные файла
```

На самом деле ближе к этому:

```text
file.txt -> inode -> данные файла
```

Файл можно переименовать:

```bash
mv file.txt notes.txt
```

Но inode останется тем же.

Проверка:

```bash
ls -i notes.txt
```

---

## Hard link

`hard link` — это ещё одно имя для того же inode.

Создать hard link:

```bash
ln file.txt hard.txt
```

Проверить:

```bash
ls -li
```

Пример:

```bash
123456 -rw-r--r-- 2 dima dima file.txt
123456 -rw-r--r-- 2 dima dima hard.txt
```

Обрати внимание:

```text
inode одинаковый
количество ссылок = 2
```

Это значит:

```text
file.txt и hard.txt — два имени одного и того же файла
```

---

## Что будет при изменении hard link

Если изменить `hard.txt`:

```bash
echo "hello" >> hard.txt
```

То изменится и `file.txt`.

Почему?

Потому что это один и тот же inode:

```text
file.txt -> inode 123456 -> данные
hard.txt -> inode 123456 -> те же данные
```

Это не копия файла.

---

## Что будет при удалении hard link

Удалим оригинальное имя:

```bash
rm file.txt
```

Файл не исчезнет, если есть другой hard link:

```bash
cat hard.txt
```

Данные останутся доступны.

Почему?

Потому что удалилось только одно имя:

```text
file.txt удалён
hard.txt всё ещё указывает на тот же inode
```

Данные удалятся только тогда, когда не останется ни одного hard link.

---

## Hard link — важные ограничения

Обычно hard link нельзя создать на директорию:

```bash
ln dir hard_dir
```

Будет ошибка.

Также hard link нельзя создать между разными файловыми системами.

Например:

```bash
ln /home/dima/file.txt /mnt/usb/hard.txt
```

Может не сработать, если `/home` и `/mnt/usb` — разные файловые системы.

Почему?

Потому что inode существует внутри одной файловой системы.

---

# Symlink

`symlink` — это символическая ссылка.

Она похожа на ярлык.

Создать symlink:

```bash
ln -s file.txt link.txt
```

Проверить:

```bash
ls -l
```

Пример:

```bash
lrwxrwxrwx 1 dima dima link.txt -> file.txt
-rw-r--r-- 1 dima dima file.txt
```

Обрати внимание:

```text
link.txt -> file.txt
```

Symlink указывает на путь, а не на inode напрямую.

---

## Symlink имеет свой inode

Проверим:

```bash
ls -li
```

Пример:

```bash
123456 -rw-r--r-- 1 dima dima file.txt
654321 lrwxrwxrwx 1 dima dima link.txt -> file.txt
```

Здесь:

```text
file.txt имеет inode 123456
link.txt имеет inode 654321
```

Symlink — это отдельный объект файловой системы.

---

## Что будет при изменении файла через symlink

Если написать:

```bash
echo "hello" >> link.txt
```

На самом деле изменится `file.txt`.

Потому что `link.txt` ведёт к `file.txt`.

Схема:

```text
link.txt -> file.txt -> inode -> данные
```

---

## Что будет при удалении оригинала

Есть файл и ссылка:

```bash
file.txt
link.txt -> file.txt
```

Удалим файл:

```bash
rm file.txt
```

Symlink останется:

```bash
ls -l
```

Будет:

```bash
link.txt -> file.txt
```

Но ссылка станет битой.

Попытка открыть:

```bash
cat link.txt
```

Даст ошибку:

```text
No such file or directory
```

Почему?

Потому что symlink указывает на путь `file.txt`, а такого пути уже нет.

---

## Битая ссылка

Битая ссылка — это symlink, цель которого не существует.

Пример:

```bash
link.txt -> missing.txt
```

Проверить можно так:

```bash
ls -l
```

Или:

```bash
readlink link.txt
```

Команда покажет, куда указывает ссылка:

```bash
file.txt
```

---

## Symlink на директорию

Symlink можно делать на директории.

Пример:

```bash
ln -s /var/log logs
```

Проверка:

```bash
ls -l
```

Результат:

```bash
logs -> /var/log
```

Теперь можно зайти так:

```bash
cd logs
```

На самом деле ты попадёшь в:

```bash
/var/log
```

---

## Symlink может указывать на другой диск

Symlink может указывать на файл или директорию в другой файловой системе.

Пример:

```bash
ln -s /mnt/usb/file.txt usb-file.txt
```

Это нормально.

Symlink хранит путь, поэтому ему не важно, где находится цель.

---

# Главное отличие hard link и symlink

## Hard link

```text
hard link указывает на inode
```

Схема:

```text
file.txt -> inode 123456 -> данные
hard.txt -> inode 123456 -> те же данные
```

Если удалить `file.txt`, `hard.txt` продолжит работать.

---

## Symlink

```text
symlink указывает на путь
```

Схема:

```text
link.txt -> file.txt -> inode 123456 -> данные
```

Если удалить `file.txt`, `link.txt` сломается.

---

# Пример руками

Создадим файл:

```bash
echo "hello" > original.txt
```

Посмотрим inode:

```bash
ls -li original.txt
```

Пример:

```bash
123456 -rw-r--r-- 1 dima dima original.txt
```

Создадим hard link:

```bash
ln original.txt hard.txt
```

Создадим symlink:

```bash
ln -s original.txt soft.txt
```

Проверим:

```bash
ls -li
```

Пример:

```bash
123456 -rw-r--r-- 2 dima dima original.txt
123456 -rw-r--r-- 2 dima dima hard.txt
654321 lrwxrwxrwx 1 dima dima soft.txt -> original.txt
```

Что видно:

```text
original.txt и hard.txt имеют одинаковый inode
soft.txt имеет другой inode
soft.txt указывает на путь original.txt
```

---

## Удалим original.txt

```bash
rm original.txt
```

Проверим hard link:

```bash
cat hard.txt
```

Работает.

Проверим symlink:

```bash
cat soft.txt
```

Не работает.

Почему:

```text
hard.txt всё ещё указывает на inode
soft.txt указывает на имя original.txt, которого больше нет
```

---

# Когда использовать hard link

Hard link полезен, когда нужно второе имя для того же файла.

Например:

```text
один и тот же файл должен быть доступен из двух мест
файл не должен исчезнуть после удаления одного имени
нужно избежать копирования данных
```

Но hard links используют осторожно, потому что легко забыть, что два имени — это один файл.

---

# Когда использовать symlink

Symlink используется чаще.

Например:

```text
сделать короткий путь к длинной директории
сослаться на файл в другом месте
сделать ссылку на директорию
переключать версии программ или конфигов
```

Примеры:

```bash
ln -s /etc/nginx/sites-available/site.conf site.conf
```

```bash
ln -s /opt/app/releases/v2 current
```

---

# Важные команды

Показать inode:

```bash
ls -i
```

Показать inode и права:

```bash
ls -li
```

Показать подробную информацию:

```bash
stat file.txt
```

Создать hard link:

```bash
ln file.txt hard.txt
```

Создать symlink:

```bash
ln -s file.txt link.txt
```

Показать, куда указывает symlink:

```bash
readlink link.txt
```

Удалить ссылку:

```bash
rm link.txt
```

Важно:

```text
rm link.txt удаляет саму ссылку
не файл, на который она указывает
```

---

# Частая путаница

## Symlink показывает права `lrwxrwxrwx`

Пример:

```bash
lrwxrwxrwx 1 dima dima link.txt -> file.txt
```

Это не значит, что все могут читать и писать в целевой файл.

Реальные права проверяются у файла, на который ведёт ссылка.

Например:

```bash
lrwxrwxrwx link.txt -> secret.txt
-rw------- secret.txt
```

Если пользователь не имеет прав читать `secret.txt`, то через `link.txt` он тоже не сможет его прочитать.

---

## Удаление symlink

Если есть:

```bash
link.txt -> file.txt
```

Команда:

```bash
rm link.txt
```

Удалит только symlink.

Файл `file.txt` останется.

---

## Удаление файла через symlink

Обычный `rm link.txt` удаляет ссылку, а не цель.

Но если программа открывает symlink для записи, она может изменить целевой файл.

Пример:

```bash
echo "new text" > link.txt
```

Это перезапишет файл, на который указывает `link.txt`.

---

# Мини-шпаргалка

```text
inode = внутренняя запись о файле
имя файла = ссылка на inode
hard link = ещё одно имя того же inode
symlink = отдельная ссылка на путь
```

Команды:

```bash
ls -i                 # показать inode
ls -li                # показать inode и права
stat file             # подробная информация

ln file hard          # создать hard link
ln -s file link       # создать symlink
readlink link         # показать цель symlink
rm link               # удалить ссылку
```

Главное запомнить:

```text
hard link не ломается при удалении оригинального имени
symlink ломается, если удалить или переместить цель
hard link обычно нельзя делать на директории
symlink можно делать на директории
hard link работает только внутри одной файловой системы
symlink может указывать куда угодно по пути
```