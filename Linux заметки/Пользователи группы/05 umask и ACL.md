## Вводная

В Linux права новых файлов и директорий зависят не только от `chmod`.

Есть ещё:

```text
umask
ACL
```

`umask` задаёт стандартные права для новых файлов и директорий.

`ACL` позволяет задавать более гибкие права, чем обычные `user/group/others`.

---

# umask

`umask` показывает, какие права будут **убраны** у новых файлов и директорий.

Посмотреть текущий `umask`:

```bash
umask
```

Пример:

```text
0022
```

Чаще пишут коротко:

```text
022
```

---

## Как работает umask

У новых объектов есть максимальные права.

Для файлов:

```text
666
```

Для директорий:

```text
777
```

Почему файл не `777`?

Потому что обычные файлы не создаются сразу исполняемыми.

```text
файл      максимум 666 = rw-rw-rw-
директория максимум 777 = rwxrwxrwx
```

`umask` вычитает права из этих максимумов.

---

## Пример umask 022

```bash
umask 022
```

Новый файл:

```text
666 - 022 = 644
```

Права:

```bash
-rw-r--r--
```

Новая директория:

```text
777 - 022 = 755
```

Права:

```bash
drwxr-xr-x
```

То есть:

```text
владелец может читать и писать
остальные могут только читать

для директорий:
владелец может всё
остальные могут входить и смотреть
```

---

## Пример umask 002

```bash
umask 002
```

Новый файл:

```text
666 - 002 = 664
```

Права:

```bash
-rw-rw-r--
```

Новая директория:

```text
777 - 002 = 775
```

Права:

```bash
drwxrwxr-x
```

Это удобно для работы в группе.

```text
владелец и группа могут писать
остальные могут только читать
```

---

## Пример umask 077

```bash
umask 077
```

Новый файл:

```text
666 - 077 = 600
```

Права:

```bash
-rw-------
```

Новая директория:

```text
777 - 077 = 700
```

Права:

```bash
drwx------
```

Это приватный вариант.

```text
только владелец имеет доступ
группа и остальные не имеют прав
```

---

# Частые значения umask

```text
022 = обычный вариант
002 = удобно для общей работы в группе
077 = приватный вариант
```

Таблица:

```text
umask 022:
file = 644
dir  = 755

umask 002:
file = 664
dir  = 775

umask 077:
file = 600
dir  = 700
```

---

# Временно изменить umask

Команда:

```bash
umask 002
```

После этого новые файлы и директории в текущей shell-сессии будут создаваться с учётом этого `umask`.

Пример:

```bash
umask 002
touch file.txt
mkdir project
ls -l
```

Результат будет примерно такой:

```bash
-rw-rw-r-- file.txt
drwxrwxr-x project
```

---

# Где задаётся umask

`umask` может задаваться в разных местах:

```text
/etc/profile
/etc/bash.bashrc
~/.profile
~/.bashrc
systemd unit
настройки shell
```

Для пользователя часто используют:

```bash
~/.profile
```

или:

```bash
~/.bashrc
```

Пример строки:

```bash
umask 022
```

---

# Важный момент про umask

`umask` влияет только на **новые** файлы и директории.

Он не меняет уже существующие файлы.

Пример:

```bash
umask 077
```

После этого старые файлы не изменятся.

Чтобы изменить права старого файла, нужен `chmod`:

```bash
chmod 600 file.txt
```

---

# ACL

Обычные права Linux делятся на:

```text
user
group
others
```

Но иногда этого мало.

Например:

```text
владелец файла — dima
группа файла — developers
но нужно дать отдельные права пользователю anna
```

Для этого используют `ACL`.

ACL = Access Control List.

---

## Проверить ACL

Команда:

```bash
getfacl file.txt
```

Пример:

```bash
getfacl file.txt
```

Вывод:

```text
# file: file.txt
# owner: dima
# group: developers
user::rw-
group::r--
other::r--
```

Это обычные права без дополнительных ACL.

---

## Добавить права конкретному пользователю

Дать пользователю `anna` права читать и писать файл:

```bash
setfacl -m u:anna:rw file.txt
```

Проверить:

```bash
getfacl file.txt
```

Пример:

```text
user::rw-
user:anna:rw-
group::r--
mask::rw-
other::r--
```

Теперь `anna` имеет отдельные права на файл.

---

## Добавить права группе

Дать группе `developers` права читать и писать:

```bash
setfacl -m g:developers:rw file.txt
```

Проверить:

```bash
getfacl file.txt
```

Пример:

```text
user::rw-
group::r--
group:developers:rw-
mask::rw-
other::r--
```

---

## Убрать ACL для пользователя

Убрать отдельные права пользователя `anna`:

```bash
setfacl -x u:anna file.txt
```

---

## Убрать ACL для группы

Убрать отдельные права группы `developers`:

```bash
setfacl -x g:developers file.txt
```

---

## Убрать все дополнительные ACL

```bash
setfacl -b file.txt
```

После этого останутся обычные права:

```text
user
group
other
```

---

# ACL на директории

ACL можно ставить и на директории.

Пример:

```bash
setfacl -m u:anna:rwx project
```

Теперь пользователь `anna` может работать с директорией `project`.

Проверка:

```bash
getfacl project
```

---

## Важно про директории

Для директории:

```text
r = видеть список файлов
w = создавать, удалять, переименовывать
x = входить в директорию
```

Поэтому для полноценной работы с директорией обычно нужно:

```text
rwx
```

Пример:

```bash
setfacl -m u:anna:rwx project
```

---

# Default ACL

Обычный ACL даёт права на саму директорию.

Но новые файлы внутри не всегда наследуют эти права.

Для наследования используют `default ACL`.

Пример:

```bash
setfacl -d -m g:developers:rwx project
```

Здесь:

```text
-d = default
-m = modify
g:developers:rwx = права для группы developers
project = директория
```

Теперь новые файлы и директории внутри `project` будут получать ACL для группы `developers`.

---

## Пример default ACL

Создаём директорию:

```bash
mkdir project
```

Ставим права для группы:

```bash
setfacl -m g:developers:rwx project
```

Ставим наследуемые права:

```bash
setfacl -d -m g:developers:rwx project
```

Проверяем:

```bash
getfacl project
```

Пример вывода:

```text
# file: project
# owner: dima
# group: dima
user::rwx
group::r-x
group:developers:rwx
mask::rwx
other::r-x
default:user::rwx
default:group::r-x
default:group:developers:rwx
default:mask::rwx
default:other::r-x
```

Строка:

```text
group:developers:rwx
```

даёт права на саму директорию.

Строка:

```text
default:group:developers:rwx
```

задаёт права для новых объектов внутри.

---

# setgid + default ACL

Для общей директории часто используют вместе:

```text
setgid
default ACL
```

`setgid` нужен, чтобы новые файлы наследовали группу директории.

`default ACL` нужен, чтобы новые файлы наследовали права.

Пример:

```bash
sudo mkdir /srv/project
sudo chgrp developers /srv/project
sudo chmod 2775 /srv/project
sudo setfacl -d -m g:developers:rwx /srv/project
```

Проверка:

```bash
ls -ld /srv/project
getfacl /srv/project
```

Что это даёт:

```text
новые файлы будут получать группу developers
новые файлы будут получать ACL для группы developers
```

---

# Важный момент про файлы и x

Даже если default ACL задан как `rwx`, обычные новые файлы часто не получают `x`.

Пример:

```text
default ACL: rwx
новый файл: rw-
новая директория: rwx
```

Это нормально.

Файлы не создаются исполняемыми автоматически.

Если это скрипт, нужно отдельно:

```bash
chmod +x script.sh
```

---

# mask в ACL

В выводе `getfacl` часто есть строка:

```text
mask::rw-
```

`mask` ограничивает максимальные права для:

```text
группы файла
именованных пользователей
именованных групп
```

Пример:

```text
user:anna:rwx
mask::rw-
```

На первый взгляд у `anna` есть `rwx`.

Но из-за mask реально будет только:

```text
rw-
```

Потому что `x` закрыт маской.

---

## Изменить mask

```bash
setfacl -m m:rwx file.txt
```

Проверить:

```bash
getfacl file.txt
```

Обычно `setfacl` сам обновляет mask, но иногда полезно знать, почему права не работают как ожидается.

---

# ACL и ls -l

Если у файла есть ACL, в `ls -l` появляется `+`.

Пример:

```bash
ls -l file.txt
```

Результат:

```bash
-rw-rw-r--+ 1 dima developers file.txt
```

Плюс в конце прав:

```text
+
```

означает:

```text
у файла есть дополнительные ACL
```

Чтобы посмотреть их:

```bash
getfacl file.txt
```

---

# Частые команды ACL

Показать ACL:

```bash
getfacl file.txt
```

Дать права пользователю:

```bash
setfacl -m u:anna:rw file.txt
```

Дать права группе:

```bash
setfacl -m g:developers:rw file.txt
```

Дать права пользователю на директорию:

```bash
setfacl -m u:anna:rwx project
```

Задать наследуемые права для группы:

```bash
setfacl -d -m g:developers:rwx project
```

Убрать ACL пользователя:

```bash
setfacl -x u:anna file.txt
```

Убрать все дополнительные ACL:

```bash
setfacl -b file.txt
```

---

# Частый пример: общая директория для группы

Задача:

```text
есть группа developers
все участники группы должны работать в /srv/project
новые файлы должны оставаться в группе developers
новые файлы должны быть доступны группе
```

Команды:

```bash
sudo mkdir /srv/project
sudo chgrp developers /srv/project
sudo chmod 2775 /srv/project
sudo setfacl -d -m g:developers:rwx /srv/project
```

Проверка:

```bash
ls -ld /srv/project
getfacl /srv/project
```

Ожидаемо:

```bash
drwxrwsr-x+ root developers /srv/project
```

Что важно:

```text
s в правах группы = setgid
+ в конце прав = есть ACL
```

---

# umask или ACL

`umask` подходит, когда нужно задать стандартные права для новых файлов пользователя.

ACL подходит, когда нужно дать права конкретному пользователю или группе.

```text
umask = общие правила создания файлов
ACL = точные права для конкретных пользователей и групп
```

Примеры:

```text
umask 077
```

Подходит для приватной работы.

```text
setfacl -m u:anna:rw file.txt
```

Подходит, если нужно дать доступ одному пользователю.

```text
setfacl -d -m g:developers:rwx project
```

Подходит, если нужно наследование прав в директории.

---

# Мини-шпаргалка

```text
umask 022 = файлы 644, директории 755
umask 002 = файлы 664, директории 775
umask 077 = файлы 600, директории 700
```

Команды:

```bash
umask                  # посмотреть umask
umask 022              # задать umask

getfacl file           # посмотреть ACL
setfacl -m u:user:rw file
setfacl -m g:group:rw file
setfacl -d -m g:group:rwx dir

setfacl -x u:user file # убрать ACL пользователя
setfacl -b file        # убрать все дополнительные ACL
```

Главное запомнить:

```text
umask убирает права у новых файлов и директорий
ACL даёт более точные права, чем user/group/others
setgid наследует группу
default ACL наследует права
```