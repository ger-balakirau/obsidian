# Rsync - синхронизация и деплой

`rsync` передаёт только изменения. Это удобнее, чем `scp`, когда нужно регулярно синхронизировать директории, деплоить проект или копировать много файлов.

## Базовый формат

```bash
rsync [options] source destination
```

Частый набор:

```bash
rsync -av ./project/ work:/srv/project/
```

Что значит:

- `-a` - archive mode: сохраняет права, времена, директории и ссылки;
- `-v` - подробный вывод;
- `/` в конце `./project/` означает “содержимое директории”, а не саму директорию.

## Локальная директория на сервер

```bash
rsync -av ./project/ work:/srv/project/
```

С прогрессом:

```bash
rsync -av --progress ./project/ work:/srv/project/
```

С компрессией:

```bash
rsync -avz ./project/ work:/srv/project/
```

## Сервер на локальный компьютер

```bash
rsync -av work:/srv/project/ ./project/
```

Скачать логи:

```bash
rsync -av work:/var/log/nginx/ ./nginx-logs/
```

## Проверка без изменений

Перед опасной синхронизацией полезно сделать dry run:

```bash
rsync -av --dry-run ./project/ work:/srv/project/
```

Коротко:

```bash
rsync -avn ./project/ work:/srv/project/
```

## Удаление лишних файлов

`--delete` удаляет в destination файлы, которых нет в source.

```bash
rsync -av --delete ./site/ work:/srv/site/
```

Перед этим почти всегда стоит сделать:

```bash
rsync -av --delete --dry-run ./site/ work:/srv/site/
```

## Исключения

Исключить директории и файлы:

```bash
rsync -av \
  --exclude '.git/' \
  --exclude 'node_modules/' \
  --exclude '.env' \
  ./project/ work:/srv/project/
```

Через файл:

```bash
rsync -av --exclude-from=.rsyncignore ./project/ work:/srv/project/
```

Пример `.rsyncignore`:

```text
.git/
node_modules/
.env
*.log
```

## Порт и ключ

Через SSH-опции:

```bash
rsync -av -e "ssh -p 2222" ./project/ user@server.com:/srv/project/
```

С ключом:

```bash
rsync -av -e "ssh -i ~/.ssh/work_ed25519" ./project/ user@server.com:/srv/project/
```

С ключом и портом:

```bash
rsync -av -e "ssh -i ~/.ssh/work_ed25519 -p 2222" ./project/ user@server.com:/srv/project/
```

Если настроен [[09 - SSH client config]], проще:

```bash
rsync -av ./project/ work:/srv/project/
```

## Типовой деплой проекта

```bash
rsync -avz --delete \
  --exclude '.git/' \
  --exclude 'node_modules/' \
  --exclude '.env' \
  ./ work:/srv/app/
```

Потом на сервере:

```bash
ssh work "cd /srv/app && docker compose up -d --build"
```

## Частые ошибки

### Лишний уровень директории

Разница:

```bash
rsync -av ./project work:/srv/
rsync -av ./project/ work:/srv/project/
```

Первый вариант копирует директорию `project`. Второй копирует содержимое `project`.

### Опасный `--delete`

Проверяй source и destination:

```bash
rsync -avn --delete ./site/ work:/srv/site/
```

### Нет прав на запись

Копируй во временную директорию или настрой права на сервере:

```bash
rsync -av ./file work:/tmp/
ssh work "sudo mv /tmp/file /protected/path/"
```

## Главное запомнить

- `rsync` хорош для повторяемой передачи и деплоя.
- Перед `--delete` используй `--dry-run`.
- Слеш в конце source важен.
- С алиасом `work` команды становятся короче и безопаснее для привычного использования.
