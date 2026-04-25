# Join computer to domain

Ввод компьютера в домен добавляет его в Active Directory и позволяет управлять им через доменные учётки и GPO.

## Что нужно заранее

- компьютер видит сеть домена;
- DNS указывает на доменный DNS;
- доступен контроллер домена;
- есть учётная запись с правом join;
- время на клиенте не сильно отличается от DC.

## Проверка перед join

На клиенте:

```cmd
ipconfig /all
ping domain.local
nslookup domain.local
nltest /dsgetdc:domain.local
```

## Через GUI

```text
System Properties
Computer Name
Change
Domain
```

После ввода домена потребуется логин и пароль учётки с правами.

## Через PowerShell

```powershell
Add-Computer -DomainName domain.local -Credential domain\admin -Restart
```

## После join

Проверить:

```cmd
whoami
gpupdate /force
gpresult /r
```

## Частые ошибки

### Не находится домен

Почти всегда проверь DNS:

```cmd
ipconfig /all
nslookup domain.local
```

### Неверное время

Kerberos чувствителен к времени.

Проверить:

```cmd
w32tm /query /status
```

### Нет прав

Учётка не имеет права вводить ПК в домен или превышен лимит.

## Главное запомнить

- Для join domain DNS должен указывать на доменный DNS.
- После join компьютер появляется в AD.
- GPO начнут применяться после связи с доменом и обновления политик.
