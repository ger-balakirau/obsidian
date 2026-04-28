# Ubuntu Server — базовая настройка

## Связанные темы

- [[Linux заметки/ssh/00 - Карта темы|SSH]]
- [[Systemd заметки/00 - Карта темы|Systemd]]
- [[Сети/00 - Карта темы|Сети]]
- [[Траблшутинг/00 - Карта темы|Troubleshooting]]

Практический чеклист для нового Ubuntu-сервера: VPS, dedicated или домашняя VM. Идея простая: сначала не потерять доступ, затем обновить систему, создать обычного пользователя, защитить SSH и включить базовую наблюдаемость.

## Перед началом

- Держи открытой текущую SSH-сессию, пока не проверишь новую.
- Если меняешь SSH-настройки, сначала проверь конфиг командой `sudo sshd -t`.
- Firewall включай только после правила для SSH.
- Команды ниже предполагают Ubuntu с `systemd` и `apt`.

## 1. Обновить систему

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
cat /var/run/reboot-required
```

Если хочешь сначала посмотреть, что будет обновлено:

```bash
sudo apt upgrade -s
```

## 2. Создать обычного пользователя

Не работай постоянно под `root`. Создай отдельного пользователя и дай ему `sudo`.

```bash
sudo adduser devops
sudo usermod -aG sudo devops
```

Проверка:

```bash
su - devops
sudo whoami
```

Ожидаемый ответ на вторую команду:

```text
root
```

## 3. Настроить SSH-доступ по ключу

С локального компьютера:

```bash
ssh-copy-id devops@server_ip
```

Проверка входа:

```bash
ssh devops@server_ip
```

Если ключ добавляешь вручную, подробности смотри в [[Linux заметки/ssh/07 - Добавление публичного ключа на сервер]].

## 4. Ужесточить SSH

Открой конфиг:

```bash
sudo nano /etc/ssh/sshd_config
```

Базовые безопасные параметры:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
PermitEmptyPasswords no
```

Проверка конфига:

```bash
sudo sshd -t
```

Применение:

```bash
sudo systemctl reload ssh
```

Проверка в новой сессии:

```bash
ssh devops@server_ip
```

Если новая сессия не открылась, не закрывай старую. Подробная заметка: [[Linux заметки/ssh/16 - Безопасная настройка SSH-сервера]].

## 5. Включить firewall

```bash
sudo apt install -y ufw
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
```

Если SSH работает на нестандартном порту, сначала разреши этот порт:

```bash
sudo ufw allow 2222/tcp
```

## 6. Настроить время и hostname

Таймзона:

```bash
sudo timedatectl set-timezone Europe/Moscow
timedatectl status
```

Hostname:

```bash
sudo hostnamectl set-hostname my-server
hostnamectl
```

Проверь `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Пример строки:

```text
127.0.1.1 my-server
```

## 7. Установить базовые пакеты

```bash
sudo apt install -y \
  curl wget git htop unzip ca-certificates \
  gnupg lsb-release net-tools
```

Минимум для диагностики:

```bash
sudo apt install -y \
  dnsutils iputils-ping traceroute netcat-openbsd
```

## 8. Включить автоматические security-обновления

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

Проверить статус:

```bash
systemctl status unattended-upgrades
```

## 9. Настроить swap

Проверка:

```bash
swapon --show
free -h
```

Создать swap-файл на 2 GB:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Добавить в автозагрузку:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Если `fallocate` не сработал:

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
```

Настроить более спокойное использование swap:

```bash
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10
```

Чтобы сохранить после перезагрузки, добавь в `/etc/sysctl.conf`:

```text
vm.swappiness=10
```

## 10. Проверить ресурсы и сервисы

```bash
uptime
free -h
df -h
ss -tulnp
systemctl list-units --type=service --state=running
```

Логи:

```bash
journalctl -xe
journalctl -u ssh
```

## 11. Установить Fail2Ban

Fail2Ban полезен как дополнительная защита от перебора паролей и шумных ботов. Он не заменяет вход по ключам и firewall.

```bash
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
```

Статус SSH-jail:

```bash
sudo fail2ban-client status sshd
```

Логи:

```bash
sudo journalctl -u fail2ban -f
```

Разбан IP:

```bash
sudo fail2ban-client set sshd unbanip 1.2.3.4
```

## 12. Создать рабочие директории

```bash
mkdir -p ~/projects ~/logs
```

Для общих директорий с группой смотри [[Linux заметки/Пользователи группы/05 - Umask и ACL]].

## Финальная проверка

```bash
sudo apt update
sudo ufw status verbose
systemctl status ssh
systemctl status fail2ban
swapon --show
timedatectl status
```

Минимальный результат:

- система обновлена;
- создан пользователь с `sudo`;
- SSH работает по ключу;
- вход под `root` отключён;
- парольный вход отключён;
- firewall включён;
- время и hostname настроены;
- swap настроен при необходимости;
- базовые команды диагностики доступны;
- Fail2Ban установлен и запущен.

## Следующие шаги

- Docker или Podman.
- Nginx или Caddy.
- Резервные копии: `rsync`, Borg, Restic.
- Мониторинг: Prometheus, Grafana, VictoriaMetrics.
- CI/CD: GitHub Actions или GitLab CI.

Смысловые продолжения:

- управление сервисами: [[Systemd заметки/02 - systemctl - управление сервисами]];
- логи: [[Systemd заметки/04 - journalctl - просмотр логов]];
- диагностика сервисов: [[Systemd заметки/07 - Диагностика systemd-сервисов]];
- передача файлов на сервер: [[Linux заметки/ssh/Передача файлов/00 - Карта темы]].
