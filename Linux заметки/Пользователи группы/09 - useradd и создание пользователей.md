# useradd и создание пользователей

`useradd` - низкоуровневая команда для создания пользователя в Linux.

На Ubuntu/Debian часто удобнее использовать `adduser`, потому что он интерактивный и сразу создаёт домашнюю директорию, спрашивает пароль и дополнительные поля.

Разница коротко:

```text
adduser - удобная интерактивная обёртка
useradd - более низкоуровневая команда, удобна для скриптов
```

## Самый частый вариант useradd

Создать пользователя с домашней директорией и shell:

```bash
sudo useradd -m -s /bin/bash devops
```

Задать пароль:

```bash
sudo passwd devops
```

Проверить:

```bash
id devops
getent passwd devops
ls -ld /home/devops
```

## Что значат опции

```text
-m           создать домашнюю директорию
-s /bin/bash задать login shell
-G group     добавить в дополнительные группы
-g group     задать основную группу
-c "text"    комментарий/gecos, например имя человека
-d /path     задать home directory
-u UID       задать UID вручную
```

## Создать пользователя с sudo

Ubuntu/Debian:

```bash
sudo useradd -m -s /bin/bash -G sudo devops
sudo passwd devops
```

Проверить:

```bash
id devops
groups devops
```

Проверить sudo:

```bash
su - devops
sudo whoami
```

Ожидаемо:

```text
root
```

## Добавить существующего пользователя в группу

```bash
sudo usermod -aG docker devops
sudo usermod -aG sudo devops
```

Важно использовать `-aG`, а не просто `-G`.

```text
-G  задаёт список дополнительных групп
-aG добавляет к существующим группам
```

Если забыть `-a`, можно случайно убрать пользователя из старых дополнительных групп.

## Создать системного пользователя

Для сервисов часто нужен системный пользователь без обычного входа.

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appuser
```

Проверить:

```bash
getent passwd appuser
```

Пример:

```text
appuser:x:999:999::/home/appuser:/usr/sbin/nologin
```

Если нужна отдельная директория приложения:

```bash
sudo mkdir -p /opt/app
sudo chown appuser:appuser /opt/app
```

## Создать пользователя с конкретным home

```bash
sudo useradd -m -d /srv/users/anna -s /bin/bash anna
sudo passwd anna
```

Проверить:

```bash
getent passwd anna
ls -ld /srv/users/anna
```

## Создать пользователя без возможности входа

```bash
sudo useradd -m -s /usr/sbin/nologin backup
```

Такой пользователь может владеть файлами или запускать сервис, но не должен входить в shell.

Проверить доступные shells:

```bash
cat /etc/shells
```

## Удалить пользователя

Удалить пользователя, но оставить home:

```bash
sudo userdel devops
```

Удалить пользователя вместе с home directory и mail spool:

```bash
sudo userdel -r devops
```

Осторожно: `userdel -r` удаляет домашнюю директорию пользователя.

Перед удалением проверить процессы:

```bash
ps -u devops
```

## Заблокировать пользователя

Заблокировать пароль:

```bash
sudo passwd -l devops
```

Разблокировать:

```bash
sudo passwd -u devops
```

Отключить shell:

```bash
sudo usermod -s /usr/sbin/nologin devops
```

## Изменить shell

```bash
sudo usermod -s /bin/bash devops
```

Посмотреть:

```bash
getent passwd devops
```

## Изменить home directory

Перенести home:

```bash
sudo usermod -d /home/newdevops -m devops
```

```text
-d новый путь
-m перенести содержимое старого home
```

## useradd defaults

Посмотреть настройки по умолчанию:

```bash
useradd -D
```

Пример:

```text
GROUP=100
HOME=/home
SHELL=/bin/sh
```

Из-за этих defaults `useradd devops` без `-m -s /bin/bash` может создать пользователя без home и с не тем shell.

## adduser для сравнения

На Ubuntu проще:

```bash
sudo adduser devops
sudo usermod -aG sudo devops
```

`adduser` сам:

- создаёт home;
- спрашивает пароль;
- задаёт shell;
- копирует файлы из `/etc/skel`.

## Частые ошибки

### Создал пользователя, но нет /home/user

Забыли `-m`.

Правильно:

```bash
sudo useradd -m -s /bin/bash user
```

### Пользователь не может sudo

Проверить группы:

```bash
groups user
```

Добавить в sudo:

```bash
sudo usermod -aG sudo user
```

Пользователю нужно перелогиниться.

### Пользователь не может войти

Проверить shell:

```bash
getent passwd user
```

Если shell:

```text
/usr/sbin/nologin
```

то интерактивный вход запрещён.

### Группы не обновились сразу

После `usermod -aG` нужно выйти и зайти заново:

```bash
exit
ssh user@server
```

Или временно:

```bash
newgrp docker
```

## Минимум на память

```bash
sudo useradd -m -s /bin/bash devops
sudo passwd devops
sudo useradd -m -s /bin/bash -G sudo devops
sudo usermod -aG docker devops
id devops
groups devops
getent passwd devops
sudo userdel devops
sudo userdel -r devops
useradd -D
```

## Связанные заметки

- [[01 - Пользователи, группы и обычные права]]
- [[06 - Мини-шпаргалка]]
- [[08 - Как сменить пароль root]]
- [[Linux заметки/Docker/01 - Установка Docker Engine на Ubuntu]]
