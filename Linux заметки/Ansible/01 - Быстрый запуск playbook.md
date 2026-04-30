# Быстрый запуск playbook

Playbook - YAML-файл, где описано, что Ansible должен сделать на серверах.

Пример задач:

```text
установить nginx
скопировать конфиг
перезапустить сервис
создать пользователя
обновить пакеты
```

## Установить Ansible

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y ansible
```

Проверить:

```bash
ansible --version
```

## Минимальная структура

```text
ansible-lab/
  inventory.ini
  site.yml
```

Создать папку:

```bash
mkdir ansible-lab
cd ansible-lab
```

## Inventory

Файл `inventory.ini`:

```ini
[web]
10.10.0.11

[db]
10.10.0.12
```

Проверить доступность:

```bash
ansible all -i inventory.ini -m ping
```

Ожидаемо:

```text
10.10.0.11 | SUCCESS
10.10.0.12 | SUCCESS
```

Важно: модуль `ping` в Ansible не ICMP ping. Он проверяет, что Ansible может подключиться по SSH и выполнить Python-код на удалённой машине.

## Первый playbook

Файл `site.yml`:

```yaml
- name: Install nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Enable and start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

Запуск:

```bash
ansible-playbook -i inventory.ini site.yml
```

## Запуск с sudo-паролем

Если удалённый пользователь требует пароль для `sudo`:

```bash
ansible-playbook -i inventory.ini site.yml --ask-become-pass
```

Коротко:

```bash
ansible-playbook -i inventory.ini site.yml -K
```

`-K` спросит become password, то есть пароль для `sudo`.

## Запуск с SSH-паролем

Если SSH-ключей нет и вход по SSH идёт по паролю:

```bash
ansible-playbook -i inventory.ini site.yml --ask-pass
```

Коротко:

```bash
ansible-playbook -i inventory.ini site.yml -k
```

Если нужен и SSH-пароль, и sudo-пароль:

```bash
ansible-playbook -i inventory.ini site.yml -k -K
```

Для подключения по паролю может понадобиться пакет `sshpass`:

```bash
sudo apt install -y sshpass
```

## Проверить синтаксис

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

## Пробный запуск без изменений

```bash
ansible-playbook -i inventory.ini site.yml --check
```

`--check` показывает, что Ansible хотел бы изменить, но не всегда все модули идеально поддерживают dry-run.

## Показать, какие хосты будут затронуты

```bash
ansible-playbook -i inventory.ini site.yml --list-hosts
```

## Запустить только на одном сервере

```bash
ansible-playbook -i inventory.ini site.yml --limit 10.10.0.11
```

Или по имени из inventory:

```bash
ansible-playbook -i inventory.ini site.yml --limit web1
```

## Запустить с подробным выводом

```bash
ansible-playbook -i inventory.ini site.yml -v
ansible-playbook -i inventory.ini site.yml -vvv
```

`-vvv` полезен при проблемах с SSH.

## Минимум команд

```bash
ansible --version
ansible all -i inventory.ini -m ping
ansible-playbook -i inventory.ini site.yml
ansible-playbook -i inventory.ini site.yml -k
ansible-playbook -i inventory.ini site.yml -K
ansible-playbook -i inventory.ini site.yml -k -K
ansible-playbook -i inventory.ini site.yml --check
ansible-playbook -i inventory.ini site.yml --syntax-check
```

## Связанные заметки

- [[02 - Inventory и подключение по SSH]]
- [[03 - Глубокий разбор playbook]]
- [[Linux заметки/ssh/02 - Подключение по SSH через консоль]]
