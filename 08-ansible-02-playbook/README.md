# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook.
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`.

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

## Необязательная часть

1. Приготовьте дополнительный хост для установки logstash.
2. Пропишите данный хост в `prod.yml` в новую группу `logstash`.
3. Дополните playbook ещё одним play, который будет исполнять установку logstash только на выделенный для него хост.
4. Все переменные для нового play определите в отдельный файл `group_vars/logstash/vars.yml`.
5. Logstash конфиг должен конфигурироваться в части ссылки на elasticsearch (можно взять, например его IP из facts или определить через vars).
6. Дополните README.md, протестируйте playbook, выложите новую версию в github. В ответ предоставьте ссылку на репозиторий.

---

# Задание 1
```
---  
     elasticsearch:  
       hosts:  
         elastic001:  
           ansible_connection: docker  
     kibana:  
       hosts:  
         kibana001:  
           ansible_connection: docker  
---  
```  

# Задание 4  

```
- name: Install kibana  
   hosts: kibana  
   tasks:  
     - name: Upload tar.gz kibana from remote URL  
       get_url:  
         url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"  
         dest: "/tmp/kibana-{{ elastic_version }}-linux-x86_64.tar.gz"  
         mode: 0755  
         timeout: 60  
         force: true  
         validate_certs: false  
       register: get_kibana  
       until: get_kibana is succeeded  
       tags: kibana  
     - name: Create directrory for kibana  
       file:  
         state: directory  
         path: "{{ kibana_home }}"  
       tags: kibana  
     - name: Extract Kibana in the installation directory  
       #become: true  
       unarchive:  
         copy: false  
         src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"  
         dest: "{{ kibana_home }}"  
         extra_opts: [--strip-components=1]  
         creates: "{{ kibana_home }}/bin/kibana"  
       tags:  
         - skip_ansible_lint  
         - kibana  
     - name: Set environment Kibana  
       #become: true  
       template:  
         src: templates/kib.sh.j2  
         dest: /etc/profile.d/kib.sh  
       tags: kibana  
```  

# Задание 5  

       ~/DevOpsStudy/mnt-homeworks/08-ansible-02-playbook/playbook❯ ansible-lint site.yml -vvv  
       Examining site.yml of type playbook  

# Задание 6  

```  
~/DevOpsStudy/mnt-homeworks/08-ansible-02-playbook/playbook❯ ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
changed: [kibana001]
changed: [elastic001]

TASK [Ensure installation dir exists] **********************************************************************************************************
changed: [elastic001]
changed: [kibana001]

TASK [Extract java in the installation directory] **********************************************************************************************
fatal: [elastic001]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.11' must be an existing dir"}
fatal: [kibana001]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.11' must be an existing dir"}

PLAY RECAP *************************************************************************************************************************************
elastic001                 : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
kibana001                  : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

```

#Задание 7  

```

~/DevOpsStudy/mnt-homeworks/08-ansible-02-playbook/playbook❯ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana001]
diff skipped: source file size is greater than 104448
changed: [elastic001]

TASK [Ensure installation dir exists] **********************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.11",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana001]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.11",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic001]

TASK [Extract java in the installation directory] **********************************************************************************************
changed: [elastic001]
changed: [kibana001]

TASK [Export environment variables] ************************************************************************************************************
--- before
+++ after: /home/solomatinkirill/.ansible/tmp/ansible-local-221239khw7w2m_/tmp_hy3d13m/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.11
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [kibana001]
--- before
+++ after: /home/solomatinkirill/.ansible/tmp/ansible-local-221239khw7w2m_/tmpgb0gs20t/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.11
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [elastic001]

PLAY [Install Elasticsearch] *******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic001]

TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************************************
changed: [elastic001]

TASK [Create directrory for Elasticsearch] *****************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/7.10.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic001]

TASK [Extract Elasticsearch in the installation directory] *************************************************************************************
changed: [elastic001]

TASK [Set environment Elastic] *****************************************************************************************************************
--- before
+++ after: /home/solomatinkirill/.ansible/tmp/ansible-local-221239khw7w2m_/tmpxfcyrgrw/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/7.10.1
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [elastic001]

PLAY [Install Kibana] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [kibana001]

TASK [Upload tar.gz Kibana from remote URL] ****************************************************************************************************
changed: [kibana001]

TASK [Create directrory for Kibana (/opt/kibana/7.12.0)] ***************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/7.12.0",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana001]

TASK [Extract Kibana in the installation directory] ********************************************************************************************
changed: [kibana001]

TASK [Set environment Kibana] ******************************************************************************************************************
--- before
+++ after: /home/solomatinkirill/.ansible/tmp/ansible-local-221239khw7w2m_/tmpq2uv5zf0/kib.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIBANA_HOME=/opt/kibana/7.12.0
+export PATH=$PATH:$KIBANA_HOME/bin
\ No newline at end of file

changed: [kibana001]

PLAY RECAP *************************************************************************************************************************************
elastic001                 : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
kibana001                  : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0



```

# Задача 8  

```
~/DevOpsStudy/mnt-homeworks/08-ansible-02-playbook/playbook❯ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Set facts for Java 8 vars] ***************************************************************************************************************
ok: [kibana001]
ok: [elastic001]

TASK [Upload .tar.gz file containing binaries from local storage] ******************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Ensure installation dir exists] **********************************************************************************************************
ok: [elastic001]
ok: [kibana001]

TASK [Extract java in the installation directory] **********************************************************************************************
skipping: [elastic001]
skipping: [kibana001]

TASK [Export environment variables] ************************************************************************************************************
ok: [elastic001]
ok: [kibana001]

PLAY [Install Elasticsearch] *******************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [elastic001]

TASK [Upload tar.gz Elasticsearch from remote URL] *********************************************************************************************
ok: [elastic001]

TASK [Create directrory for Elasticsearch] *****************************************************************************************************
ok: [elastic001]

TASK [Extract Elasticsearch in the installation directory] *************************************************************************************
skipping: [elastic001]

TASK [Set environment Elastic] *****************************************************************************************************************
ok: [elastic001]

PLAY [Install Kibana] **************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [kibana001]

TASK [Upload tar.gz Kibana from remote URL] ****************************************************************************************************
ok: [kibana001]

TASK [Create directrory for Kibana (/opt/kibana/7.12.0)] ***************************************************************************************
ok: [kibana001]

TASK [Extract Kibana in the installation directory] ********************************************************************************************
skipping: [kibana001]

TASK [Set environment Kibana] ******************************************************************************************************************
ok: [kibana001]

PLAY RECAP *************************************************************************************************************************************
elastic001                 : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
kibana001                  : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   


```
