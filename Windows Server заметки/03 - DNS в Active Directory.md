# DNS в Active Directory

DNS в домене Windows - критичная служба. Клиенты и серверы находят контроллеры домена через DNS.

## Почему DNS важен

Active Directory использует DNS для поиска:

- контроллеров домена;
- LDAP;
- Kerberos;
- глобального каталога;
- доменных служб.

Если DNS неправильный, домен начинает “сыпаться”.

## Главное правило

Клиенты домена должны использовать доменный DNS, а не внешний DNS напрямую.

Плохо:

```text
DNS: 8.8.8.8
```

Хорошо:

```text
DNS: IP контроллера домена
```

## Зоны

Forward lookup zone:

```text
company.local -> IP
```

Reverse lookup zone:

```text
IP -> name
```

## Записи

Частые записи:

```text
A      имя -> IPv4
CNAME  alias -> другое имя
PTR    IP -> имя
SRV    служебные записи AD
```

## Проверка на клиенте

```cmd
ipconfig /all
nslookup domain.local
nslookup dc01.domain.local
nltest /dsgetdc:domain.local
```

Очистить DNS cache:

```cmd
ipconfig /flushdns
```

## Проверка SRV-записей

```cmd
nslookup
set type=SRV
_ldap._tcp.dc._msdcs.domain.local
```

## Частые проблемы

- клиент использует внешний DNS;
- DHCP выдаёт неправильный DNS;
- DNS-записи устарели;
- контроллер домена недоступен;
- нет нужных SRV-записей;
- неправильная reverse zone.

## Главное запомнить

- DNS - фундамент Active Directory.
- Клиенты домена должны смотреть на DNS контроллера домена.
- Если ПК не входит в домен или GPO не применяются, проверь DNS одним из первых.
