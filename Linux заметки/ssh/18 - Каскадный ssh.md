## Что такое каскадный SSH

Каскадный SSH — это подключение к серверу через один или несколько промежуточных серверов.

Схема:

```text
твой компьютер → jump-сервер → целевой сервер
```

Промежуточный сервер часто называют:

```text
jump host
bastion host
gateway
```

---

## Простой вариант в два шага

Сначала подключиться к промежуточному серверу:

```bash
ssh jumpuser@jump.example.com
```

Потом уже с него подключиться к целевому серверу:

```bash
ssh targetuser@target.example.com
```

Минус: второй SSH-запуск происходит уже с jump-сервера.

---

## Правильный вариант через `-J`

Ключ `-J` позволяет подключиться через jump host одной командой:

```bash
ssh -J jumpuser@jump.example.com targetuser@target.example.com
```

Пример:

```bash
ssh -J admin@185.10.20.10 deploy@10.0.0.5
```

Схема:

```text
локальный компьютер → 185.10.20.10 → 10.0.0.5
```

---

## Jump host с нестандартным портом

Если jump-сервер слушает SSH на порту `2222`:

```bash
ssh -J jumpuser@jump.example.com:2222 targetuser@target.example.com
```

Пример:

```bash
ssh -J admin@185.10.20.10:2222 deploy@10.0.0.5
```

---

## Целевой сервер с нестандартным портом

Если целевой сервер слушает порт `2222`:

```bash
ssh -J jumpuser@jump.example.com -p 2222 targetuser@target.example.com
```

Пример:

```bash
ssh -J admin@185.10.20.10 -p 2222 deploy@10.0.0.5
```

---

## Несколько jump-серверов

Можно подключаться через цепочку серверов:

```bash
ssh -J user1@jump1.example.com,user2@jump2.example.com user3@target.example.com
```

Схема:

```text
твой компьютер → jump1 → jump2 → target
```

Пример:

```bash
ssh -J admin@1.1.1.1,deploy@10.0.0.2 app@10.0.0.5
```

---

## Настройка через `~/.ssh/config`

Чтобы не писать длинную команду каждый раз:

```sshconfig
Host jump
    HostName 185.10.20.10
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host target
    HostName 10.0.0.5
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ProxyJump jump
```

После этого подключение короткое:

```bash
ssh target
```

---

## Несколько jump-серверов в config

```sshconfig
Host jump1
    HostName 1.1.1.1
    User admin

Host jump2
    HostName 10.0.0.2
    User deploy
    ProxyJump jump1

Host target
    HostName 10.0.0.5
    User app
    ProxyJump jump2
```

Подключение:

```bash
ssh target
```

---

## Важно про ключи

При использовании `-J` или `ProxyJump` приватный ключ остаётся на твоём компьютере.

Не нужно копировать приватный ключ на jump-сервер.

Правильно:

```text
ключ лежит локально → подключение идёт через jump → target проверяет твой ключ
```

Плохо:

```text
копировать приватный ключ на jump-сервер
```

---

## Проверка с подробным выводом

```bash
ssh -v -J jumpuser@jump.example.com targetuser@target.example.com
```

Ещё подробнее:

```bash
ssh -vvv -J jumpuser@jump.example.com targetuser@target.example.com
```

---

## Частая ошибка

```text
Permission denied (publickey)
```

Причина: нет подходящего ключа для jump-сервера или target-сервера.

Решение: явно указать ключи в `~/.ssh/config`.

Пример:

```sshconfig
Host jump
    HostName 185.10.20.10
    User admin
    IdentityFile ~/.ssh/id_ed25519_jump

Host target
    HostName 10.0.0.5
    User deploy
    IdentityFile ~/.ssh/id_ed25519_target
    ProxyJump jump
```

---

# Главное запомнить

Подключение через jump host:

```bash
ssh -J jumpuser@jump.example.com targetuser@target.example.com
```

Через несколько серверов:

```bash
ssh -J user1@jump1,user2@jump2 user3@target
```

Через config:

```sshconfig
Host target
    HostName 10.0.0.5
    User deploy
    ProxyJump jump
```

Подключение:

```bash
ssh target
```