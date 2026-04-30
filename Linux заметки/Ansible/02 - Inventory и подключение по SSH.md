# Inventory и подключение по SSH

Inventory - список серверов, которыми управляет Ansible.

Ansible почти всегда подключается к Linux-серверам по SSH.

## Простой inventory

`inventory.ini`:

```ini
[web]
10.10.0.11
10.10.0.12

[db]
10.10.0.13
```

Проверка:

```bash
ansible all -i inventory.ini -m ping
```

## Inventory с именами

```ini
[web]
web1 ansible_host=10.10.0.11
web2 ansible_host=10.10.0.12

[db]
db1 ansible_host=10.10.0.13
```

Здесь:

```text
web1         - имя внутри Ansible
ansible_host - реальный IP или DNS-имя
```

## Указать пользователя

Для всей группы:

```ini
[web]
web1 ansible_host=10.10.0.11
web2 ansible_host=10.10.0.12

[web:vars]
ansible_user=devops
```

Для одного хоста:

```ini
web1 ansible_host=10.10.0.11 ansible_user=devops
```

## Подключение по SSH-ключу

Inventory:

```ini
[web]
web1 ansible_host=10.10.0.11 ansible_user=devops ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Проверка обычным SSH:

```bash
ssh -i ~/.ssh/id_ed25519 devops@10.10.0.11
```

Проверка Ansible:

```bash
ansible web -i inventory.ini -m ping
```

## Подключение по паролю

Inventory:

```ini
[web]
web1 ansible_host=10.10.0.11 ansible_user=devops
```

Запуск с запросом SSH-пароля:

```bash
ansible web -i inventory.ini -m ping --ask-pass
```

Коротко:

```bash
ansible web -i inventory.ini -m ping -k
```

Для playbook:

```bash
ansible-playbook -i inventory.ini site.yml -k
```

Если Ansible просит `sshpass`, установить:

```bash
sudo apt install -y sshpass
```

Важно: для постоянной работы лучше SSH-ключи, а не пароль.

## Пароль sudo / become

Если команды требуют `sudo`, в playbook обычно пишут:

```yaml
become: true
```

Запуск с запросом sudo-пароля:

```bash
ansible-playbook -i inventory.ini site.yml --ask-become-pass
```

Коротко:

```bash
ansible-playbook -i inventory.ini site.yml -K
```

SSH-пароль + sudo-пароль:

```bash
ansible-playbook -i inventory.ini site.yml -k -K
```

## Указать порт SSH

```ini
[web]
web1 ansible_host=10.10.0.11 ansible_user=devops ansible_port=2222
```

Проверка:

```bash
ssh -p 2222 devops@10.10.0.11
ansible web -i inventory.ini -m ping
```

## Подключение через jump host

Если сервер доступен только через proxy/bastion, удобнее настроить `~/.ssh/config`.

Пример:

```sshconfig
Host lab-proxy
    HostName 10.10.0.10
    User devops

Host lab-app
    HostName 10.10.0.11
    User devops
    ProxyJump lab-proxy
```

Inventory:

```ini
[web]
lab-app
```

Проверка:

```bash
ssh lab-app
ansible all -i inventory.ini -m ping
```

## Полезные ansible-переменные подключения

```ini
ansible_host=10.10.0.11
ansible_user=devops
ansible_port=22
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3
```

## Пример полного inventory

```ini
[proxy]
proxy1 ansible_host=10.10.0.10

[web]
web1 ansible_host=10.10.0.11
web2 ansible_host=10.10.0.12

[db]
db1 ansible_host=10.10.0.13

[all:vars]
ansible_user=devops
ansible_python_interpreter=/usr/bin/python3
```

Проверить все:

```bash
ansible all -i inventory.ini -m ping
```

Проверить только группу:

```bash
ansible web -i inventory.ini -m ping
```

## Частые ошибки подключения

### UNREACHABLE

Проверить обычный SSH:

```bash
ssh devops@10.10.0.11
```

Проверить порт:

```bash
nc -vz 10.10.0.11 22
```

### Permission denied

Причины:

- неверный пользователь;
- не тот SSH-ключ;
- парольный вход отключён;
- ключ не добавлен в `authorized_keys`.

### Missing sudo password

Запустить с:

```bash
ansible-playbook -i inventory.ini site.yml -K
```

### Host key checking failed

Сначала подключиться обычным SSH:

```bash
ssh devops@10.10.0.11
```

И подтвердить fingerprint.

Для lab можно временно отключить host key checking:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible all -i inventory.ini -m ping
```

Но в нормальной среде лучше не отключать проверку без причины.

## Минимум на память

```bash
ansible all -i inventory.ini -m ping
ansible web -i inventory.ini -m ping -k
ansible-playbook -i inventory.ini site.yml -k -K
ssh -i ~/.ssh/id_ed25519 devops@10.10.0.11
nc -vz 10.10.0.11 22
```

## Связанные заметки

- [[01 - Быстрый запуск playbook]]
- [[Linux заметки/ssh/09 - SSH client config]]
- [[Linux заметки/ssh/18 - Каскадный SSH]]
- [[Linux заметки/04 - Ubuntu lab proxy bastion и reverse proxy]]
