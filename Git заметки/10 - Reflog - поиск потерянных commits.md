# Reflog - поиск потерянных commits

`git reflog` показывает, куда раньше указывали `HEAD` и ветки. Это часто спасает после ошибочного `reset`, `rebase`, `commit --amend` или переключений.

## Посмотреть reflog

```bash
git reflog
```

Пример:

```text
abc1234 HEAD@{0}: reset: moving to HEAD~1
def5678 HEAD@{1}: commit: Add deploy config
```

Если commit пропал из обычного `git log`, он всё ещё может быть в `reflog`.

## Восстановить потерянный commit в новую ветку

Самый безопасный вариант:

```bash
git switch -c rescue def5678
```

Теперь commit сохранён в отдельной ветке.

## Вернуться к состоянию из reflog

Осторожно:

```bash
git reset --hard def5678
```

Перед этим лучше:

```bash
git status
git branch backup-before-reset
```

## Когда reflog полезен

- случайно сделал `git reset --hard`;
- сделал `commit --amend` и потерял старый commit;
- неудачно сделал rebase;
- удалил ветку, но помнишь, что там были commits.

## Главное запомнить

- `reflog` - локальная история перемещений HEAD.
- Лучший способ спасения - создать ветку от найденного commit.
- `reflog` не заменяет remote backup и нормальные commits.
