# systemctl - управление сервисами

`systemctl` - основная команда для управления systemd units.

## Проверить статус

```bash
systemctl status nginx
```

Показывает:

- запущен ли сервис;
- когда стартовал;
- основной PID;
- последние строки логов;
- причину ошибки, если сервис упал.

## Запустить сервис

```bash
sudo systemctl start nginx
```

## Остановить сервис

```bash
sudo systemctl stop nginx
```

## Перезапустить сервис

```bash
sudo systemctl restart nginx
```

## Перечитать конфиг без полного restart

Если сервис поддерживает reload:

```bash
sudo systemctl reload nginx
```

Безопасный вариант:

```bash
sudo systemctl reload-or-restart nginx
```

## Включить автозапуск

```bash
sudo systemctl enable nginx
```

Включить и сразу запустить:

```bash
sudo systemctl enable --now nginx
```

## Отключить автозапуск

```bash
sudo systemctl disable nginx
```

## Посмотреть включён ли автозапуск

```bash
systemctl is-enabled nginx
```

## Посмотреть активен ли сервис

```bash
systemctl is-active nginx
```

## Список сервисов

Запущенные сервисы:

```bash
systemctl list-units --type=service --state=running
```

Все service unit files:

```bash
systemctl list-unit-files --type=service
```

## Главное запомнить

- `start` запускает сейчас.
- `enable` включает автозапуск.
- `restart` полностью перезапускает.
- `reload` перечитывает конфиг, если сервис это умеет.
- `status` - первая команда диагностики.
