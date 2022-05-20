vector-role
=========

Роль выполняет:
1. установку Vector из архива с дистрибутивом 
2. первичную конфигурацию Vector с демо конфигурацией по генерации событий syslog и отправку данные в БД Clickhouse
3. созданию юнита systemd для Vector
4. создание пользователя ОС для Vector

Requirements
------------

1. Ubuntu ОС
2. Предустановленная БД Clickhouse.

Role Variables
--------------

`vars/main.yml`:
Если требуется, можно изменить пользователя ОС владельца По Vector:    
```yaml
vector_os_user: "vector"
vector_os_group: "vector"
```

`defaults/main.yml`:
Параметры, определяющие версию Vector для установки, архитектуру ОС, рабочий каталог для установки Vector и каталог для данных Vector:   
```yaml
vector_version: "0.21.1"
vector_os_arh: "x86_64"
vector_workdir: "/tmp/vector"
vector_data_dir: "/var/lib/vector"
```

Dependencies
------------

Для установки Clickhouse требуется роль `ansible-clickhouse`: https://github.com/AlexeySetevoi/ansible-clickhouse  
(или аналогичная по функционалу роль).



Example Playbook
----------------

Пример добавление роли в playbook:  
```yaml
- name: Install Vector
  hosts: vector
  gather_facts: false
  roles:
    - role: vector-role
      tags: vector
```      

License
-------

BSD


