# apt через https

`apt через https` - это ситуация, когда репозитории Ubuntu/Debian доступны по `https://`, а не по `http://`.

Современные Ubuntu и Debian обычно уже умеют работать с HTTPS-репозиториями из коробки. Но на старых или минимальных системах может понадобиться пакет:

```bash
sudo apt install -y apt-transport-https ca-certificates
```

## Когда это нужно

- Добавляешь сторонний репозиторий с URL `https://...`.
- `apt update` ругается на HTTPS transport.
- Минимальный Debian/Ubuntu образ без нужных пакетов.
- Настраиваешь Docker, Kubernetes, Zabbix, Grafana, PostgreSQL или другой внешний репозиторий.

## Базовая установка

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg
```

Что за пакеты:

```text
apt-transport-https - поддержка HTTPS transport для старых apt
ca-certificates     - корневые сертификаты для проверки HTTPS
curl                - скачать ключи/файлы репозитория
gnupg               - работа с GPG-ключами репозиториев
```

Важно: в новых версиях `apt` пакет `apt-transport-https` может быть переходным или уже не обязательным. Но поставить его обычно не вредно.

## Проверить sources

Список репозиториев может быть в разных местах:

```bash
cat /etc/apt/sources.list
ls -la /etc/apt/sources.list.d/
```

Посмотреть все строки с репозиториями:

```bash
grep -R "deb " /etc/apt/sources.list /etc/apt/sources.list.d/
```

Пример HTTPS-репозитория:

```text
deb https://download.docker.com/linux/ubuntu noble stable
```

## Где менять http на https для основных репозиториев Ubuntu

В Ubuntu основные репозитории обычно лежат в одном из двух форматов:

```text
/etc/apt/sources.list
/etc/apt/sources.list.d/ubuntu.sources
```

Старые версии чаще используют `sources.list`, новые Ubuntu могут использовать `.sources`-файлы.

## Вариант 1 - старый формат sources.list

Открыть файл:

```bash
sudo nano /etc/apt/sources.list
```

Найти строки вида:

```text
deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu noble-security main restricted universe multiverse
```

Заменить `http://` на `https://`:

```text
deb https://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
deb https://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
deb https://security.ubuntu.com/ubuntu noble-security main restricted universe multiverse
```

Для российских/локальных зеркал адрес может быть другим:

```text
http://ru.archive.ubuntu.com/ubuntu
```

Тогда меняется только схема:

```text
https://ru.archive.ubuntu.com/ubuntu
```

Проверить, какие HTTP-репозитории есть сейчас:

```bash
grep -R "http://.*ubuntu" /etc/apt/sources.list /etc/apt/sources.list.d/
```

Быстрая замена в `/etc/apt/sources.list`:

```bash
sudo sed -i 's|http://archive.ubuntu.com/ubuntu|https://archive.ubuntu.com/ubuntu|g' /etc/apt/sources.list
sudo sed -i 's|http://security.ubuntu.com/ubuntu|https://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list
sudo sed -i 's|http://ru.archive.ubuntu.com/ubuntu|https://ru.archive.ubuntu.com/ubuntu|g' /etc/apt/sources.list
```

После замены:

```bash
sudo apt update
```

## Вариант 2 - новый формат ubuntu.sources

Открыть файл:

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```

Пример до:

```text
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

Пример после:

```text
Types: deb
URIs: https://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: https://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

Быстрая замена:

```bash
sudo sed -i 's|http://archive.ubuntu.com/ubuntu/|https://archive.ubuntu.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources
sudo sed -i 's|http://security.ubuntu.com/ubuntu/|https://security.ubuntu.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources
sudo sed -i 's|http://ru.archive.ubuntu.com/ubuntu/|https://ru.archive.ubuntu.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources
```

После замены:

```bash
sudo apt update
```

## Как понять кодовое имя Ubuntu

Команда:

```bash
lsb_release -cs
```

Примеры:

```text
jammy  - Ubuntu 22.04
noble  - Ubuntu 24.04
```

В репозиториях должно быть кодовое имя именно твоей версии Ubuntu.

## Если после замены apt update не работает

Проверить, поддерживает ли зеркало HTTPS:

```bash
curl -I https://archive.ubuntu.com/ubuntu/
curl -I https://security.ubuntu.com/ubuntu/
```

Если локальное зеркало не поддерживает HTTPS, можно:

- вернуть `http`;
- заменить зеркало на `archive.ubuntu.com`;
- выбрать другое зеркало с HTTPS.

Проверить текущие строки:

```bash
grep -R "URIs:\\|deb " /etc/apt/sources.list /etc/apt/sources.list.d/
```

## Обновить индексы

После установки сертификатов и добавления репозитория:

```bash
sudo apt update
```

Если всё хорошо, `apt` скачает индексы пакетов без ошибок TLS/HTTPS.

## Частые ошибки

### Certificate verification failed

Возможные причины:

- не установлен `ca-certificates`;
- неправильное время на сервере;
- корпоративный proxy подменяет сертификаты;
- старый образ системы.

Проверить время:

```bash
timedatectl status
```

Поставить сертификаты:

```bash
sudo apt install -y ca-certificates
sudo update-ca-certificates
```

### The method driver /usr/lib/apt/methods/https could not be found

На старых системах не установлен HTTPS transport.

Решение:

```bash
sudo apt update
sudo apt install -y apt-transport-https
```

### GPG error / NO_PUBKEY

Это не проблема HTTPS. Это проблема ключа репозитория.

Смотреть нужно в сторону:

```text
/etc/apt/keyrings/
signed-by=
gpg --dearmor
```

## Современный подход к ключам

Для новых инструкций лучше не использовать старый `apt-key`.

Типовой порядок:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Скачать ключ и сохранить в keyring:

```bash
curl -fsSL https://example.com/repo.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/example.gpg
```

Добавить репозиторий с `signed-by`:

```text
deb [signed-by=/etc/apt/keyrings/example.gpg] https://example.com/ubuntu noble main
```

Потом:

```bash
sudo apt update
```

## Минимум на память

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg
sudo update-ca-certificates
grep -R "deb " /etc/apt/sources.list /etc/apt/sources.list.d/
grep -R "URIs:\\|deb " /etc/apt/sources.list /etc/apt/sources.list.d/
sudo apt update
```

## Связанные заметки

- [[02 - Ubuntu Server - базовая настройка]]
- [[03 - Смена hostname в Linux]]
- [[Сети/05 - DNS]]
