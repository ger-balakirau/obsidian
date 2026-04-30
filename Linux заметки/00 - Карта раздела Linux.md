# Linux - карта раздела

Эта карта отвечает только за Linux-раздел: базовые команды, права, SSH и первичную настройку Ubuntu Server.

## Порядок изучения

1. [[01 - Linux - базовая навигация, поиск и файлы]]
2. [[Linux заметки/Пользователи группы/00 - Карта темы|Пользователи, группы и права]]
3. [[Linux заметки/ssh/00 - Карта темы|SSH]]
4. [[02 - Ubuntu Server - базовая настройка]]
5. [[03 - Смена hostname в Linux]]
6. [[04 - Ubuntu lab proxy bastion и reverse proxy]]
7. [[05 - apt через https]]
8. [[Linux заметки/Docker/00 - Карта темы|Docker]]
9. [[06 - Архивы в Linux - tar gzip zip]]
10. [[Linux заметки/Ansible/00 - Карта темы|Ansible]]

## Что закрывает раздел

- Навигация по файловой системе.
- Работа с файлами и директориями.
- Поиск файлов и текста.
- Пайпы, фильтрация, просмотр логов.
- Пользователи, группы и права.
- Смена пароля `root` и безопасная работа через `sudo`.
- SSH-доступ и ключи.
- Базовая настройка Ubuntu Server.
- `apt` через HTTPS и базовые пакеты для внешних репозиториев.
- Смена hostname и связь hostname с DNS/hosts.
- Lab-схема Ubuntu proxy/bastion для SSH и reverse proxy.
- Docker Engine, Docker Compose plugin и базовый запуск контейнеров.
- Архивы: упаковка, распаковка, выбор отдельных файлов и масок.
- Ansible: inventory, запуск playbook, SSH-подключение и автоматизация.

## Связанные разделы

- [[Systemd заметки/00 - Карта темы|Systemd]]
- [[Сети/00 - Карта темы|Сети]]
- [[Траблшутинг/00 - Карта темы|Troubleshooting]]
- [[00 - Главная карта базы знаний|Главная карта базы знаний]]
- [[01 - Маршруты обучения|Маршруты обучения]]

## Минимум на память

```bash
pwd
ls -la
cd
find
grep
less
tail -f
chmod
chown
sudo passwd root
hostnamectl
sudo hostnamectl set-hostname proxy-vm
sudo apt install -y apt-transport-https ca-certificates
docker compose version
tar -xzf archive.tar.gz
zip -r archive.zip folder/
ansible-playbook -i inventory.ini site.yml
ssh user@host
```
