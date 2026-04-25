# Диагностика systemd-сервисов

Если сервис не стартует или падает, не начинай с перезагрузки сервера. Сначала собери факты.

## 1. Проверить статус

```bash
systemctl status myapp
```

Смотри:

- `Active`;
- `Main PID`;
- `ExecStart`;
- последние строки логов;
- exit code.

## 2. Посмотреть логи

```bash
journalctl -u myapp -n 100
```

В реальном времени:

```bash
journalctl -u myapp -f
```

С текущей загрузки:

```bash
journalctl -u myapp -b
```

## 3. Проверить unit-файл

```bash
systemctl cat myapp
```

После изменений:

```bash
sudo systemctl daemon-reload
```

## 4. Запустить команду вручную

Найди `ExecStart`:

```bash
systemctl cat myapp
```

Попробуй выполнить команду вручную из нужной директории и под нужным пользователем.

## 5. Проверить пользователя и права

```bash
id deploy
ls -ld /srv/myapp
ls -l /srv/myapp
```

Если сервис пишет файлы:

```bash
ls -ld /var/log/myapp
```

## 6. Проверить порт

```bash
ss -tulnp
```

Если порт занят, сервис может не стартовать.

## 7. Reset failed state

Если сервис в состоянии failed:

```bash
sudo systemctl reset-failed myapp
```

Потом:

```bash
sudo systemctl start myapp
```

## Частые причины

- неверный путь в `ExecStart`;
- нет `daemon-reload` после изменения unit;
- нет прав у пользователя сервиса;
- не существует `WorkingDirectory`;
- занят порт;
- не установлены переменные окружения;
- приложение падает сразу после запуска.

## Главное запомнить

Минимальная диагностика:

```bash
systemctl status myapp
journalctl -u myapp -n 100
systemctl cat myapp
```
