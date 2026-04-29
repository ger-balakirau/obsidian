# Ubuntu lab proxy bastion и reverse proxy

Эта заметка по практической lab-схеме из чата: Windows 11 + Hyper-V NAT + несколько Ubuntu VM. Одна VM используется как `proxy/bastion`: через неё ходим по SSH на остальные VM и через неё открываем веб-сервисы.

## Рекомендуемая схема

```text
Windows 11 Pro / Hyper-V NAT
  vEthernet: 10.10.0.1
  NAT:      10.10.0.0/23

proxy-vm:       10.10.0.10/23
vm-app:         10.10.0.11/23
vm-www-db:      10.10.0.12/23
vm-mongo-track: 10.10.0.13/23
```

Логика:

```text
Windows -> proxy-vm -> остальные VM
```

`proxy-vm` выполняет две роли:

- SSH bastion / jump host;
- Nginx reverse proxy для HTTP-сервисов.

## Почему proxy-vm удобна

Без proxy:

```text
нужно открывать доступ к каждой VM отдельно
```

С proxy:

```text
открыта только proxy-vm
остальные VM доступны через неё
```

Это ближе к нормальной серверной практике: наружу выставляется точка входа, внутренние сервисы остаются внутри lab-сети.

## Hostname и имена

На proxy-vm:

```bash
sudo hostnamectl set-hostname proxy-vm
```

Проверить:

```bash
hostnamectl
hostname -I
```

Если хочешь ходить с Windows по имени:

```text
proxy.lab
```

нужна DNS-запись или запись в Windows `hosts`.

Файл на Windows:

```text
C:\Windows\System32\drivers\etc\hosts
```

Пример:

```text
10.10.0.10 proxy.lab
10.10.0.10 app.lab
10.10.0.10 www-db.lab
```

Подробно: [[03 - Смена hostname в Linux]].

## Статический IP через netplan

На каждой Ubuntu VM сначала узнать имя интерфейса:

```bash
ip a
```

Обычно это:

```text
eth0
ens160
enp0s3
```

Файл netplan:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Пример для `proxy-vm`:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 10.10.0.10/23
      routes:
        - to: default
          via: 10.10.0.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Применить:

```bash
sudo netplan apply
```

Проверить:

```bash
ip -4 addr
ip route
ping 10.10.0.1
ping 8.8.8.8
```

Важно: если Hyper-V NAT создан на `10.10.0.0/23`, на Ubuntu тоже указывай `/23`, а не `/24`.

## Включить SSH на всех VM

```bash
sudo apt update
sudo apt install -y openssh-server curl
sudo systemctl enable --now ssh
systemctl status ssh
```

С Windows проверить proxy:

```powershell
ssh user@10.10.0.10
```

Или по имени, если настроен `hosts`/DNS:

```powershell
ssh user@proxy.lab
```

## SSH через ProxyJump

На Windows открыть:

```text
C:\Users\ТВОЙ_ПОЛЬЗОВАТЕЛЬ\.ssh\config
```

Пример:

```sshconfig
Host lab-proxy
    HostName 10.10.0.10
    User user

Host lab-app
    HostName 10.10.0.11
    User user
    ProxyJump lab-proxy

Host lab-www-db
    HostName 10.10.0.12
    User user
    ProxyJump lab-proxy

Host lab-mongo-track
    HostName 10.10.0.13
    User user
    ProxyJump lab-proxy
```

После этого:

```powershell
ssh lab-proxy
ssh lab-app
ssh lab-www-db
ssh lab-mongo-track
```

Фактически подключение к внутренним VM пойдёт через `lab-proxy`.

## Termius

Для proxy:

```text
Host: 10.10.0.10 или proxy.lab
Port: 22
User: пользователь Ubuntu
Auth: SSH key
```

Для остальных VM:

```text
Host: 10.10.0.11
Port: 22
User: пользователь нужной VM
Jump Host: proxy.lab / lab-proxy
```

Если заходишь извне через port forwarding:

```text
External: 95.x.x.x:2222
Internal: Windows:2222 -> proxy-vm:22
```

## Nginx reverse proxy

На `proxy-vm`:

```bash
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

Создать конфиг:

```bash
sudo nano /etc/nginx/sites-available/lab.conf
```

Пример:

```nginx
server {
    listen 80;
    server_name app.lab;

    location / {
        proxy_pass http://10.10.0.11:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name www-db.lab;

    location / {
        proxy_pass http://10.10.0.12:80;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Активировать:

```bash
sudo ln -s /etc/nginx/sites-available/lab.conf /etc/nginx/sites-enabled/lab.conf
sudo nginx -t
sudo systemctl reload nginx
```

На Windows в `hosts`:

```text
10.10.0.10 app.lab
10.10.0.10 www-db.lab
```

Проверить в браузере:

```text
http://app.lab
http://www-db.lab
```

## MongoDB не открывать наружу

MongoDB лучше не публиковать наружу. Для доступа с Windows использовать SSH-туннель через proxy:

```powershell
ssh -L 27017:10.10.0.13:27017 lab-proxy
```

После этого подключаться к:

```text
localhost:27017
```

Для нескольких MongoDB:

```powershell
ssh -L 27017:10.10.0.11:27017 -L 27018:10.10.0.12:27017 -L 27019:10.10.0.13:27017 lab-proxy
```

## Ограничить доступ через UFW

На внутренней VM можно разрешить доступ только от proxy.

Пример для `vm-app`:

```bash
sudo apt install -y ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow from 10.10.0.10 to any port 22 proto tcp
sudo ufw allow from 10.10.0.10 to any port 3000 proto tcp

sudo ufw enable
sudo ufw status verbose
```

Важно: сначала разреши SSH от proxy, потом включай `ufw`, чтобы не потерять доступ.

## Если нужен доступ извне

Если работаешь только с Windows-хоста, port forwarding через NAT не нужен: Windows напрямую видит `10.10.0.x`.

Если нужен доступ из интернета или другой сети, пробрасывай только proxy-vm.

На Windows PowerShell от администратора:

```powershell
Get-NetNat
```

Проброс SSH:

```powershell
Add-NetNatStaticMapping `
  -NatName "LabNAT" `
  -Protocol TCP `
  -ExternalIPAddress "0.0.0.0/0" `
  -ExternalPort 2222 `
  -InternalIPAddress "10.10.0.10" `
  -InternalPort 22
```

Проброс HTTP:

```powershell
Add-NetNatStaticMapping `
  -NatName "LabNAT" `
  -Protocol TCP `
  -ExternalIPAddress "0.0.0.0/0" `
  -ExternalPort 8080 `
  -InternalIPAddress "10.10.0.10" `
  -InternalPort 80
```

Открыть firewall Windows:

```powershell
New-NetFirewallRule -DisplayName "Lab Proxy SSH 2222" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2222
New-NetFirewallRule -DisplayName "Lab Proxy HTTP 8080" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080
```

Проверить:

```powershell
Get-NetNatStaticMapping -NatName "LabNAT"
```

Удалить ошибочный проброс:

```powershell
Remove-NetNatStaticMapping -NatName "LabNAT" -ExternalPort 2222 -Protocol TCP
```

## VPN может мешать

VPN на Windows может ломать доступ к lab-сети или входящий port forwarding.

Проверить в VPN-клиенте:

```text
Block LAN / Блокировать локальную сеть
Kill Switch
Block inbound connections
Route all traffic through VPN
```

В исключения VPN полезно добавить:

```text
192.168.1.0/24  домашняя LAN
10.10.0.0/23    Hyper-V NAT lab
10.10.0.1       gateway Hyper-V NAT
10.10.0.10      proxy-vm
```

Если через VPN нужно попасть на `10.10.0.10`, VPN должен маршрутизировать сеть:

```text
10.10.0.0/23
```

## Проверочный чеклист

С Windows:

```powershell
ping 10.10.0.10
ping proxy.lab
ssh lab-proxy
ssh lab-app
```

На proxy-vm:

```bash
hostnamectl
hostname -I
ip route
systemctl status ssh
systemctl status nginx
sudo nginx -t
```

Проверка портов:

```bash
ss -tulnp
```

## Главное запомнить

- `hostname` внутри Linux не создаёт DNS автоматически.
- Для `proxy.lab` нужна DNS-запись или `hosts`.
- В Hyper-V NAT сети `/23` на Ubuntu тоже ставь `/23`.
- Для SSH к внутренним VM используй `ProxyJump`.
- Для веба используй Nginx reverse proxy на `proxy-vm`.
- Базы данных наружу не открывать, лучше SSH-туннель.
- Если нужен доступ извне, пробрасывай только proxy-vm.

## Связанные заметки

- [[03 - Смена hostname в Linux]]
- [[Linux заметки/ssh/09 - SSH client config]]
- [[Linux заметки/ssh/14 - Port forwarding через SSH]]
- [[Windows Server заметки/13 - Hyper-V lab сеть - проблемы и решения]]
- [[Windows Server заметки/11 - Windows NAT через PowerShell]]
- [[Сети/06 - NAT]]
