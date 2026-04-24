
## `sshd_config` — это конфиг **SSH-сервера**. Он управляет тем, как сервер принимает входящие SSH-подключения.

Файл:

```text
/etc/ssh/sshd_config
```

Важно: `sshd_config` — это **серверный** конфиг.  
`ssh_config` или `~/.ssh/config` — это **клиентский** конфиг.

---

## Как читается `sshd_config`

Файл состоит из строк вида:

```text
Параметр значение
```

Пример:

```sshconfig
PasswordAuthentication no
```

Строки с `#` считаются комментариями и не применяются:

```sshconfig
#Port 22
```

Это значит: строка показана как пример или дефолт, но сейчас она не активна.

В OpenSSH для большинства параметров используется первое найденное значение, а строки с `#` и пустые строки игнорируются. Поэтому порядок строк и `Include` важны.

---

# Что активно в твоём конфиге

В твоём примере реально активны вот эти строки:

```sshconfig
Include /etc/ssh/sshd_config.d/*.conf

PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes

X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/openssh/sftp-server
```

Остальное закомментировано через `#`.

---

# Главная фишка: `Include`

```sshconfig
Include /etc/ssh/sshd_config.d/*.conf
```

Эта строка подключает дополнительные конфиги из директории:

```text
/etc/ssh/sshd_config.d/
```

Например:

```text
/etc/ssh/sshd_config.d/50-cloud-init.conf
/etc/ssh/sshd_config.d/99-custom.conf
```

Файлы подключаются по маске `*.conf` и обрабатываются в лексическом порядке. Это значит, что настройки могут лежать не только в `/etc/ssh/sshd_config`, но и в отдельных файлах внутри `sshd_config.d`.

Проверить, что реально применилось:

```bash
sudo sshd -T
```

Отфильтровать важное:

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|usepam|x11forwarding|allowusers)'
```

`sshd -T` показывает итоговую конфигурацию, а `sshd -t` только проверяет конфиг на ошибки.

---

# Безопасный порядок изменения SSH

Перед изменениями сделать копию:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Открыть файл:

```bash
sudo nano /etc/ssh/sshd_config
```

Проверить синтаксис:

```bash
sudo sshd -t
```

Если ошибок нет, применить:

```bash
sudo systemctl reload ssh
```

Если сервис называется `sshd`:

```bash
sudo systemctl reload sshd
```

Важно: **не закрывать текущую SSH-сессию**.  
Сначала открыть новое окно терминала и проверить новый вход.

---

# Разбор параметров

## `Port`

```sshconfig
#Port 22
```

Порт SSH-сервера.

По умолчанию SSH слушает порт `22`.

Активировать порт 22:

```sshconfig
Port 22
```

Нестандартный порт:

```sshconfig
Port 2222
```

Подключение после смены порта:

```bash
ssh -p 2222 user@server.com
```

Если меняешь порт, нужно открыть его в firewall.

Для `ufw`:

```bash
sudo ufw allow 2222/tcp
```

Проверить, слушает ли сервер порт:

```bash
sudo ss -tlnp | grep ssh
```

---

## `AddressFamily`

```sshconfig
#AddressFamily any
```

Какие IP-адреса использовать:

```sshconfig
AddressFamily any
```

Варианты:

```text
any   — IPv4 и IPv6
inet  — только IPv4
inet6 — только IPv6
```

---

## `ListenAddress`

```sshconfig
#ListenAddress 0.0.0.0
#ListenAddress ::
```

На каких адресах сервер будет слушать SSH.

Слушать на всех IPv4:

```sshconfig
ListenAddress 0.0.0.0
```

Слушать на всех IPv6:

```sshconfig
ListenAddress ::
```

Слушать только на конкретном IP сервера:

```sshconfig
ListenAddress 192.168.1.10
```

Фишка: если на сервере несколько IP, можно разрешить SSH только на одном из них.

---

## `HostKey`

```sshconfig
#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key
```

Это **ключи самого сервера**.

Не путать с пользовательскими ключами:

```text
~/.ssh/id_ed25519
~/.ssh/authorized_keys
```

`HostKey` нужен, чтобы клиент мог понять: это тот же сервер или кто-то подменил сервер.

Обычно руками не трогают.

---

# Аутентификация

## `PermitRootLogin`

У тебя сейчас:

```sshconfig
PermitRootLogin yes
```

Это разрешает вход по SSH под пользователем `root`.

Возможные значения:

```sshconfig
PermitRootLogin yes
PermitRootLogin no
PermitRootLogin prohibit-password
PermitRootLogin forced-commands-only
```

Значение `prohibit-password` запрещает root-вход по паролю и keyboard-interactive, но оставляет возможность входа root по ключу. Значение `no` полностью запрещает root-вход по SSH.

Лучший вариант для обычного сервера:

```sshconfig
PermitRootLogin no
```

Если root временно нужен только по ключу:

```sshconfig
PermitRootLogin prohibit-password
```

Фишка: лучше создать обычного пользователя и дать ему `sudo`, чем заходить сразу под `root`.

Пример:

```bash
ssh deploy@server.com
```

Потом:

```bash
sudo -i
```

---

## `PubkeyAuthentication`

У тебя:

```sshconfig
PubkeyAuthentication yes
```

Разрешает вход по SSH-ключу.

Обычно должно быть включено:

```sshconfig
PubkeyAuthentication yes
```

OpenSSH по умолчанию поддерживает public key authentication, а пользовательские публичные ключи обычно ищутся через `AuthorizedKeysFile`.

---

## `AuthorizedKeysFile`

У тебя строка закомментирована:

```sshconfig
#AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
```

Это файл, где сервер ищет публичные ключи пользователя.

Обычно используется:

```sshconfig
AuthorizedKeysFile .ssh/authorized_keys
```

Для пользователя `deploy` это будет:

```text
/home/deploy/.ssh/authorized_keys
```

Для `root`:

```text
/root/.ssh/authorized_keys
```

Обычно менять не нужно.

---

## `PasswordAuthentication`

У тебя:

```sshconfig
PasswordAuthentication no
```

Это запрещает вход по обычному паролю.

```sshconfig
PasswordAuthentication yes
```

Разрешает пароль.

```sshconfig
PasswordAuthentication no
```

Запрещает пароль.

Для сервера с SSH-ключами лучше:

```sshconfig
PasswordAuthentication no
```

`PasswordAuthentication` отвечает именно за парольную аутентификацию; по умолчанию она разрешена, если явно не переопределена.

Проверить, что пароль реально отключён:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@server.com
```

Если всё правильно, вход по паролю не должен пройти.

---

## `KbdInteractiveAuthentication`

У тебя:

```sshconfig
KbdInteractiveAuthentication no
```

Это отключает keyboard-interactive authentication.

Часто через этот механизм могут работать PAM, одноразовые коды, 2FA или дополнительные запросы.

Для простого сервера с ключами:

```sshconfig
KbdInteractiveAuthentication no
```

Если используется 2FA через PAM, этот параметр может понадобиться включить:

```sshconfig
KbdInteractiveAuthentication yes
```

---

## `UsePAM`

У тебя:

```sshconfig
UsePAM yes
```

PAM — это системный механизм Linux для проверки пользователей, сессий, лимитов, 2FA и других правил.

Обычно на Ubuntu/Debian оставляют:

```sshconfig
UsePAM yes
```

Важная связка:

```sshconfig
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
```

Так PAM остаётся для account/session-проверок, но парольный вход через SSH не включается.

OpenSSH указывает, что при `UsePAM yes` PAM может участвовать в `PasswordAuthentication` и `KbdInteractiveAuthentication`, поэтому при отключении паролей важно явно отключать и password, и keyboard-interactive.

---

## `PermitEmptyPasswords`

У тебя закомментировано:

```sshconfig
#PermitEmptyPasswords no
```

Запрещает вход пользователям с пустым паролем.

Лучше явно указать:

```sshconfig
PermitEmptyPasswords no
```

---

## `StrictModes`

```sshconfig
#StrictModes yes
```

Проверяет права на домашнюю директорию, `.ssh` и `authorized_keys`.

Лучше оставить:

```sshconfig
StrictModes yes
```

Если права слишком открытые, SSH может отказать во входе по ключу.

Правильные права:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

# Ограничение пользователей

## `AllowUsers`

Разрешить вход только конкретным пользователям:

```sshconfig
AllowUsers deploy admin
```

После этого по SSH смогут зайти только:

```text
deploy
admin
```

Ограничить пользователя по IP:

```sshconfig
AllowUsers deploy@203.0.113.*
```

Или несколько вариантов:

```sshconfig
AllowUsers deploy@203.0.113.* admin
```

`AllowUsers` разрешает вход только указанным пользователям; можно использовать форму `USER@HOST`.

---

## `DenyUsers`

Запретить вход конкретным пользователям:

```sshconfig
DenyUsers test guest
```

Фишка: чаще удобнее использовать `AllowUsers`, потому что это белый список.

---

## `AllowGroups`

Разрешить SSH только пользователям из группы:

```sshconfig
AllowGroups sshusers
```

Создать группу:

```bash
sudo groupadd sshusers
```

Добавить пользователя:

```bash
sudo usermod -aG sshusers deploy
```

---

# Лимиты входа

## `LoginGraceTime`

```sshconfig
#LoginGraceTime 2m
```

Сколько времени даётся на успешную авторизацию.

Пример:

```sshconfig
LoginGraceTime 30
```

Если пользователь не авторизовался за 30 секунд, подключение закрывается.

---

## `MaxAuthTries`

```sshconfig
#MaxAuthTries 6
```

Количество попыток авторизации за одно подключение.

Более строгий вариант:

```sshconfig
MaxAuthTries 3
```

---

## `MaxSessions`

```sshconfig
#MaxSessions 10
```

Количество сессий внутри одного SSH-соединения.

Обычно можно оставить:

```sshconfig
MaxSessions 10
```

---

## `MaxStartups`

```sshconfig
#MaxStartups 10:30:100
```

Ограничивает количество одновременных неавторизованных подключений.

Формат:

```text
старт:процент_отбрасывания:максимум
```

Пример:

```sshconfig
MaxStartups 10:30:60
```

Это защита от большого количества одновременных попыток подключения.

---

# Forwarding и туннели

## `AllowAgentForwarding`

```sshconfig
#AllowAgentForwarding yes
```

Разрешает проброс SSH Agent на сервер.

Если не нужен:

```sshconfig
AllowAgentForwarding no
```

Фишка: на production-серверах лучше отключать, если нет явной необходимости.

---

## `AllowTcpForwarding`

```sshconfig
#AllowTcpForwarding yes
```

Разрешает SSH-туннели:

```bash
ssh -L 8080:localhost:80 user@server
```

Если туннели не нужны:

```sshconfig
AllowTcpForwarding no
```

Если нужны только локальные туннели:

```sshconfig
AllowTcpForwarding local
```

Если нужны только удалённые:

```sshconfig
AllowTcpForwarding remote
```

OpenSSH поддерживает значения `yes/all`, `no`, `local`, `remote`; но отключение forwarding само по себе не считается полноценной защитой, если пользователь всё равно имеет shell-доступ.

---

## `GatewayPorts`

```sshconfig
#GatewayPorts no
```

Влияет на удалённый проброс портов `ssh -R`.

По умолчанию удалённый порт обычно слушает только loopback.

Опасный вариант:

```sshconfig
GatewayPorts yes
```

Он может сделать проброшенный порт доступным снаружи.

Обычно лучше:

```sshconfig
GatewayPorts no
```

---

## `X11Forwarding`

У тебя:

```sshconfig
X11Forwarding yes
```

Разрешает запуск графических приложений через SSH.

Пример:

```bash
ssh -X user@server.com
```

Для обычного сервера чаще лучше отключить:

```sshconfig
X11Forwarding no
```

OpenSSH прямо указывает, что X11 forwarding может создавать дополнительный риск для клиента, если пользователь запрашивает такой проброс.

---

## `DisableForwarding`

Жёстко отключает все виды forwarding:

```sshconfig
DisableForwarding yes
```

Отключает:

```text
TCP forwarding
X11 forwarding
agent forwarding
stream local forwarding
```

Удобно для ограниченных пользователей.

Пример:

```sshconfig
Match User backup
    DisableForwarding yes
```

`DisableForwarding` отключает все forwarding-функции и переопределяет отдельные forwarding-параметры.

---

# Поддержание соединения

## `ClientAliveInterval`

```sshconfig
#ClientAliveInterval 0
```

Через сколько секунд сервер проверяет, жив ли клиент.

Пример:

```sshconfig
ClientAliveInterval 300
```

---

## `ClientAliveCountMax`

```sshconfig
#ClientAliveCountMax 3
```

Сколько проверок клиент может не ответить.

Пример:

```sshconfig
ClientAliveInterval 300
ClientAliveCountMax 2
```

Это значит: сервер проверяет клиента каждые 300 секунд. Если клиент не ответил 2 раза, соединение закрывается.

ClientAlive-проверки идут через зашифрованный канал SSH и отличаются от обычного TCP keepalive.

---

# Логи

## `SyslogFacility`

```sshconfig
#SyslogFacility AUTH
```

Куда отправлять логи SSH.

Обычно:

```sshconfig
SyslogFacility AUTH
```

---

## `LogLevel`

```sshconfig
#LogLevel INFO
```

Уровень логирования.

Обычно:

```sshconfig
LogLevel INFO
```

Для диагностики:

```sshconfig
LogLevel VERBOSE
```

Для глубокой отладки временно:

```sshconfig
LogLevel DEBUG
```

Фишка: `DEBUG` не стоит держать постоянно.

Смотреть логи:

```bash
sudo journalctl -u ssh -f
```

Или:

```bash
sudo journalctl -u sshd -f
```

На Ubuntu/Debian часто:

```bash
sudo tail -f /var/log/auth.log
```

---

# Сообщения при входе

## `PrintMotd`

У тебя:

```sshconfig
PrintMotd no
```

Отключает вывод `/etc/motd` самим SSH-сервером.

Обычно на Ubuntu/Debian это нормально, потому что MOTD часто выводится через PAM.

---

## `PrintLastLog`

```sshconfig
#PrintLastLog yes
```

Показывает время последнего входа пользователя.

Обычно можно оставить:

```sshconfig
PrintLastLog yes
```

---

## `Banner`

```sshconfig
#Banner none
```

Показывает текст до авторизации.

Пример:

```sshconfig
Banner /etc/ssh/banner
```

Создать баннер:

```bash
sudo nano /etc/ssh/banner
```

Пример содержимого:

```text
Authorized access only.
```

---

# Переменные окружения

## `AcceptEnv`

У тебя:

```sshconfig
AcceptEnv LANG LC_*
```

Разрешает клиенту передавать переменные локали:

```text
LANG
LC_TIME
LC_ALL
LC_CTYPE
```

Это влияет на язык, кодировку, формат дат и похожие вещи.

Обычно можно оставить:

```sshconfig
AcceptEnv LANG LC_*
```

---

## `PermitUserEnvironment`

```sshconfig
#PermitUserEnvironment no
```

Разрешает пользователю задавать переменные окружения через:

```text
~/.ssh/environment
```

Обычно лучше:

```sshconfig
PermitUserEnvironment no
```

OpenSSH предупреждает, что разрешение пользовательского окружения может помогать обходить ограничения в некоторых конфигурациях.

---

# SFTP

## `Subsystem sftp`

У тебя:

```sshconfig
Subsystem sftp /usr/lib/openssh/sftp-server
```

Это включает SFTP.

После этого работает:

```bash
sftp user@server.com
```

Можно использовать внешний `sftp-server`:

```sshconfig
Subsystem sftp /usr/lib/openssh/sftp-server
```

Или встроенный вариант:

```sshconfig
Subsystem sftp internal-sftp
```

`internal-sftp` удобен для ограниченных SFTP-пользователей и chroot-настроек. OpenSSH поддерживает оба варианта: внешний `sftp-server` и встроенный `internal-sftp`.

---

# `Match` — настройки только для отдельных пользователей

`Match` позволяет применять настройки не ко всем, а только к конкретным пользователям, группам, адресам или другим условиям. После строки `Match` параметры действуют до следующего `Match` или до конца файла.

## Пример: отдельные правила для пользователя `backup`

```sshconfig
Match User backup
    PasswordAuthentication no
    PubkeyAuthentication yes
    X11Forwarding no
    AllowTcpForwarding no
```

---

## Пример: SFTP-only пользователь

```sshconfig
Subsystem sftp internal-sftp

Match User sftpuser
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    DisableForwarding yes
    PermitTTY no
    X11Forwarding no
```

Что это значит:

```text
ChrootDirectory       — ограничить пользователя директорией
ForceCommand          — запускать только SFTP
DisableForwarding     — отключить пробросы
PermitTTY no          — запретить shell-сессию
X11Forwarding no      — запретить X11
```

Важно для chroot:

```bash
sudo chown root:root /srv/sftp/sftpuser
sudo chmod 755 /srv/sftp/sftpuser
```

Папка для загрузок внутри:

```bash
sudo mkdir /srv/sftp/sftpuser/upload
sudo chown sftpuser:sftpuser /srv/sftp/sftpuser/upload
```

---

# Проверка конкретного пользователя через `sshd -T`

Показать итоговый конфиг:

```bash
sudo sshd -T
```

Показать итоговый конфиг как будто подключается пользователь `deploy`:

```bash
sudo sshd -T -C user=deploy,host=example.com,addr=203.0.113.10
```

Проверить конкретные параметры:

```bash
sudo sshd -T -C user=deploy,host=example.com,addr=203.0.113.10 | grep -E 'passwordauthentication|permitrootlogin|allowtcpforwarding|x11forwarding'
```

---

# Что я бы поменял в твоём конфиге

Сейчас у тебя спорные места:

```sshconfig
PermitRootLogin yes
X11Forwarding yes
```

Лучше заменить на:

```sshconfig
PermitRootLogin no
X11Forwarding no
```

Также добавить:

```sshconfig
PermitEmptyPasswords no
StrictModes yes
MaxAuthTries 3
LoginGraceTime 30
```

Если нужен только конкретный пользователь:

```sshconfig
AllowUsers deploy
```

---

# Пример нормального конфига для сервера по ключам

```sshconfig
Include /etc/ssh/sshd_config.d/*.conf

Port 22
AddressFamily any

PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
PermitEmptyPasswords no
StrictModes yes

LoginGraceTime 30
MaxAuthTries 3
MaxSessions 10

X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no

ClientAliveInterval 300
ClientAliveCountMax 2

PrintMotd no
PrintLastLog yes
AcceptEnv LANG LC_*

LogLevel INFO

Subsystem sftp internal-sftp
```

Если используешь SSH-туннели, не ставь:

```sshconfig
AllowTcpForwarding no
```

Тогда оставь:

```sshconfig
AllowTcpForwarding yes
```

---

# Пример более строгого конфига с доступом только для `deploy`

```sshconfig
Include /etc/ssh/sshd_config.d/*.conf

Port 22

PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
PermitEmptyPasswords no
StrictModes yes

AllowUsers deploy

LoginGraceTime 30
MaxAuthTries 3
MaxSessions 5
MaxStartups 10:30:60

X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no

ClientAliveInterval 300
ClientAliveCountMax 2

PrintMotd no
PrintLastLog yes
AcceptEnv LANG LC_*

Subsystem sftp internal-sftp
```

---

# Быстрые команды администратора

Открыть конфиг:

```bash
sudo nano /etc/ssh/sshd_config
```

Проверить ошибки:

```bash
sudo sshd -t
```

Показать итоговый конфиг:

```bash
sudo sshd -T
```

Показать важные параметры:

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|usepam|x11forwarding|allowusers|allowtcpforwarding)'
```

Применить изменения:

```bash
sudo systemctl reload ssh
```

Или:

```bash
sudo systemctl reload sshd
```

Проверить статус:

```bash
sudo systemctl status ssh
```

Или:

```bash
sudo systemctl status sshd
```

Смотреть логи:

```bash
sudo journalctl -u ssh -f
```

Проверить подключение по ключу:

```bash
ssh -i ~/.ssh/id_ed25519 user@server.com
```

Проверить, что пароль отключён:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@server.com
```

---

# Главное запомнить

Файл:

```text
/etc/ssh/sshd_config
```

Проверить перед применением:

```bash
sudo sshd -t
```

Посмотреть итоговую конфигурацию:

```bash
sudo sshd -T
```

Безопасная база:

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
PermitEmptyPasswords no
StrictModes yes
X11Forwarding no
```

Самая опасная ошибка:

```sshconfig
PermitRootLogin yes
PasswordAuthentication yes
```

Нормальный вариант:

```sshconfig
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```