## вводная

Кроме обычных прав:

```text
r = read
w = write
x = execute
```

в Linux есть ещё специальные права:

```text
setuid
setgid
sticky bit
```

Они добавляют особое поведение для файлов и директорий.

Посмотреть их можно через:

```bash
ls -l
```

---

## Обычные права для сравнения

Пример:

```bash
-rwxr-xr-x file
```

Разделяем:

```text
rwx   r-x   r-x
user  group others
```

Теперь посмотрим, где появляются специальные права.

---

# setuid

`setuid` применяется к **исполняемым файлам**.

Обычный исполняемый файл:

```bash
-rwxr-xr-x program
```

Файл с `setuid`:

```bash
-rwsr-xr-x program
```

Обрати внимание:

```text
rws
  ^
```

Буква `s` стоит на месте `x` у владельца.

---

## Что делает setuid

Обычно программа запускается от имени пользователя, который её запустил.

Но если у файла стоит `setuid`, программа запускается с правами **владельца файла**.

Пример:

```bash
ls -l /usr/bin/passwd
```

Можно увидеть примерно такое:

```bash
-rwsr-xr-x 1 root root /usr/bin/passwd
```

`passwd` нужен, чтобы менять пароль.

Обычный пользователь не может напрямую менять системные файлы с паролями.  
Но команда `passwd` запускается с правами владельца файла, то есть `root`.

Поэтому пользователь может изменить **свой** пароль.

---

## Как поставить setuid

```bash
chmod u+s program
```

Или числом:

```bash
chmod 4755 program
```

Проверка:

```bash
ls -l program
```

Результат:

```bash
-rwsr-xr-x program
```

---

## Как убрать setuid

```bash
chmod u-s program
```

---

## Важно про setuid

`setuid` может быть опасен.

Особенно если файл принадлежит `root`:

```bash
-rwsr-xr-x 1 root root program
```

Такой файл может выполнять действия с правами `root`.

Поэтому не стоит ставить `setuid` на случайные скрипты и программы.

---

# setgid

`setgid` может работать с файлами и директориями.

Для файлов он похож на `setuid`, но связан с группой.

Для директорий он намного полезнее.

---

## setgid на файле

Обычный файл:

```bash
-rwxr-xr-x program
```

Файл с `setgid`:

```bash
-rwxr-sr-x program
```

Буква `s` стоит на месте `x` у группы:

```text
r-s
  ^
```

Если запустить такой файл, он будет выполняться с правами **группы файла**.

Поставить `setgid` на файл:

```bash
chmod g+s program
```

Или числом:

```bash
chmod 2755 program
```

Убрать:

```bash
chmod g-s program
```

---

# setgid на директории

Вот это используется часто.

`setgid` на директории заставляет новые файлы и папки внутри наследовать **группу директории**.

Пример есть директория проекта:

```bash
mkdir project
```

Меняем группу директории:

```bash
sudo chgrp developers project
```

Ставим `setgid`:

```bash
chmod g+s project
```

Проверяем:

```bash
ls -ld project
```

Результат:

```bash
drwxrwsr-x 2 dima developers project
```

Обрати внимание:

```text
drwxrwsr-x
      ^
```

У группы вместо `x` стоит `s`.

---

## Что это даёт

Допустим, директория принадлежит группе `developers`:

```bash
drwxrwsr-x 2 dima developers project
```

Если пользователь создаст внутри файл:

```bash
touch project/app.py
```

Файл получит группу `developers`:

```bash
-rw-r--r-- 1 dima developers app.py
```

Даже если основная группа пользователя другая.

Это удобно для общих рабочих директорий.

---

## Важный момент

`setgid` на директории наследует **группу**, но не гарантирует нужные права.

Например, новый файл может получиться таким:

```bash
-rw-r--r-- 1 dima developers app.py
```

Группа `developers` есть, но писать в файл группа не может.

Потому что права зависят ещё от `umask`.

Для наследования прав обычно используют ACL.  
Это лучше вынести в отдельную заметку.

---

# sticky bit

`sticky bit` чаще всего используется на директориях.

Он запрещает пользователям удалять чужие файлы внутри общей директории.

Классический пример:

```bash
ls -ld /tmp
```

Результат примерно такой:

```bash
drwxrwxrwt 10 root root /tmp
```

Обрати внимание на последнюю букву:

```text
drwxrwxrwt
         ^
```

Там стоит `t`.

---

## Что делает sticky bit

Обычная директория с правами `777`:

```bash
drwxrwxrwx shared
```

Если у пользователя есть `w` и `x` на директорию, он может удалять файлы внутри.

Даже чужие.

С `sticky bit`:

```bash
drwxrwxrwt shared
```

Пользователь может удалить только:

```text
свой файл
файл, если он владелец директории
файл, если он root
```

---

## Как поставить sticky bit

```bash
chmod +t shared
```

Или числом:

```bash
chmod 1777 shared
```

Проверка:

```bash
ls -ld shared
```

Результат:

```bash
drwxrwxrwt shared
```

---

## Как убрать sticky bit

```bash
chmod -t shared
```

---

# Специальные права в числовой форме

Обычные права:

```text
755
644
700
```

Специальные права добавляются четвёртой цифрой слева.

```text
setuid     = 4
setgid     = 2
sticky bit = 1
```

Примеры:

```bash
chmod 4755 program
```

Значит:

```text
4    setuid
755  обычные права
```

```bash
chmod 2755 program
```

Значит:

```text
2    setgid
755  обычные права
```

```bash
chmod 1777 shared
```

Значит:

```text
1    sticky bit
777  обычные права
```

Можно комбинировать:

```bash
chmod 6755 program
```

Это значит:

```text
4 setuid
2 setgid
755 обычные права
```

---

# Маленькие буквы s и t

Обычно видно так:

```bash
-rwsr-xr-x
```

```text
s = setuid и execute есть
```

```bash
-rwxr-sr-x
```

```text
s = setgid и execute есть
```

```bash
drwxrwxrwt
```

```text
t = sticky bit и execute есть
```

---

# Большие буквы S и T

Иногда можно увидеть `S` или `T`.

Пример:

```bash
-rwSr--r--
```

Это значит:

```text
setuid стоит, но execute у владельца нет
```

Пример:

```bash
-rw-r-Sr--
```

Это значит:

```text
setgid стоит, но execute у группы нет
```

Пример:

```bash
drwxrwxrwT
```

Это значит:

```text
sticky bit стоит, но execute у others нет
```

Обычно большие `S` и `T` — признак ошибки в правах.

---

# Частые команды

Поставить `setuid`:

```bash
chmod u+s program
```

Убрать `setuid`:

```bash
chmod u-s program
```

Поставить `setgid`:

```bash
chmod g+s file_or_dir
```

Убрать `setgid`:

```bash
chmod g-s file_or_dir
```

Поставить `sticky bit`:

```bash
chmod +t dir
```

Убрать `sticky bit`:

```bash
chmod -t dir
```

---

# Частые примеры

## Общая директория проекта

```bash
sudo mkdir /srv/project
sudo chgrp developers /srv/project
sudo chmod 2775 /srv/project
```

Проверка:

```bash
ls -ld /srv/project
```

Результат:

```bash
drwxrwsr-x 2 root developers /srv/project
```

Что это значит:

```text
группа директории — developers
новые файлы внутри получат группу developers
владелец и группа могут работать с директорией
остальные могут читать и заходить
```

---

## Общая временная директория

```bash
sudo mkdir /srv/shared
sudo chmod 1777 /srv/shared
```

Проверка:

```bash
ls -ld /srv/shared
```

Результат:

```bash
drwxrwxrwt 2 root root /srv/shared
```

Что это значит:

```text
все могут создавать файлы
но удалять чужие файлы нельзя
```

---

# Мини-шпаргалка

```text
setuid  = файл запускается с правами владельца файла
setgid  = файл запускается с правами группы файла
setgid на директории = новые файлы наследуют группу директории
sticky bit = нельзя удалять чужие файлы в общей директории
```

Команды:

```bash
chmod u+s file     # setuid
chmod g+s file     # setgid
chmod g+s dir      # setgid на директорию
chmod +t dir       # sticky bit
```

Числа:

```text
4 = setuid
2 = setgid
1 = sticky bit
```

Примеры:

```text
4755 = setuid + rwxr-xr-x
2755 = setgid + rwxr-xr-x
2775 = setgid + rwxrwxr-x
1777 = sticky bit + rwxrwxrwx
```

Главное запомнить:

```text
setuid чаще опасен
setgid на директории полезен для общей работы
sticky bit нужен для общих директорий вроде /tmp
```