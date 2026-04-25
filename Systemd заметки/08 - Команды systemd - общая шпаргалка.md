# Команды systemd - общая шпаргалка

## Сервисы

Подробнее: [[02 - systemctl - управление сервисами]]

```bash
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl reload-or-restart nginx
```

## Автозапуск

Подробнее: [[05 - Автозапуск и зависимости]]

```bash
sudo systemctl enable nginx
sudo systemctl enable --now nginx
sudo systemctl disable nginx
systemctl is-enabled nginx
systemctl is-active nginx
```

## Списки

```bash
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service
systemctl list-dependencies nginx
```

## Unit files

Подробнее: [[03 - Unit files - сервисы и параметры]]

```bash
systemctl cat nginx
sudo systemctl edit nginx
sudo systemctl daemon-reload
```

## Логи

Подробнее: [[04 - journalctl - просмотр логов]]

```bash
journalctl -u nginx
journalctl -u nginx -f
journalctl -u nginx -n 100
journalctl -u nginx -b
journalctl -p err -b
```

## Timers

Подробнее: [[06 - Timers - задачи по расписанию]]

```bash
systemctl list-timers
systemctl list-timers --all
sudo systemctl enable --now backup.timer
sudo systemctl start backup.service
journalctl -u backup.service
```

## Диагностика

Подробнее: [[07 - Диагностика systemd-сервисов]]

```bash
systemctl status myapp
journalctl -u myapp -n 100
systemctl cat myapp
sudo systemctl reset-failed myapp
sudo systemctl daemon-reload
```

## Минимум на память

```bash
systemctl status service
sudo systemctl restart service
sudo systemctl enable --now service
journalctl -u service -f
sudo systemctl daemon-reload
```
