# Unit files - сервисы и параметры

Unit file описывает, как systemd должен запускать и контролировать сервис.

## Где лежат unit files

Системные units из пакетов:

```text
/lib/systemd/system/
```

Локальные units администратора:

```text
/etc/systemd/system/
```

Обычно свои unit-файлы кладут в:

```text
/etc/systemd/system/
```

## Пример простого сервиса

```ini
[Unit]
Description=My app
After=network.target

[Service]
WorkingDirectory=/srv/myapp
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
User=deploy
Group=deploy

[Install]
WantedBy=multi-user.target
```

## Основные секции

### `[Unit]`

Описание и зависимости.

```ini
Description=My app
After=network.target
```

### `[Service]`

Как запускать процесс.

```ini
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
User=deploy
```

### `[Install]`

Как unit включается в автозапуск.

```ini
WantedBy=multi-user.target
```

## Применить изменения unit-файла

После изменения unit-файла:

```bash
sudo systemctl daemon-reload
```

Потом:

```bash
sudo systemctl restart myapp
```

## Проверить unit

```bash
systemctl status myapp
journalctl -u myapp -f
```

## Drop-in override

Чтобы не редактировать unit из пакета напрямую:

```bash
sudo systemctl edit nginx
```

Посмотреть итоговый unit:

```bash
systemctl cat nginx
```

## Главное запомнить

- Свои units лучше хранить в `/etc/systemd/system/`.
- После изменения unit-файла нужен `daemon-reload`.
- `systemctl cat service` показывает итоговый unit.
- Units из `/lib/systemd/system/` лучше не править напрямую.
