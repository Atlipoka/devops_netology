1. При запросе telnet stackoverflow.com 80 и отправке сообщения GET /questions HTTP/1.0 HOST: stackoverflow.com, был получен HTTP ответ HTTP/1.1 301 Moved Permanently.
Код 300 означает, что сраница переехала
* ![Task1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task1.png)
2. Следуя инструкции из задания получил следующие ответы:
*  Полученный HTTP код - ![Task2-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task2-1.png)
*  Самый долгий запрос - ![Task2-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task2-2.png)
3. Используя адрес https://whoer.net/ru узнал свой ip адрес - 109.196.73.177
4. С помощью whois 109.196.73.177 проверил инормацию, ответы:
* Владелец IP - Полученный HTTP код - ![Task4-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task4-1.png)
* Автономная система - Полученный HTTP код - ![Task4-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task4-2.png)
5. Используя команду traceroute -A -I 8.8.8.8, сделал трасировку и получил результат
* ![Task5](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task5.png)
6. Используя команду mtr -z 8.8.8.8, получил результат
* ![Task6](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task6.png)
7. Используя команду dig +trace dns.google получил результат
* Сервера - ![Task7-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task7-1.png)
* А записи - ![Task7-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/Lecture1/CS1-task7-2.png)
