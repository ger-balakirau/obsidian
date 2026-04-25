# Timers - задачи по расписанию

Systemd timers - альтернатива cron. Timer запускает связанный service по расписанию.

## Из чего состоит timer

Обычно нужно два файла:

```text
backup.service
backup.timer
```

Timer отвечает за расписание, service - за действие.

## Пример service

```ini
[Unit]
Description=Run backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

## Пример timer

```ini
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

## Включить timer

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
```

## Посмотреть timers

```bash
systemctl list-timers
```

Все timers:

```bash
systemctl list-timers --all
```

## Запустить service вручную

```bash
sudo systemctl start backup.service
```

## Посмотреть логи

```bash
journalctl -u backup.service
```

## Частые расписания

```ini
OnCalendar=hourly
OnCalendar=daily
OnCalendar=weekly
OnCalendar=*-*-* 03:00:00
OnCalendar=Mon..Fri 09:00
```

## Главное запомнить

- Timer запускает service.
- Для одноразовой задачи в service часто используют `Type=oneshot`.
- После создания timer нужен `daemon-reload`.
- Автозапуск timer включают через `enable --now name.timer`.
