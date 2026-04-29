# Join computer to domain

Ввод компьютера в домен добавляет его в Active Directory и позволяет управлять им через доменные учётки и GPO.

Автоматизация ввода с флешки и Offline Domain Join: [[16 - Автоматический ввод ПК в домен с флешки]].

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

Сразу в нужную OU:

```powershell
Add-Computer `
    -DomainName domain.local `
    -OUPath "OU=Workstations,DC=domain,DC=local" `
    -Credential domain\admin `
    -Restart
```

## Через Offline Domain Join

Если не хочешь вводить доменный логин и пароль на новом ПК, используй `djoin`.

Подробная практика: [[16 - Автоматический ввод ПК в домен с флешки]].

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
