# Команды Git - общая шпаргалка

Быстрый индекс команд. Подробности разнесены по отдельным заметкам.

## Состояние и история

Подробнее: [[02 - Базовый рабочий цикл]]

```bash
git status
git diff
git diff --staged
git log --oneline
git log --oneline --graph --decorate --all
git show
```

## Staging и commit

```bash
git add file.txt
git add src/
git add .
git restore --staged file.txt
git commit -m "Message"
git commit --amend
```

## Ветки

Подробнее: [[04 - Ветки и переключение контекста]]

```bash
git branch
git branch -a
git switch main
git switch -c feature/name
git merge feature/name
git branch -d feature/name
```

## Remote

Подробнее: [[05 - Remote, push и pull]]

```bash
git remote -v
git fetch
git pull
git push
git push -u origin feature/name
```

## Отмена изменений

Подробнее: [[07 - Отмена изменений и восстановление]]

```bash
git restore file.txt
git restore --staged file.txt
git reset --soft HEAD~1
git revert <commit>
```

## `.gitignore`

Подробнее: [[06 - Gitignore и чистота репозитория]]

```bash
git status --ignored
git rm --cached file
git rm -r --cached dir
```

## Диагностика

Подробнее: [[08 - Частые ошибки Git]]

```bash
git status
git remote -v
git branch -vv
git log --oneline --graph --decorate --all
```

## Минимум на память

```bash
git status
git add .
git commit -m "Message"
git pull
git push
```
