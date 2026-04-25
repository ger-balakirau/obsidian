# Частые ошибки Git

## `nothing to commit`

Git не видит изменений для commit.

Проверить:

```bash
git status
```

Возможные причины:

- файлы не изменены;
- изменения не добавлены через `git add`;
- файлы игнорируются `.gitignore`.

## `Please tell me who you are`

Git не знает имя и email автора commits.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Проверить:

```bash
git config --global --list
```

## `non-fast-forward`

На remote есть изменения, которых нет локально.

Обычно:

```bash
git pull
git push
```

Если есть конфликты, их нужно решить вручную.

## Merge conflict

Git не смог автоматически объединить изменения.

Проверить:

```bash
git status
```

После ручного исправления файлов:

```bash
git add file
git commit
```

## `fatal: not a git repository`

Ты не внутри Git-репозитория.

Проверить текущую папку:

```bash
pwd
ls -la
```

Или подняться выше:

```bash
cd ..
```

## `Permission denied (publickey)`

Проблема SSH-доступа к remote.

Проверить remote:

```bash
git remote -v
```

Проверить SSH:

```bash
ssh -T git@github.com
```

Связанные темы:

- [[Git заметки/05 - Remote, push и pull]]
- [[Linux заметки/ssh/15 - Диагностика SSH-подключения]]
- [[Linux заметки/ssh/17 - Частые ошибки SSH]]

## Главное запомнить

При ошибке сначала запускай:

```bash
git status
git remote -v
git branch -vv
```
