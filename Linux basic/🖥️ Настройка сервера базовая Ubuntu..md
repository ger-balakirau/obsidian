
Ниже — обновлённая версия заметки для Obsidian с добавленным разделом по swap и несколькими полезными дополнениями.

---

# 🖥️ Ubuntu Server — базовая настройка

Подходит для Ubuntu серверов (VPS / dedicated)

---

## 📌 1. Обновление системы

```bash
sudo apt update && sudo apt upgrade -y
```

Режим симуляции:

```bash
sudo apt upgrade -s
```

Очистка:

```bash
sudo apt autoremove -y
```

---

## 👤 2. Создание пользователя

```bash
adduser devops
usermod -aG sudo devops
```

Проверка:

```bash
su - devops
```

---

## 🔐 3. Настройка SSH

Копирование ключа:

```bash
ssh-copy-id devops@server_ip
```

Конфиг:

```bash
sudo nano /etc/ssh/sshd_config
```

Изменить:

```bash
PermitRootLogin no
PasswordAuthentication no
UsePAM yes
PrintMotd no
```

Перезапуск:

```bash
sudo systemctl restart ssh
```

---

## 🔥 4. Firewall (UFW)

```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

---

## 🌐 5. Таймзона

```bash
timedatectl set-timezone Europe/Berlin
timedatectl status
```

---

## 📦 6. Базовые пакеты

```bash
sudo apt install -y \
curl wget git htop unzip net-tools \
ca-certificates gnupg lsb-release
```

---

## 📊 7. Мониторинг

```bash
htop
free -h
df -h
uptime
```

---

## 🧠 8. Hostname

```bash
hostnamectl set-hostname my-server
```

```bash
sudo nano /etc/hosts
```

Добавить:

```bash
127.0.1.1 my-server
```

---

## 🔄 9. Автообновления

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

---
## 🛡️ 10. Fail2Ban (метрики и мониторинг)

### 📊 Общий статус

```bash
sudo fail2ban-client status
```

Показывает:

- список jail’ов (ssh, nginx и т.д.)
- общее состояние

---

### 🔍 Статистика по конкретному jail (например SSH)

```bash
sudo fail2ban-client status sshd
```

Ключевые поля:

- **Currently banned** — сколько IP сейчас заблокировано
- **Total banned** — всего банов
- **Banned IP list** — список IP

---

### 📈 Смотреть логи (важно)

```bash
sudo journalctl -u fail2ban -f
```

или файл логов:

```bash
sudo tail -f /var/log/fail2ban.log
```

---

### 📉 Количество банов (быстрая метрика)

```bash
grep "Ban " /var/log/fail2ban.log | wc -l
```

---

### 📌 Последние забаненные IP

```bash
grep "Ban " /var/log/fail2ban.log | tail
```

---

### 🔓 Разбан IP

```bash
sudo fail2ban-client set sshd unbanip 1.2.3.4
```

---

### 🔎 Проверка активных правил (iptables)

```bash
sudo iptables -L -n | grep f2b
```

или (если используется nftables):

```bash
sudo nft list ruleset | grep f2b
```

---

### ⚙️ Включить автозапуск

```bash
sudo systemctl enable fail2ban
```

---

### 📊 Мини-мониторинг (одной командой)

```bash
watch -n 2 "sudo fail2ban-client status sshd"
```

---

## 💡 Дополнительно (если нужен продакшн уровень)

- увеличить `bantime`
- уменьшить `maxretry`
- добавить jail для:
    - nginx
    - docker
    - sshd-ddos

Конфиг:

```bash
sudo nano /etc/fail2ban/jail.local
```


---

## 💾 11. Swap (создание / увеличение / пересоздание)

### 📌 Проверка текущего swap

```bash
swapon --show
free -h
```

---

### ➕ Создать swap файл (например 2GB)

```bash
sudo fallocate -l 2G /swapfile
```

Если не сработало:

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
```

Права:

```bash
sudo chmod 600 /swapfile
```

Создание swap:

```bash
sudo mkswap /swapfile
sudo swapon /swapfile
```

Добавить в автозагрузку:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### 🔼 Увеличить swap

1. Отключить:

```bash
sudo swapoff /swapfile
```

2. Удалить:

```bash
sudo rm /swapfile
```

3. Создать заново (с нужным размером) — см. выше

---

### 🔧 Настройка swappiness

```bash
cat /proc/sys/vm/swappiness
```

Изменить (например 10):

```bash
sudo sysctl vm.swappiness=10
```

Постоянно:

```bash
sudo nano /etc/sysctl.conf
```

Добавить:

```bash
vm.swappiness=10
```

---

## 📁 12. Директории

```bash
mkdir -p ~/projects
mkdir -p ~/logs
```

---

## ⚙️ 13. Сервисы

```bash
systemctl list-units --type=service --state=running
```

---

## 📜 14. Логи

```bash
journalctl -xe
journalctl -u ssh
```

---

## 🌍 15. Сеть (быстрая проверка)

```bash
ip a
ss -tulnp
```

---

## 🚀 Минимальный итог

- Система обновлена
- Создан sudo-пользователь
- SSH защищён (ключи, root off)
- Firewall включён
- Время настроено
- Swap настроен
- Автообновления включены
- Fail2Ban активен

---

## 📌 Следующие шаги

- Docker
- Nginx / Caddy
- CI/CD (GitHub Actions / GitLab CI)
- Резервные копии (rsync / borg)
- Мониторинг (Prometheus + Grafana)
---
