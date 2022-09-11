1. Ответы на впросы по первому заданию
 * Паттерны IaaC являются определенным набором правил и практик работы с ифраструктурой, програмным кодом и доставкой програмного кода до определенного окружения.
 * Сам по себе паттерн IaaC является основополвгвющим и дает начало всем последующим правилам и практикам, которые включает в себя.
2. тветы на впросы по второму заданию
 * Осноное отличие Ansible от других ситем управленя кофнигурациями заключается в том, что он может исользовать уже созданную ssh инфраструктуру.
 * По моему мнению, pull метод более подходящий чем push, при методе pull машины сами запрашивают измнения когда им это необходимо, при методе push хост не всегда может быть доступен и может не получить обновления или изменения, что в последующем может вызвать проблемы.
3. Версии программ по порядку
 * ``C:\Program Files\Oracle\VirtualBox>VBoxManage.exe --version | 6.1.34r150636``
 * ``D:\Maxim\Education\Netology\Vagrant>vagrant -v | Vagrant 2.2.19``
 * ``root@vagrant:/etc/ansible# ansible --version | ansible 2.9.6``
4. При выполнении задания появляется ошибка, выполнял на локальном ПК, ОС Windows 7
 * ``D:\Maxim\Education\Netology\Vagrant>vagrant provision
    ==> server1.netology: Running provisioner: ansible...
    Windows is not officially supported for the Ansible Control Machine.
    Please check https://docs.ansible.com/intro_installation.html#control-machine-requirements
    Vagrant gathered an unknown Ansible version:


    and falls back on the compatibility mode '1.8'.

    Alternatively, the compatibility mode can be specified in your Vagrantfile:
    https://www.vagrantup.com/docs/provisioning/ansible_common.html#compatibility_mode
    server1.netology: Running ansible-playbook...
    The Ansible software could not be found! Please verify
    that Ansible is correctly installed on your host system.

    If you haven't installed Ansible yet, please install Ansible
    on your host system. Vagrant can't do this for you in a safe and
    automated way.
    Please check https://docs.ansible.com for more information.``
