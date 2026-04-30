# Docker объекты - создание и очистка

Docker хранит несколько типов объектов:

```text
containers - контейнеры
images     - образы
volumes    - постоянные данные
networks   - docker-сети
build cache - кэш сборки
```

Главное правило:

```text
контейнер можно пересоздать
volume может содержать важные данные
```

Поэтому чистить `volumes` нужно осторожнее всего.

## Посмотреть, что есть в Docker

Контейнеры:

```bash
docker ps
docker ps -a
```

Образы:

```bash
docker images
```

Volumes:

```bash
docker volume ls
```

Networks:

```bash
docker network ls
```

Сколько места занимает Docker:

```bash
docker system df
```

Подробно:

```bash
docker system df -v
```

## Создать и запустить контейнер

Простой запуск:

```bash
docker run nginx:alpine
```

Запустить в фоне:

```bash
docker run -d nginx:alpine
```

Запустить с именем:

```bash
docker run -d --name web nginx:alpine
```

Опубликовать порт:

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

Проверить:

```bash
docker ps
curl http://localhost:8080
```

## Остановить, запустить, перезапустить контейнер

```bash
docker stop web
docker start web
docker restart web
```

Посмотреть логи:

```bash
docker logs web
docker logs -f web
```

Зайти внутрь контейнера:

```bash
docker exec -it web sh
```

Для Debian/Ubuntu-based контейнеров часто:

```bash
docker exec -it container_name bash
```

## Удалить контейнер

Сначала остановить:

```bash
docker stop web
```

Удалить:

```bash
docker rm web
```

Принудительно удалить работающий контейнер:

```bash
docker rm -f web
```

Удалить все остановленные контейнеры:

```bash
docker container prune
```

Перед удалением Docker спросит подтверждение.

Без подтверждения:

```bash
docker container prune -f
```

## Образы

Скачать образ:

```bash
docker pull nginx:alpine
```

Посмотреть образы:

```bash
docker images
```

Удалить образ:

```bash
docker rmi nginx:alpine
```

Если образ используется контейнером, сначала удали контейнер.

Удалить неиспользуемые образы:

```bash
docker image prune
```

Удалить все неиспользуемые образы, не только dangling:

```bash
docker image prune -a
```

Осторожно: `-a` удалит все образы, которые сейчас не используются контейнерами. Потом их придётся скачивать заново.

## Собрать свой образ

Пример `Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

Сборка:

```bash
docker build -t my-nginx:1.0 .
```

Запуск:

```bash
docker run -d --name my-web -p 8080:80 my-nginx:1.0
```

Посмотреть историю образа:

```bash
docker history my-nginx:1.0
```

## Volumes

Volume нужен, чтобы данные жили отдельно от контейнера.

Создать volume:

```bash
docker volume create app-data
```

Посмотреть:

```bash
docker volume ls
docker volume inspect app-data
```

Запустить контейнер с volume:

```bash
docker run -d \
  --name web-with-volume \
  -p 8080:80 \
  -v app-data:/usr/share/nginx/html \
  nginx:alpine
```

Удалить volume:

```bash
docker volume rm app-data
```

Если volume используется контейнером, сначала останови и удали контейнер.

Удалить неиспользуемые volumes:

```bash
docker volume prune
```

Осторожно: volume может содержать базу данных, файлы приложения, uploads и другие важные данные.

## Bind mount

Bind mount подключает папку хоста в контейнер.

```bash
mkdir -p ~/site
echo "hello" > ~/site/index.html

docker run -d \
  --name web-bind \
  -p 8080:80 \
  -v ~/site:/usr/share/nginx/html:ro \
  nginx:alpine
```

`:ro` значит read-only.

Плюс bind mount:

```text
удобно для конфигов и разработки
```

Минус:

```text
контейнер зависит от конкретного пути на хосте
```

## Networks

Создать bridge-сеть:

```bash
docker network create app-net
```

Посмотреть сети:

```bash
docker network ls
docker network inspect app-net
```

Запустить контейнер в сети:

```bash
docker run -d --name web --network app-net nginx:alpine
```

Подключить существующий контейнер к сети:

```bash
docker network connect app-net web
```

Отключить:

```bash
docker network disconnect app-net web
```

Удалить сеть:

```bash
docker network rm app-net
```

Удалить неиспользуемые сети:

```bash
docker network prune
```

## Docker Compose - создать всё из compose.yaml

Пример `compose.yaml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - site-data:/usr/share/nginx/html
    networks:
      - app-net
    restart: unless-stopped

volumes:
  site-data:

networks:
  app-net:
```

Запустить:

```bash
docker compose up -d
```

Посмотреть:

```bash
docker compose ps
docker compose logs -f
```

Остановить без удаления:

```bash
docker compose stop
```

Запустить снова:

```bash
docker compose start
```

Остановить и удалить контейнеры/сети проекта:

```bash
docker compose down
```

Удалить ещё и volumes проекта:

```bash
docker compose down -v
```

Осторожно: `down -v` удаляет volumes, созданные compose-проектом. Если там база данных, данные исчезнут.

Пересоздать после изменения compose:

```bash
docker compose up -d --force-recreate
```

Скачать новые версии образов:

```bash
docker compose pull
docker compose up -d
```

## Очистка build cache

Посмотреть место:

```bash
docker system df
```

Удалить неиспользуемый build cache:

```bash
docker builder prune
```

Без подтверждения:

```bash
docker builder prune -f
```

Более агрессивно:

```bash
docker builder prune -a
```

## Общая очистка Docker

Удалить:

- остановленные контейнеры;
- неиспользуемые networks;
- dangling images;
- build cache.

```bash
docker system prune
```

Без подтверждения:

```bash
docker system prune -f
```

Более агрессивно: удалить ещё все неиспользуемые images.

```bash
docker system prune -a
```

Ещё агрессивнее: удалить также volumes.

```bash
docker system prune -a --volumes
```

Очень осторожно:

```text
docker system prune -a --volumes
```

может удалить данные приложений, если они лежат в неиспользуемых volumes.

## Безопасный порядок очистки

1. Посмотреть, что занимает место:

```bash
docker system df -v
```

2. Посмотреть контейнеры:

```bash
docker ps -a
```

3. Остановить ненужное:

```bash
docker stop container_name
```

4. Удалить ненужные контейнеры:

```bash
docker rm container_name
```

5. Осторожно удалить неиспользуемые образы:

```bash
docker image prune
```

6. Volumes удалять только если понимаешь, что внутри:

```bash
docker volume ls
docker volume inspect volume_name
```

## Практический пример полной пересборки compose-проекта

В папке проекта:

```bash
docker compose down
docker compose pull
docker compose up -d
docker compose ps
docker compose logs -f
```

Если нужно пересобрать локальный образ:

```bash
docker compose build --no-cache
docker compose up -d
```

Если нужно полностью удалить проект вместе с данными:

```bash
docker compose down -v
```

Перед `down -v` спросить себя:

```text
Точно ли в volume нет базы данных или нужных файлов?
```

## Что нельзя делать бездумно

```bash
docker system prune -a --volumes
docker volume prune
docker compose down -v
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
```

Эти команды могут быстро освободить место, но так же быстро удалить нужные данные.

## Минимум на память

```bash
docker ps
docker ps -a
docker images
docker volume ls
docker network ls
docker system df

docker run -d --name web -p 8080:80 nginx:alpine
docker stop web
docker rm web
docker rmi nginx:alpine

docker compose up -d
docker compose ps
docker compose logs -f
docker compose down

docker container prune
docker image prune
docker network prune
docker builder prune
docker system prune
```

## Связанные заметки

- [[00 - Карта темы]]
- [[01 - Установка Docker Engine на Ubuntu]]
- [[Systemd заметки/04 - journalctl - просмотр логов]]
- [[Сети/06 - NAT]]
