# Глубокий разбор playbook

Playbook состоит из одного или нескольких plays.

Play отвечает на вопросы:

```text
на каких хостах выполнять
с какими правами
какие переменные использовать
какие tasks выполнить
какие handlers вызвать
```

## Минимальный playbook

```yaml
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
```

Разбор:

```text
name   - описание play
hosts  - группа или хост из inventory
become - выполнять задачи через sudo
tasks  - список задач
apt    - модуль Ansible для apt-пакетов
```

## Play

Пример:

```yaml
- name: Configure all Linux servers
  hosts: all
  become: true
```

`hosts` может быть:

```text
all
web
db
proxy
web:&ubuntu
web:!old
```

## Task

Task - одно действие.

```yaml
- name: Install curl
  ansible.builtin.apt:
    name: curl
    state: present
```

Хороший `name` должен отвечать на вопрос:

```text
что делает задача
```

## Modules

Ansible modules - готовые действия.

Частые модули:

```text
ansible.builtin.apt      - пакеты Debian/Ubuntu
ansible.builtin.copy     - копирование файла
ansible.builtin.template - Jinja2-шаблон
ansible.builtin.service  - управление сервисом
ansible.builtin.file     - файлы, директории, права
ansible.builtin.user     - пользователи
ansible.builtin.group    - группы
ansible.builtin.lineinfile - изменить строку в файле
ansible.builtin.command  - выполнить команду без shell
ansible.builtin.shell    - выполнить команду через shell
```

## become

Если нужны root-права:

```yaml
become: true
```

Можно указать пользователя:

```yaml
become: true
become_user: root
```

Если нужен sudo-пароль:

```bash
ansible-playbook -i inventory.ini site.yml -K
```

## Variables

Переменные внутри playbook:

```yaml
- name: Configure nginx
  hosts: web
  become: true

  vars:
    nginx_port: 8080

  tasks:
    - name: Show port
      ansible.builtin.debug:
        msg: "Nginx port is {{ nginx_port }}"
```

Переменная используется через:

```jinja2
{{ nginx_port }}
```

## Копирование файла

```yaml
- name: Copy index.html
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: "0644"
```

## Шаблон template

Файл `templates/nginx.conf.j2`:

```nginx
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
}
```

Task:

```yaml
- name: Deploy nginx config
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/app.conf
  notify: Reload nginx
```

## Handlers

Handler запускается только если task что-то изменил.

```yaml
handlers:
  - name: Reload nginx
    ansible.builtin.service:
      name: nginx
      state: reloaded
```

Task вызывает handler через:

```yaml
notify: Reload nginx
```

Это удобно для конфигов: если конфиг не поменялся, сервис не перезагружается.

## Условия when

```yaml
- name: Install nginx only on Ubuntu
  ansible.builtin.apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```

Факты Ansible посмотреть так:

```bash
ansible all -i inventory.ini -m setup
```

## Циклы loop

Установить несколько пакетов:

```yaml
- name: Install base packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - curl
    - git
    - htop
```

Чаще для apt можно проще:

```yaml
- name: Install base packages
  ansible.builtin.apt:
    name:
      - curl
      - git
      - htop
    state: present
```

## Register

Сохранить результат команды:

```yaml
- name: Check nginx config
  ansible.builtin.command: nginx -t
  register: nginx_test
  changed_when: false

- name: Show nginx test result
  ansible.builtin.debug:
    var: nginx_test.stdout
```

`changed_when: false` нужен, чтобы проверочная команда не помечалась как изменение.

## Tags

Tags позволяют запускать часть playbook.

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
  tags:
    - nginx
    - packages
```

Запустить только tag:

```bash
ansible-playbook -i inventory.ini site.yml --tags nginx
```

Пропустить tag:

```bash
ansible-playbook -i inventory.ini site.yml --skip-tags nginx
```

Показать tags:

```bash
ansible-playbook -i inventory.ini site.yml --list-tags
```

## Полный пример playbook

```yaml
- name: Configure nginx web server
  hosts: web
  become: true

  vars:
    nginx_port: 80
    server_name: app.lab

  tasks:
    - name: Install packages
      ansible.builtin.apt:
        name:
          - nginx
          - curl
        state: present
        update_cache: true
      tags:
        - packages

    - name: Deploy nginx config
      ansible.builtin.template:
        src: templates/app.conf.j2
        dest: /etc/nginx/sites-available/app.conf
        owner: root
        group: root
        mode: "0644"
      notify: Reload nginx
      tags:
        - nginx

    - name: Enable nginx site
      ansible.builtin.file:
        src: /etc/nginx/sites-available/app.conf
        dest: /etc/nginx/sites-enabled/app.conf
        state: link
      notify: Reload nginx

    - name: Ensure nginx is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

## Структура проекта побольше

```text
ansible-lab/
  inventory.ini
  site.yml
  files/
    index.html
  templates/
    app.conf.j2
  group_vars/
    web.yml
  host_vars/
    web1.yml
```

`group_vars/web.yml`:

```yaml
nginx_port: 80
server_name: app.lab
```

## Проверки перед запуском

Синтаксис:

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

Какие хосты будут затронуты:

```bash
ansible-playbook -i inventory.ini site.yml --list-hosts
```

Dry-run:

```bash
ansible-playbook -i inventory.ini site.yml --check
```

Diff:

```bash
ansible-playbook -i inventory.ini site.yml --diff
```

Dry-run + diff:

```bash
ansible-playbook -i inventory.ini site.yml --check --diff
```

## Идемпотентность

Хороший playbook можно запускать много раз.

Первый запуск:

```text
changed
```

Повторный запуск:

```text
ok
changed=0
```

Если каждый запуск что-то меняет без причины, playbook написан плохо или используется неидемпотентная команда.

## command vs shell

`command` безопаснее:

```yaml
- name: Check uptime
  ansible.builtin.command: uptime
```

`shell` нужен, если есть shell-синтаксис:

```yaml
- name: Find nginx processes
  ansible.builtin.shell: "ps aux | grep nginx"
```

Не используй `shell`, если есть нормальный модуль Ansible.

## Минимум на память

```bash
ansible-playbook -i inventory.ini site.yml
ansible-playbook -i inventory.ini site.yml --syntax-check
ansible-playbook -i inventory.ini site.yml --check --diff
ansible-playbook -i inventory.ini site.yml --limit web1
ansible-playbook -i inventory.ini site.yml --tags nginx
ansible all -i inventory.ini -m setup
```

## Связанные заметки

- [[01 - Быстрый запуск playbook]]
- [[02 - Inventory и подключение по SSH]]
- [[Systemd заметки/02 - systemctl - управление сервисами]]
- [[Linux заметки/06 - Архивы в Linux - tar gzip zip]]
