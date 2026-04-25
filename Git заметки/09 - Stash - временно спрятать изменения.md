# Stash - временно спрятать изменения

`git stash` временно сохраняет незакоммиченные изменения и очищает working tree. Это удобно, когда нужно быстро переключиться на другую ветку или сделать `pull`, но commit ещё рано делать.

## Спрятать изменения

```bash
git stash
```

С сообщением:

```bash
git stash push -m "work in progress"
```

## Посмотреть список stash

```bash
git stash list
```

Пример:

```text
stash@{0}: On main: work in progress
```

## Вернуть последний stash и удалить его из списка

```bash
git stash pop
```

## Вернуть stash, но оставить его в списке

```bash
git stash apply stash@{0}
```

## Удалить stash

```bash
git stash drop stash@{0}
```

Удалить все stash:

```bash
git stash clear
```

## Спрятать untracked files

Обычно `git stash` не забирает untracked files.

```bash
git stash -u
```

## Частый сценарий

```bash
git status
git stash push -m "before switching branch"
git switch main
git pull
git switch feature/name
git stash pop
```

## Главное запомнить

- `stash` - временная полка, не замена commit.
- `stash pop` применяет и удаляет stash.
- `stash apply` применяет, но оставляет stash в списке.
- Для важных изменений лучше сделать commit.
