# journalctl - просмотр логов

`journalctl` показывает логи systemd journal. Это главный инструмент для просмотра логов сервисов.

## Логи сервиса

```bash
journalctl -u nginx
```

В реальном времени:

```bash
journalctl -u nginx -f
```

Последние строки:

```bash
journalctl -u nginx -n 100
```

## Логи с текущей загрузки

```bash
journalctl -b
```

Для сервиса с текущей загрузки:

```bash
journalctl -u nginx -b
```

## Логи по времени

```bash
journalctl --since "1 hour ago"
journalctl --since "2026-04-25 10:00"
journalctl --since today
```

Для сервиса:

```bash
journalctl -u nginx --since "30 minutes ago"
```

## Ошибки

```bash
journalctl -p err
```

Ошибки текущей загрузки:

```bash
journalctl -p err -b
```

## Логи ядра

```bash
journalctl -k
```

## Удобный вывод

Без pager:

```bash
journalctl -u nginx --no-pager
```

С точным временем:

```bash
journalctl -u nginx -o short-iso
```

## Главное запомнить

- `journalctl -u service` - логи сервиса.
- `-f` - live-режим.
- `-b` - текущая загрузка.
- `--since` - фильтр по времени.
- `-p err` - только ошибки.
