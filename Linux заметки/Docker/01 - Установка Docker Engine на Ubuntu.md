# Установка Docker Engine на Ubuntu

Docker Engine - серверная часть Docker для Linux. На Ubuntu Server обычно ставят именно Docker Engine, а не Docker Desktop.

Официальная документация Docker даёт несколько способов установки:

- через официальный apt-репозиторий;
- через `.deb`-пакеты;
- через convenience script `get.docker.com`.

Для лаборатории и быстрого развёртывания удобен официальный скрипт. Для production лучше понимать ручной способ через apt-репозиторий.

## Быстрый вариант - официальный скрипт

Этот вариант удобен для lab-сервера, домашней VM, тестового стенда или быстрого bootstrap.

Скачать скрипт:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

По желанию посмотреть, что будет выполняться:

```bash
less get-docker.sh
```

Запустить:

```bash
sudo sh get-docker.sh
```

Что делает скрипт:

```text
добавляет официальный Docker apt-репозиторий
ставит Docker Engine
ставит Docker CLI
ставит containerd
ставит Buildx
ставит Docker Compose plugin
```

Проверить сервис:

```bash
systemctl status docker
```

Если сервис не запущен:

```bash
sudo systemctl enable --now docker
```

Проверить Docker:

```bash
sudo docker run hello-world
```

Проверить Compose V2:

```bash
docker compose version
```

Важно: команда современного Compose пишется через пробел:

```bash
docker compose
```

Старый вариант через дефис:

```bash
docker-compose
```

может отсутствовать, если установлен только Compose V2 plugin.

## Добавить пользователя в группу docker

Чтобы не писать `sudo docker ...` каждый раз:

```bash
sudo usermod -aG docker $USER
```

Применить группу в текущей сессии:

```bash
newgrp docker
```

Или выйти и зайти заново:

```bash
exit
ssh user@server
```

Проверить:

```bash
groups
docker run hello-world
```

Если `docker run hello-world` работает без `sudo`, пользователь попал в группу `docker`.

Важно по безопасности:

```text
Пользователь в группе docker фактически получает root-возможности на хосте.
```

Не добавляй туда всех подряд.

## Ручной официальный вариант через apt

Этот способ лучше понимать, потому что он прозрачнее скрипта.

Удалить старые конфликтующие пакеты:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y $pkg
done
```

Поставить базовые пакеты:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

Создать директорию для ключей:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Скачать официальный GPG-ключ Docker:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Добавить репозиторий Docker:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Обновить индексы:

```bash
sudo apt update
```

Установить Docker Engine и Compose plugin:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Проверить:

```bash
sudo docker run hello-world
docker compose version
```

## Минимальный compose.yaml

Создать папку проекта:

```bash
mkdir -p ~/docker-test
cd ~/docker-test
```

Создать `compose.yaml`:

```bash
nano compose.yaml
```

Пример:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    restart: unless-stopped
```

Запустить:

```bash
docker compose up -d
```

Проверить:

```bash
docker compose ps
curl http://localhost:8080
```

Посмотреть логи:

```bash
docker compose logs -f
```

Остановить:

```bash
docker compose down
```

## Базовые post-install настройки

Включить Docker при загрузке:

```bash
sudo systemctl enable docker
```

Проверить версии:

```bash
docker version
docker compose version
docker buildx version
```

Проверить место:

```bash
docker system df
```

Посмотреть контейнеры:

```bash
docker ps
docker ps -a
```

## Логи Docker

Сервис Docker:

```bash
journalctl -u docker -f
```

Логи контейнера:

```bash
docker logs container_name
docker logs -f container_name
```

## Где Docker хранит данные

Обычно:

```text
/var/lib/docker
```

Проверить:

```bash
docker info | grep "Docker Root Dir"
```

Важно: если переносишь Docker data-root, сначала останови Docker и сделай backup.

## Firewall и опубликованные порты

Если контейнер опубликован так:

```yaml
ports:
  - "8080:80"
```

то Docker откроет доступ к порту `8080` на хосте.

Проверить:

```bash
ss -tulnp
docker ps
```

Важно: Docker может управлять iptables/nftables и опубликованные порты могут обходить привычную логику `ufw`. На сервере всегда проверяй, какие порты реально слушают наружу.

## Частые ошибки

### Permission denied при docker без sudo

Проверить группы:

```bash
groups
```

Если нет группы `docker`:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Если группа есть, но не работает - перелогиниться.

### docker compose не найден

Проверить:

```bash
docker compose version
dpkg -l | grep docker-compose-plugin
```

Установить plugin:

```bash
sudo apt update
sudo apt install -y docker-compose-plugin
```

### docker-compose работает, а docker compose нет

Значит установлен старый отдельный `docker-compose`, но нет Compose V2 plugin.

Решение:

```bash
sudo apt install -y docker-compose-plugin
```

### Cannot connect to the Docker daemon

Проверить сервис:

```bash
systemctl status docker
sudo systemctl start docker
```

### Порт занят

Проверить:

```bash
sudo ss -tulnp | grep ':8080'
```

Изменить порт в `compose.yaml`, например:

```yaml
ports:
  - "8081:80"
```

## Минимум на память

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
docker compose version
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
systemctl status docker
```

## Связанные заметки

- [[00 - Карта темы]]
- [[Linux заметки/05 - apt через https]]
- [[Systemd заметки/02 - systemctl - управление сервисами]]
- [[Сети/06 - NAT]]
