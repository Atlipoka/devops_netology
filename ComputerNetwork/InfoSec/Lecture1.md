1. Установил расширение Bitwarden для Chrome, зарегистрировался и сохранил креды.
* ![Task-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task1.png)
2. Скачал приложение Google Auth и привязал к аккаунту в Bitwarden вдухэтапную аутентификацию
* ![Task-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task2.png)
3. Распишу по шагам (конфигурировал apache используюя инструкцию - https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04)
* apt install apache2
* Команда для генерации сертификати - openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crtsudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
* Настроил виртуальный хост, где прописал все настройки
 * ![Task-3](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task3.png)
* Перезапустил apache2, пробросил порты на VB, на локальном ПК прописал в host ДНС и IP сайта, открыл его в браузере
 * ![Task-3](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task3-2.png)
4. Для проверки сайта на уязвимости использовал инструкцию на сайте - https://www.tecmint.com/testssl-sh-test-tls-ssl-encryption-in-linux-commandline/
* Результат работы - ![Task-4](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task4.png)
5. Распишу по шагам
* Сгенерировал ключ с помощью команды ssh-keygen
* Используя вагрант развернул вторую ВМ и настроил в конфиге вагранта DHCP - config.vm.network "public_network", use_dhcp_assigned_default_route: true
* Используя команду ssh-copy-id vagrant@192.168.88.198 подключился к "соседней" ВМ, результат:
 * ![Task-5](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task5.png)
6. На предыдушем шаге генерировал ключ под пользователем root, потом пришлось повторно упражнятся с ключами под пользователем vagrant. Создал в домашней директории, в папке .ssh/, файл config и прописал следующие настройки:
* Host vagrant, User vagrant, HostName 192.168.88.198, Port 22
* Далее подключился к ВМ используя команду ssh vagrant
 * ![Task-6](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task6.png)
8. Распишу по шагам
* Собрал 100 пакетов утилитой tcpdump и зприсал результат в файл - tcpdump -i eth0 -c 100 -w file.pcap
* Установли tshark -  apt install tshark
* Прочитал файл tshark-ом - tshark -r file.pcap
 * ![Task-7](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task7.png)
