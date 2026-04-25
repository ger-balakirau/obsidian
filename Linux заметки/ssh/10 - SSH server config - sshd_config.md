# SSH server config: `sshd_config`

`/etc/ssh/sshd_config` — это конфиг SSH-сервера. Он определяет, как сервер принимает входящие подключения: на каком порту слушает, кого пускает, какие способы входа разрешены и какие функции SSH доступны.

Не путай:

- `~/.ssh/config` — конфиг SSH-клиента на твоём компьютере;
- `/etc/ssh/sshd_config` — конфиг SSH-сервера на удалённой машине;
- `~/.ssh/authorized_keys` — публичные ключи, которым разрешён вход под конкретным пользователем.

Практический рецепт безопасной настройки вынесен отдельно: [[16 - Безопасная настройка SSH-сервера]].

## Как читать конфиг

Открыть файл:

```bash
sudo nano /etc/ssh/sshd_config
```

Проверить синтаксис:

```bash
sudo sshd -t
```

Показать итоговые настройки с учётом дефолтов и `Include`:

```bash
sudo sshd -T
```

Строки с `#` обычно являются комментариями. Но это не всегда значит, что параметр выключен: у SSH есть значения по умолчанию.

## `Include`

На Ubuntu часто есть строка:

```sshconfig
Include /etc/ssh/sshd_config.d/*.conf
```

Это значит, что SSH читает не только основной файл, но и дополнительные файлы из директории:

```text
/etc/ssh/sshd_config.d/
```

Если настройка непонятно откуда берётся, смотри итоговый результат через:

```bash
sudo sshd -T
```

## Безопасный порядок изменений

1. Не закрывай текущую SSH-сессию.
2. Сделай резервную копию.
3. Измени конфиг.
4. Проверь `sudo sshd -t`.
5. Примени через `reload`, а не через грубый restart.
6. Открой новое окно терминала и проверь вход.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo sshd -t
sudo systemctl reload ssh
```

Если сервис называется `sshd`:

```bash
sudo systemctl reload sshd
```

## Сетевые параметры

### `Port`

Порт, на котором SSH принимает подключения.

```sshconfig
Port 22
```

Если меняешь порт, сначала открой его в firewall:

```bash
sudo ufw allow 2222/tcp
```

### `ListenAddress`

Адрес, на котором SSH слушает подключения. Обычно не трогают.

```sshconfig
ListenAddress 0.0.0.0
```

## Аутентификация

### `PermitRootLogin`

Управляет входом под `root`.

```sshconfig
PermitRootLogin no
```

Частые значения:

```text
no                 — root не может входить по SSH
yes                — root может входить по SSH
prohibit-password  — root может входить по ключу, но не по паролю
```

Для обычного сервера чаще выбирают `no`.

### `PubkeyAuthentication`

Разрешает вход по публичному ключу.

```sshconfig
PubkeyAuthentication yes
```

### `AuthorizedKeysFile`

Где сервер ищет публичные ключи пользователя.

```sshconfig
AuthorizedKeysFile .ssh/authorized_keys
```

Путь считается относительно домашней директории пользователя.

### `PasswordAuthentication`

Разрешает или запрещает вход по паролю.

```sshconfig
PasswordAuthentication no
```

Для сервера с ключами обычно ставят `no`.

### `KbdInteractiveAuthentication`

Управляет keyboard-interactive входом. На практике может выглядеть как парольный вход или MFA-промпт.

```sshconfig
KbdInteractiveAuthentication no
```

Если хочешь полностью отключить парольный вход, проверь и `PasswordAuthentication`, и `KbdInteractiveAuthentication`.

### `UsePAM`

Подключает PAM-механизмы Linux: сессии, ограничения, окружение, иногда MFA.

```sshconfig
UsePAM yes
```

На Ubuntu обычно оставляют `yes`.

### `PermitEmptyPasswords`

Запрещает вход с пустым паролем.

```sshconfig
PermitEmptyPasswords no
```

### `StrictModes`

Заставляет SSH проверять владельцев и права на домашнюю директорию, `.ssh` и `authorized_keys`.

```sshconfig
StrictModes yes
```

Если права слишком открытые, вход по ключу может быть отклонён.

## Ограничение пользователей

### `AllowUsers`

Разрешает вход только перечисленным пользователям.

```sshconfig
AllowUsers deploy admin
```

Можно ограничить пользователя по адресу:

```sshconfig
AllowUsers deploy@203.0.113.*
```

### `DenyUsers`

Запрещает вход перечисленным пользователям.

```sshconfig
DenyUsers test guest
```

### `AllowGroups`

Разрешает вход только пользователям из указанных групп.

```sshconfig
AllowGroups ssh-users admins
```

## Лимиты входа

```sshconfig
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 5
MaxStartups 10:30:60
```

Что это значит:

- `LoginGraceTime` — сколько времени даётся на авторизацию;
- `MaxAuthTries` — сколько попыток аутентификации разрешено;
- `MaxSessions` — сколько сессий можно открыть в одном соединении;
- `MaxStartups` — ограничение неавторизованных подключений.

## Forwarding и туннели

```sshconfig
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no
```

Коротко:

- `X11Forwarding` — проброс графических приложений;
- `AllowAgentForwarding` — проброс SSH-agent на сервер;
- `AllowTcpForwarding` — локальные и удалённые SSH-туннели;
- `GatewayPorts` — может открыть remote forwarding наружу;
- `PermitTunnel` — TUN/TAP-туннели через SSH.

Если на сервере нужны SSH-туннели, не отключай `AllowTcpForwarding` без понимания последствий. Подробности: [[14 - Port forwarding через SSH]].

## Поддержание соединения

```sshconfig
ClientAliveInterval 300
ClientAliveCountMax 2
```

Сервер периодически проверяет, жив ли клиент. Если клиент не отвечает, соединение закрывается.

## Логи

```sshconfig
LogLevel INFO
```

Для временной диагностики можно:

```sshconfig
LogLevel VERBOSE
```

Смотреть логи:

```bash
sudo journalctl -u ssh -f
```

или:

```bash
sudo journalctl -u sshd -f
```

## Сообщения при входе

```sshconfig
PrintMotd no
PrintLastLog yes
Banner none
```

- `PrintMotd` — показывать message of the day;
- `PrintLastLog` — показывать информацию о последнем входе;
- `Banner` — показывать отдельный баннер до входа.

## SFTP

```sshconfig
Subsystem sftp /usr/lib/openssh/sftp-server
```

SFTP использует SSH как транспорт. Если отключить или сломать `Subsystem sftp`, `sftp`-клиенты могут перестать работать.

## `Match`

`Match` позволяет задавать отдельные правила для пользователя, группы, адреса или хоста.

Пример для пользователя `backup`:

```sshconfig
Match User backup
    PasswordAuthentication no
    AllowTcpForwarding no
    X11Forwarding no
```

Пример SFTP-only пользователя:

```sshconfig
Match User files
    ChrootDirectory /srv/sftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

## Проверка конкретного пользователя

Посмотреть итоговые настройки для конкретного пользователя и адреса:

```bash
sudo sshd -T -C user=deploy,host=server.com,addr=203.0.113.10
```

## Минимальная безопасная база

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
PermitEmptyPasswords no
StrictModes yes
```

## Главное запомнить

- `sshd_config` управляет сервером, а не клиентом.
- Комментированная строка не всегда означает “настройка выключена”.
- Реальную итоговую конфигурацию показывает `sudo sshd -T`.
- Перед применением всегда запускай `sudo sshd -t`.
- Не закрывай текущую SSH-сессию, пока не проверишь новое подключение.
