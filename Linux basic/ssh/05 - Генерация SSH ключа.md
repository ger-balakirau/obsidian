Рекомендуемый тип ключа:

```bash
ed25519
```

Создать ключ со своим именем:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/my_key -C "comment"
```

Пример для GitHub:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_personal -C "github personal"
```

Будут созданы два файла:

```text
~/.ssh/github_personal      # приватный ключ
~/.ssh/github_personal.pub  # публичный ключ
```

Приватный ключ — никому не передавать.  
Публичный ключ — можно добавлять на сервер, GitHub, GitLab и другие сервисы.

---

## Что значит `-f`

Параметр `-f` задаёт имя и путь файла ключа:

```bash
-f ~/.ssh/github_personal
```

То есть ключ сохранится не как стандартный:

```text
~/.ssh/id_ed25519
```

а как:

```text
~/.ssh/github_personal
```

---

## Что значит `-C`

Параметр `-C` добавляет комментарий:

```bash
-C "github personal"
```

Комментарий нужен только для удобства. Например, чтобы потом понять, для чего этот ключ.

---

## Создание ключа без пароля

```bash
ssh-keygen -t ed25519 -f ~/.ssh/deploy_key -C "deploy key" -N ""
```

`-N ""` означает пустой пароль.

Такой вариант удобно использовать для автоматизации, но он менее безопасен.

---

## Создание ключа с паролем

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_personal -C "github personal"
```

Когда появится:

```text
Enter passphrase:
```

введи пароль.

Когда появится:

```text
Enter same passphrase again:
```

повтори пароль.

---

## Посмотреть публичный ключ

```bash
cat ~/.ssh/github_personal.pub
```

Именно этот вывод копируют на сервер или в GitHub/GitLab.

---

## Подключиться с этим ключом

```bash
ssh -i ~/.ssh/github_personal user@server.com
```

С портом:

```bash
ssh -i ~/.ssh/github_personal -p 2222 user@server.com
```

---

## Частые имена ключей

```text
github_personal
github_work
gitlab
server_prod
server_stage
deploy_key
```

---

## Главное

Создание ключа с нужным именем:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/my_key_name -C "comment"
```

Подключение с этим ключом:

```bash
ssh -i ~/.ssh/my_key_name user@server.com
```