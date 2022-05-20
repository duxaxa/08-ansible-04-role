#### 1. Создать в старой версии playbook файл requirements.yml и заполнить его следующим содержимым:  
[playbook/requirements.yml](playbook/requirements.yml)



#### 2. При помощи `ansible-galaxy` скачать себе эту роль.  

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ ansible-galaxy role install -p roles -r requirements.yml
Starting galaxy role install process
- extracting clickhouse to /ansible/08-ansible-04-role/playbook/roles/clickhouse
- clickhouse (1.11.0) was installed successfully

```



#### 3. Создать новый каталог с ролью при помощи:   
`ansible-galaxy role init vector-role`  
`ansible-galaxy role init lighthouse-role`  

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ cd roles/

vagrant@test-netology:/ansible/08-ansible-04-role/playbook/roles $ ansible-galaxy role init vector-role
- Role vector-role was created successfully

vagrant@test-netology:/ansible/08-ansible-04-role/playbook/roles $ ansible-galaxy role init lighthouse-role
- Role lighthouse-role was created successfully
```



#### 4. На основе tasks из старого playbook заполните новую role. Разнесите переменные между vars и default.  
#### 5. Перенести нужные шаблоны конфигов в templates. 
#### 6. Описать в README.md обе роли и их параметры.  
#### 7. Повторите шаги 3-6 для lighthouse. Помните, что одна роль должна настраивать один продукт.  

#### vector-role:  

role: [vector-role/tasks/main.yml](playbook/roles/vector-role/tasks/main.yml)  
handlers: [vector-role/handlers/main.yml](playbook/roles/vector-role/handlers/main.yml)  
шаблон `templates/vector/vector.yaml.j2` перемещен в каталог роли [vector-role/templates/vector.yaml.j2](playbook/roles/vector-role/templates/vector.yaml.j2)  
vars: [vector-role/vars/main.yml](playbook/roles/vector-role/vars/main.yml)  
в переменные роли вынесены параметры `пользователь и его группа в ОС, - владелец ПО vector`, переопределение которых на других окружениях не требуется:  
```yaml
vector_os_user: "vector"
vector_os_group: "vector"
```

defaults:  [vector-role/defaults/main.yml](playbook/roles/vector-role/defaults/main.yml)  
в переменные по умолчанию вынесены параметры `версия vector`, `архтектура ОС`, 
`временный рабочий каталог для размещения дистрибутива с vector`, `каталог кофигов для vector`, 
которые с большой вероятностью могут быть переопределены по мере изменения версии vector; рабочий каталог и каталог 
конфигов тоже может быть изменен на других окружениях.
```yaml
vector_version: "0.21.1"
vector_os_arh: "x86_64"
vector_workdir: "/tmp/vector"
vector_data_dir: "/var/lib/vector"
```



<details>
<summary>ansible-playbook -i inventory/vagrant.yml site.yml --tags vector</summary>

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ ansible-playbook -i inventory/vagrant.yml site.yml --tags vector

PLAY [Install Clickhouse] **********************************************************************************************************************************************************

PLAY [Install Vector] **************************************************************************************************************************************************************

TASK [vector-role : Vector. Create work directory] *********************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Get Vector distributive] *******************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Unzip archive] *****************************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Install vector binary file] ****************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Check Vector installation] *****************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create etc directory] **********************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create Vector config vector.yaml] **********************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create vector.service daemon] **************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Modify vector.service file ExecStart] ******************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Modify vector.service file ExecStartPre] ***************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create user vector] ************************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create data_dir] ***************************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Remove work directory] *********************************************************************************************************************************
changed: [vector-01]

RUNNING HANDLER [vector-role : Start Vector service] *******************************************************************************************************************************
changed: [vector-01]

PLAY [Install Lighthouse] **********************************************************************************************************************************************************

PLAY RECAP *************************************************************************************************************************************************************************
vector-01                  : ok=14   changed=14   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>
  



#### lighthouse-role:  

role: [lighthouse-role/tasks/main.yml](playbook/roles/lighthouse-role/tasks/main.yml)  
handlers: [lighthouse-role/handlers/main.yml](playbook/roles/lighthouse-role/handlers/main.yml)  
шаблон `templates/nginx/ligthouse.conf.j2` перемещен в каталог роли [lighthouse-role/templates/nginx/ligthouse.conf.j2](playbook/roles/lighthouse-role/templates/nginx/ligthouse.conf.j2)  
vars:  [lighthouse-role/vars/main.yml](playbook/roles/lighthouse-role/vars/main.yml)  
в переменные роли вынесены параметры `репозиторий с исходным кодом lighthouse`, `пакеты, обязательные к установке`, 
являющиеся неотъемлемой частью роли и поэтому не требующие возможности переопределения:  
```yaml
lighthouse_code_src: "https://github.com/VKCOM/lighthouse.git"
lighthouse_packages:
  - git
  - nginx
```
defaults:  [lighthouse-role/defaults/main.yml](playbook/roles/lighthouse-role/defaults/main.yml)  
в переменные по умолчанию вынесены параметры `каталог установки lighthouse`, `порт веб-сервера`, `имя конфига веб-сервера`, 
которые с большой вероятностью могут быть переопределены на других окружениях.
```yaml
lighthouse_data_dir: "/lighthouse/"
lighthouse_nginx_port: 8888
lighthouse_nginx_conf: "lighthouse.conf"
```

<details>
<summary>ansible-playbook -i inventory/vagrant.yml site.yml --tags lighthouse</summary>

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ ansible-playbook -i inventory/vagrant.yml site.yml --tags lighthouse

PLAY [Install Clickhouse] **********************************************************************************************************************************************************

PLAY [Install Vector] **************************************************************************************************************************************************************

PLAY [Install Lighthouse] **********************************************************************************************************************************************************

TASK [lighthouse-role : Lighthouse. Pre-install nginx & git client] ****************************************************************************************************************
changed: [lighthouse-01]

TASK [lighthouse-role : Lighthouse. Clone source code by git client] ***************************************************************************************************************
changed: [lighthouse-01]

TASK [lighthouse-role : Lighthouse. Prepare nginx config] **************************************************************************************************************************
changed: [lighthouse-01]

RUNNING HANDLER [lighthouse-role : Start Lighthouse service] ***********************************************************************************************************************
changed: [lighthouse-01]

PLAY RECAP *************************************************************************************************************************************************************************
lighthouse-01              : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>



#### 8. Выложите все roles в репозитории. Проставьте тэги, используя семантическую нумерацию Добавьте roles в requirements.yml в playbook.  

[vector-role, tag: 1.0.0](https://github.com/duxaxa/vector-role/releases/tag/1.0.0)    
[lighthouse-role, tag: 1.0.0](https://github.com/duxaxa/lighthouse-role/releases/tag/1.0.0)      
[requirements.yml](playbook/requirements.yml)





#### 9. Переработайте playbook на использование roles. Не забудьте про зависимости lighthouse и возможности совмещения roles с tasks.

[site.yml](playbook/site.yml)

Работа playbook и предварительное скачивание требуемых ролей, описанных в `requirements.yml`:    

<details>
<summary>ansible-galaxy role install -p roles -r requirements.yml</summary>

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ ansible-galaxy role install -p roles -r requirements.yml 
Starting galaxy role install process
- extracting clickhouse to /ansible/08-ansible-04-role/playbook/roles/clickhouse
- clickhouse (1.11.0) was installed successfully
- extracting vector-role to /ansible/08-ansible-04-role/playbook/roles/vector-role
- vector-role (1.0.0) was installed successfully
- extracting lighthouse-role to /ansible/08-ansible-04-role/playbook/roles/lighthouse-role
- lighthouse-role (1.0.0) was installed successfully

```

</details>


<details>
<summary>ansible-playbook -i inventory/vagrant.yml site.yml</summary>

```shell
vagrant@test-netology:/ansible/08-ansible-04-role/playbook $ ansible-playbook -i inventory/vagrant.yml site.yml

PLAY [Install Clickhouse] **********************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Include OS Family Specific Variables] ***************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/precheck.yml for clickhouse-01

TASK [clickhouse : Requirements check | Checking sse4_2 support] *******************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Requirements check | Not supported distribution && release] *****************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/params.yml for clickhouse-01

TASK [clickhouse : Set clickhouse_service_enable] **********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Set clickhouse_service_ensure] **********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/install/apt.yml for clickhouse-01

TASK [clickhouse : Install by APT | Apt-key add repo key] **************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Install by APT | Remove old repo] *******************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Install by APT | Repo installation] *****************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Install by APT | Package installation] **************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Install by APT | Package installation] **************************************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : Hold specified version during APT upgrade | Package installation] ***********************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
ok: [clickhouse-01] => (item=clickhouse-common-static)

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/configure/sys.yml for clickhouse-01

TASK [clickhouse : Check clickhouse config, data and logs] *************************************************************************************************************************
ok: [clickhouse-01] => (item=/var/log/clickhouse-server)
ok: [clickhouse-01] => (item=/etc/clickhouse-server)
ok: [clickhouse-01] => (item=/var/lib/clickhouse/tmp/)
ok: [clickhouse-01] => (item=/var/lib/clickhouse/)

TASK [clickhouse : Config | Create config.d folder] ********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Config | Create users.d folder] *********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Config | Generate system config] ********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Config | Generate users config] *********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Config | Generate remote_servers config] ************************************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : Config | Generate macros config] ********************************************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : Config | Generate zookeeper servers config] *********************************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : Config | Fix interserver_http_port and intersever_https_port collision] *****************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : Notify Handlers Now] ********************************************************************************************************************************************

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/service.yml for clickhouse-01

TASK [clickhouse : Ensure clickhouse-server.service is enabled: True and state: started] *******************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Wait for Clickhouse Server to Become Ready] *********************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/configure/db.yml for clickhouse-01

TASK [clickhouse : Set ClickHose Connection String] ********************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Gather list of existing databases] ******************************************************************************************************************************
ok: [clickhouse-01]

TASK [clickhouse : Config | Delete database config] ********************************************************************************************************************************

TASK [clickhouse : Config | Create database config] ********************************************************************************************************************************

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
included: /ansible/08-ansible-04-role/playbook/roles/clickhouse/tasks/configure/dict.yml for clickhouse-01

TASK [clickhouse : Config | Generate dictionary config] ****************************************************************************************************************************
skipping: [clickhouse-01]

TASK [clickhouse : include_tasks] **************************************************************************************************************************************************
skipping: [clickhouse-01]

PLAY [Install Vector] **************************************************************************************************************************************************************

TASK [vector-role : Vector. Create work directory] *********************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Get Vector distributive] *******************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Unzip archive] *****************************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Install vector binary file] ****************************************************************************************************************************
ok: [vector-01]

TASK [vector-role : Vector. Check Vector installation] *****************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create etc directory] **********************************************************************************************************************************
ok: [vector-01]

TASK [vector-role : Vector. Create Vector config vector.yaml] **********************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create vector.service daemon] **************************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Modify vector.service file ExecStart] ******************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Modify vector.service file ExecStartPre] ***************************************************************************************************************
changed: [vector-01]

TASK [vector-role : Vector. Create user vector] ************************************************************************************************************************************
ok: [vector-01]

TASK [vector-role : Vector. Create data_dir] ***************************************************************************************************************************************
ok: [vector-01]

TASK [vector-role : Vector. Remove work directory] *********************************************************************************************************************************
changed: [vector-01]

RUNNING HANDLER [vector-role : Start Vector service] *******************************************************************************************************************************
ok: [vector-01]

PLAY [Install Lighthouse] **********************************************************************************************************************************************************

TASK [lighthouse-role : Lighthouse. Pre-install nginx & git client] ****************************************************************************************************************
ok: [lighthouse-01]

TASK [lighthouse-role : Lighthouse. Clone source code by git client] ***************************************************************************************************************
ok: [lighthouse-01]

TASK [lighthouse-role : Lighthouse. Prepare nginx config] **************************************************************************************************************************
ok: [lighthouse-01]

PLAY RECAP *************************************************************************************************************************************************************************
clickhouse-01              : ok=26   changed=0    unreachable=0    failed=0    skipped=10   rescued=0    ignored=0   
lighthouse-01              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=14   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>