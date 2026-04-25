# Gitignore и чистота репозитория

`.gitignore` говорит Git, какие файлы не нужно добавлять в репозиторий.

## Что обычно игнорируют

- секреты: `.env`, ключи, токены;
- зависимости: `node_modules/`, `vendor/`;
- артефакты сборки: `dist/`, `build/`;
- логи: `*.log`;
- временные файлы редактора и ОС.

## Пример `.gitignore`

```gitignore
.env
*.log
node_modules/
dist/
build/
.DS_Store
Thumbs.db
```

## Проверить игнорируемые файлы

```bash
git status --ignored
```

## Если файл уже попал в Git

Добавить файл в `.gitignore` недостаточно, если Git уже отслеживает этот файл.

Убрать из индекса, но оставить на диске:

```bash
git rm --cached .env
```

Для директории:

```bash
git rm -r --cached node_modules/
```

Потом сделать commit:

```bash
git add .gitignore
git commit -m "Update gitignore"
```

## Проверить, почему файл игнорируется

```bash
git check-ignore -v file
```

## Главное запомнить

- `.gitignore` работает для неотслеживаемых файлов.
- Если файл уже в Git, используй `git rm --cached`.
- Секреты нельзя хранить в репозитории.
