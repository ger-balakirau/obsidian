# Безопасная настройка SSH-сервера

Эта заметка — практический рецепт. Подробный справочник параметров находится в [[10 - SSH server config - sshd_config]].

## Цель

Сделать SSH-доступ безопаснее:

```text
запретить вход root
отключить вход по паролю
разрешить вход только по ключам
ограничить пользователей
убрать лишние возможности
проверять конфиг перед применением
```

---

## Перед изменениями

Не закрывай текущую SSH-сессию.

Сначала сделай резервную копию:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Открой конфиг:

```bash
sudo nano /etc/ssh/sshd_config
```

Проверить конфиг перед применением:

```bash
sudo sshd -t
```

Показать итоговую конфигурацию:

```bash
sudo sshd -T
```

`sshd -t` проверяет конфиг и ключи, а `sshd -T` выводит итоговые применённые настройки.

---

## Безопасный базовый конфиг

```sshconfig
Port 22

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
MaxSessions 5
MaxStartups 10:30:60

X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no

ClientAliveInterval 300
ClientAliveCountMax 2

PrintMotd no
PrintLastLog yes
LogLevel INFO

Subsystem sftp /usr/lib/openssh/sftp-server
```

---

## Запрет входа под root

Плохо:

```sshconfig
PermitRootLogin yes
```

Лучше:

```sshconfig
PermitRootLogin no
```

После этого нельзя подключаться так:

```bash
ssh root@server.com
```

Нужно заходить обычным пользователем:

```bash
ssh deploy@server.com
```

А потом повышать права:

```bash
sudo -i
```

`PermitRootLogin no` полностью запрещает root-вход по SSH. Значение `prohibit-password` запрещает root-вход по паролю, но оставляет вход по ключу.

---

## Вход только по SSH-ключам

Включить вход по ключам:

```sshconfig
PubkeyAuthentication yes
```

Отключить вход по паролю:

```sshconfig
PasswordAuthentication no
```

Отключить keyboard-interactive:

```sshconfig
KbdInteractiveAuthentication no
```

Такой набор означает:

```text
по ключу — можно
по паролю — нельзя
через keyboard-interactive — нельзя
```

`PasswordAuthentication` управляет входом по паролю, а `KbdInteractiveAuthentication` — keyboard-interactive-аутентификацией. У обоих параметров есть значения `yes` или `no`.

---

## PAM оставить включённым

Обычно на Linux-серверах оставляют:

```sshconfig
UsePAM yes
```

Нормальная связка:

```sshconfig
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
```

---

## Запрет пустых паролей

```sshconfig
PermitEmptyPasswords no
```

Это запрещает вход пользователям с пустым паролем.

---

## Проверка прав на `.ssh`

```sshconfig
StrictModes yes
```

SSH будет проверять права на домашнюю директорию, `.ssh` и `authorized_keys`.

Правильные права:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Для приватного ключа:

```bash
chmod 600 ~/.ssh/id_ed25519
```

---

## Разрешить вход только нужным пользователям

Пример:

```sshconfig
AllowUsers deploy admin
```

После этого войти смогут только:

```text
deploy
admin
```

Пример ограничения по IP:

```sshconfig
AllowUsers deploy@203.0.113.*
```

---

## Ограничить попытки входа

```sshconfig
LoginGraceTime 30
MaxAuthTries 3
MaxStartups 10:30:60
```

Где:

```text
LoginGraceTime 30     — 30 секунд на авторизацию
MaxAuthTries 3        — максимум 3 попытки
MaxStartups 10:30:60  — ограничение неавторизованных подключений
```

---

## Отключить X11 forwarding

```sshconfig
X11Forwarding no
```

Для обычного сервера графический проброс не нужен.

---

## Отключить agent forwarding

```sshconfig
AllowAgentForwarding no
```

Agent forwarding нужен редко.  
На production-серверах лучше отключать.

---

## Отключить port forwarding

```sshconfig
AllowTcpForwarding no
```

Это отключит SSH-туннели:

```bash
ssh -L 8080:localhost:80 user@server.com
```

Если на сервере нужны SSH-туннели, оставь:

```sshconfig
AllowTcpForwarding yes
```

---

## Запретить внешний remote forwarding

```sshconfig
GatewayPorts no
```

Это безопаснее, чем:

```sshconfig
GatewayPorts yes
```

`GatewayPorts yes` может открыть удалённый проброшенный порт наружу.

---

## Запретить VPN-туннели через SSH

```sshconfig
PermitTunnel no
```

Обычно SSH VPN-туннели не нужны.

---

## Автоотключение мёртвых сессий

```sshconfig
ClientAliveInterval 300
ClientAliveCountMax 2
```

Это значит:

```text
сервер проверяет клиента каждые 300 секунд
если клиент не отвечает 2 раза — соединение закрывается
```

---

## Логирование

```sshconfig
LogLevel INFO
```

Для диагностики временно можно:

```sshconfig
LogLevel VERBOSE
```

Смотреть логи:

```bash
sudo journalctl -u ssh -f
```

Или:

```bash
sudo journalctl -u sshd -f
```

---

## Если меняешь порт SSH

Пример:

```sshconfig
Port 2222
```

Подключение:

```bash
ssh -p 2222 deploy@server.com
```

Открыть порт в firewall:

```bash
sudo ufw allow 2222/tcp
```

Потом только после проверки можно закрывать старый порт.

---

## Применение изменений

Проверить конфиг:

```bash
sudo sshd -t
```

Если ошибок нет:

```bash
sudo systemctl reload ssh
```

Если сервис называется `sshd`:

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

---

## Проверка после настройки

Открыть новое окно терминала и проверить вход:

```bash
ssh deploy@server.com
```

Проверить, что пароль отключён:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no deploy@server.com
```

Если парольный вход отключён правильно, подключение не должно пройти.

Проверить итоговые параметры:

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|allowusers|x11forwarding|allowtcpforwarding|gatewayports)'
```

---

## Готовый вариант для обычного сервера

```sshconfig
Port 22

PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

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
GatewayPorts no
PermitTunnel no

ClientAliveInterval 300
ClientAliveCountMax 2

PrintMotd no
PrintLastLog yes
LogLevel INFO

Subsystem sftp /usr/lib/openssh/sftp-server
```

---

## Главное запомнить

Минимальная безопасная база:

```sshconfig
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitEmptyPasswords no
StrictModes yes
```

Проверить перед применением:

```bash
sudo sshd -t
```

Применить:

```bash
sudo systemctl reload ssh
```

Не закрывать текущую SSH-сессию, пока не проверишь новое подключение.
