# Remote, push и pull

Remote - это удалённый репозиторий, например на GitHub, GitLab или Bitbucket.

## Посмотреть remote

```bash
git remote -v
```

Пример:

```text
origin  git@github.com:user/project.git (fetch)
origin  git@github.com:user/project.git (push)
```

## Добавить remote

```bash
git remote add origin git@github.com:user/project.git
```

## Получить изменения без слияния

```bash
git fetch
```

`fetch` обновляет информацию о remote-ветках, но не меняет твои файлы.

## Получить и применить изменения

```bash
git pull
```

По сути это:

```text
fetch + merge или rebase
```

## Отправить изменения

```bash
git push
```

Для новой ветки:

```bash
git push -u origin feature/name
```

После `-u` дальше можно писать просто:

```bash
git push
```

## Посмотреть связь локальной и remote-ветки

```bash
git branch -vv
```

## SSH-доступ к GitHub/GitLab

Git через SSH использует те же базовые идеи ключей, что и обычный SSH.

Полезные связанные темы:

- [[Linux заметки/ssh/04 - Типы SSH-ключей и надежность]]
- [[Linux заметки/ssh/05 - Генерация SSH-ключа]]
- [[Linux заметки/ssh/06 - Где хранятся SSH-ключи]]
- [[Linux заметки/ssh/12 - SSH agent]]

Проверка GitHub SSH:

```bash
ssh -T git@github.com
```

Проверка GitLab SSH:

```bash
ssh -T git@gitlab.com
```

## Главное запомнить

- `remote` - удалённый репозиторий.
- `fetch` получает информацию, но не меняет рабочие файлы.
- `pull` получает и применяет изменения.
- `push` отправляет твои commits.
- `git push -u origin branch` связывает локальную ветку с remote-веткой.
