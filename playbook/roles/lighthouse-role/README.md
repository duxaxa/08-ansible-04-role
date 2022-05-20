lighthouse-role
=========

Роль выполняет:  
1. установку nginx для использования в качестве веб-сервера   
2. клонирование git-репозитория Lighthouse с исходным кодом  
3. создание конфигурационного файла nginx для доступа к статическим ресурсам Lighthouse  
4. запуск nginx  

Requirements
------------

1. Ubuntu ОС
2. Предустановленная БД Clickhouse  

Role Variables
--------------

`vars/main.yml`:  
Содержит в параметрах ссылку на git-репозитория Lighthouse с исходным кодом и список пакетов, обязательных к установке для работы роли:     
```yaml
lighthouse_code_src: "https://github.com/VKCOM/lighthouse.git"
lighthouse_packages:
  - git
  - nginx
```

`defaults/main.yml`:  
Параметры, определяющие каталог для установки Lighthouse, порт веб-сервера и имя конфигурационного файла nginx:     
```yaml
lighthouse_data_dir: "/lighthouse/"
lighthouse_nginx_port: 8888
lighthouse_nginx_conf: "lighthouse.conf"
```

Dependencies
------------

Для установки Clickhouse требуется роль `ansible-clickhouse`: https://github.com/AlexeySetevoi/ansible-clickhouse  
(или аналогичная по функционалу роль).

Example Playbook
----------------

Пример добавление роли в playbook:  
```yaml
- name: Install Lighthouse
  hosts: lighthouse
  gather_facts: false
  roles:
    - role: lighthouse-role
      tags: lighthouse
```


License
-------

BSD

