1. Результат выполнения playbook для test.yml
```
root@vagrant:/home/vagrant/ansible/mnt-homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/test.yml site.yml

PLAY [Print os facts] *******************************************************************************************************


TASK [Gathering Facts] ******************************************************************************************************

ok: [localhost]

TASK [Print OS] *************************************************************************************************************

ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************

ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ******************************************************************************************************************

localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
2. После изменения some_facts
```

PLAY [Print os facts] ***********

TASK [Gathering Facts] **********

ok: [localhost]

TASK [Print OS] *****************

ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***************

ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP **********************

localhost                  : ok=3
```
3. Тестовое окружение уже было создано ранее.
4. Результат выполнения playbook для prod.yml
```
PLAY [Print os facts] ************************


TASK [Gathering Facts] ***********************

ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ******************************

ok: [centos7] => {
    "msg": "Ubuntu"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ****************************

ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP ***********************************

centos7                    : ok=3    changed=0
ubuntu                     : ok=3    changed=0
```
