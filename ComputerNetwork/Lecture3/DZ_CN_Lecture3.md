1. Подключился телнетом к route-views.routeviews.org. Ниже результаты выполненных команд:
* show ip route 83.220.236.129 - ![Task1-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task1-1.png)
* show bgp 83.220.236.129 - ![Task1-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task1-2.png)
2. Включил dummy интерфейсы в ядре echo "dummy" >> /etc/modules. В конфигурационный файл /etc/modprobe.d/dummy.conf можно внести значения о кол-ве аодключаемых интерфейсов, например echo "options dummy numdummies=2" > /etc/modprobe.d/dummy.conf. Добавление статических муршрутов можно выполнить с помощью команды  ip route add 10.0.2.80/32 via 10.0.2.2 dev dummy0. и в табличке маршрутизации появляется запись unicast 10.0.2.80 via 10.0.2.2 dev dummy0 proto boot scope global. Для внесения данных на постонний остнове необходимо внести данные о нтерфейсе dummy в /etc/network/interfaces, пример:
* Кусок конфига из /etc/network/interfaces - ![Task2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task2.png)
3. Используя команду ss -tnap и ss -tnapr получил всю необходимую информацию, вывод команд:
* Результат ss -tnap - ![Task3-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task3.png)
* Результат ss -tnapr - ![Task3-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task3-2.png)
4. Комнанда для простмотра UPD сокетов ss -unapr, вывод команды
* Результат ss -unapr - ![Task4](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture3/CN3-task4.png)
