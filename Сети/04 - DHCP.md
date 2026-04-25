# DHCP

DHCP автоматически выдаёт устройствам сетевые настройки: IP-адрес, маску, шлюз, DNS и дополнительные параметры.

## Что выдаёт DHCP

Обычно DHCP выдаёт:

```text
IP address
Subnet mask
Default gateway
DNS servers
Lease time
```

## Lease

Lease - срок аренды IP-адреса.

Клиент получает адрес не навсегда, а на время. Потом он продлевает аренду.

## DHCP scope

Scope - диапазон адресов, которые DHCP может выдавать.

Пример:

```text
Network: 192.168.1.0/24
Scope:   192.168.1.100 - 192.168.1.200
Gateway: 192.168.1.1
DNS:     192.168.1.1
```

## Reservation

Reservation - закрепление конкретного IP за конкретным устройством по MAC-адресу.

Используют для:

- принтеров;
- камер;
- серверов;
- сетевого оборудования.

## APIPA 169.254.x.x

Если Windows получила адрес вида:

```text
169.254.x.x
```

значит DHCP не выдал адрес.

Частые причины:

- DHCP-сервер недоступен;
- кабель или Wi-Fi не работает;
- VLAN не та;
- закончились адреса в scope;
- firewall или relay блокирует DHCP.

## Проверка на Windows

```cmd
ipconfig /all
ipconfig /release
ipconfig /renew
```

Что смотреть:

```text
DHCP Enabled
IPv4 Address
Subnet Mask
Default Gateway
DHCP Server
DNS Servers
Lease Obtained
Lease Expires
```

## Проверка на Linux

```bash
ip a
ip route
resolvectl status
```

Запросить адрес заново:

```bash
sudo dhclient -r
sudo dhclient
```

На системах с NetworkManager:

```bash
nmcli device status
nmcli connection show
```

## Типовая диагностика

1. Есть ли физическое подключение.
2. Есть ли IP.
3. Не `169.254.x.x` ли адрес.
4. Есть ли gateway.
5. Есть ли DNS.
6. Доступен ли DHCP-сервер.
7. Не закончился ли пул адресов.

## Главное запомнить

- DHCP выдаёт сетевые настройки автоматически.
- `169.254.x.x` почти всегда значит “DHCP не сработал”.
- `ipconfig /all` - главная команда на Windows.
- DHCP не заменяет DNS: DHCP выдаёт адрес DNS-сервера, но сам имена не резолвит.
