# Типы SSH-ключей и надёжность

Для новых ключей обычно выбирай `Ed25519`: он современный, короткий и удобный. `RSA` всё ещё встречается и подходит для совместимости со старыми системами. `DSA` лучше не использовать.

## Ed25519

Современный рекомендуемый вариант:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Файлы:

```text
~/.ssh/id_ed25519      — приватный ключ
~/.ssh/id_ed25519.pub  — публичный ключ
```

Подключение:

```bash
ssh -i ~/.ssh/id_ed25519 user@server.com
```

---

## RSA

Старый, но всё ещё часто используемый вариант:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Файлы:

```text
~/.ssh/id_rsa      — приватный ключ
~/.ssh/id_rsa.pub  — публичный ключ
```

Подключение:

```bash
ssh -i ~/.ssh/id_rsa user@server.com
```

---

## ECDSA

Ещё один вариант ключей:

```bash
ssh-keygen -t ecdsa -b 521 -C "your_email@example.com"
```

Файлы:

```text
~/.ssh/id_ecdsa      — приватный ключ
~/.ssh/id_ecdsa.pub  — публичный ключ
```

Подключение:

```bash
ssh -i ~/.ssh/id_ecdsa user@server.com
```

---

## DSA

Устаревший вариант. Обычно использовать не нужно.

```bash
ssh-keygen -t dsa
```

Файлы:

```text
~/.ssh/id_dsa
~/.ssh/id_dsa.pub
```

## Главное запомнить

- Новый ключ: чаще всего `ed25519`.
- Для совместимости: `rsa -b 4096`.
- Не используй `dsa` для новых подключений.
