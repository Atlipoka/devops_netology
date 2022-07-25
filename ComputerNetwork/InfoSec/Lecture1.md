1. Установил расширение Bitwarden для Chrome, зарегистрировался и сохранил креды.
 * ![Task-1](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task1.png)
2. Скачал приложение Google Auth и привязал к аккаунту в Bitwarden вдухэтапную аутентификацию
 * ![Task-2](https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task2.png)
3. Распишу по шагам (конфигурировал apache используюя инструкцию - https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04)
* apt install apache2
* Команда для генерации сертификати - openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crtsudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
* Настроил виртуальный хост, где прописал все настройки - (https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task3.png)
* Перезапустил apache2, пробросил порты на VB, на локальном ПК прописал в host ДНС и IP сайта, открыл его в браузере - (https://github.com/Atlipoka/devops_netology/blob/main/ComputerNetwork/InfoSec/IS-task3-2.png)
