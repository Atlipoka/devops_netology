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
5. Изменил фокты для окружения prod.yml
```
PLAY [Print os facts] ************


TASK [Gathering Facts] ***********

ok: [centos7]
ok: [ubuntu]

TASK [Print OS] ******************

ok: [centos7] => {
    "msg": "Ubuntu"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ****************

ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ***********************

centos7                    : ok=3
ubuntu                     : ok=3
```
7. Зашифровал факты
```
root@vagrant:/home/vagrant/ansible/mnt-homeworks/08-ansible-01-base/playbook/group_vars/deb# cat examp.yml
$ANSIBLE_VAULT;1.1;AES256
32393166396530383762636334393536666636356634383630643330376336323133306339633538
6532656266666265643632323366363635623766656363330a643731643332316631333631633362
64643839333535663365333331313731666163316138653330366432393537343363393330343966
6539376266396663610a633366333463616536316632383431303861376263386430633465623766
39316530343136396463393038363132326665336563346439373965333464386462306137613436
3230303434373666376266323333323363633463623932663064
root@vagrant:/home/vagrant/ansible/mnt-homeworks/08-ansible-01-base/playbook/group_vars/deb# cd ..
root@vagrant:/home/vagrant/ansible/mnt-homeworks/08-ansible-01-base/playbook/group_vars# cd el/
root@vagrant:/home/vagrant/ansible/mnt-homeworks/08-ansible-01-base/playbook/group_vars/el# cat examp.yml
$ANSIBLE_VAULT;1.1;AES256
31363762333231356431663561666137396531306466613736356462373264313566363762643865
6330633863613735393462386230643232643062646338620a313534316263356436623939323737
33646436633537623136343734336562366663666235346635386365633238396564393134623232
6331363035346634640a613239366538313535643539396330366532336539393862353031643533
62366661653939393762306533383338313362326264626437306133346433643933316436333032
3261663665643862656362386236326137356631376364663937
```
8. апапрапр
