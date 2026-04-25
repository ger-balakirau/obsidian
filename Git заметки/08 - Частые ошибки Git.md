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

## `Your local changes would be overwritten`

Git не даёт переключиться или сделать pull, потому что локальные изменения будут перезаписаны.

Варианты:

Сохранить изменения в commit:

```bash
git add .
git commit -m "Save work"
```

Временно спрятать:

```bash
git stash
```

Выкинуть изменения, если они точно не нужны:

```bash
git reset --hard HEAD
```

## `Untracked working tree files would be overwritten`

Есть неотслеживаемые файлы, которые конфликтуют с файлами из ветки или remote.

Проверить:

```bash
git status
```

Удалить неотслеживаемые файлы осторожно:

```bash
git clean -fdn
git clean -fd
```

## Случайно сделал `reset --hard`

Не паникуй, сначала посмотри:

```bash
git reflog
```

Если нужный commit есть в reflog:

```bash
git switch -c rescue <commit>
```

Подробно: [[10 - Reflog - поиск потерянных commits]]

## Нужно сделать force push

Сначала проверь, что это твоя ветка, а не общая ветка команды.

Безопаснее:

```bash
git fetch
git push --force-with-lease
```

Опаснее:

```bash
git push --force
```

`--force` может перезаписать чужие commits на remote. Для обычной работы почти всегда лучше `--force-with-lease`.

## Главное запомнить

При ошибке сначала запускай:

```bash
git status
git remote -v
git branch -vv
git reflog
```
