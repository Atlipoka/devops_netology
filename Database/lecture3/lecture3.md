1. Распишу по пунктам:
  * Развернул контейнер с MySQL ver.8 
 ```
 docker run --name testdb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 \
 -v "/home/data/":/home/vagrant/testdb/data/ -v "/home/backup/":/home/vagrant/testdb/backup/ -d mysql:8.
 ```
  * Подключился к контейнеру и внутри контенера к mysql ``mysql -u root -p``. Создал БД - CREATE DATABASE test_db; и после восстановился из бэкапа - ``mysql -u root -p test_db < /home/vagrant/testdb/backup/test_dump.sql``
  * Используя команду ``\s`` получил информацию о версии БД ``Server version: 8.0.31 MySQL Community Server - GPL``
  * Подключился к БД test_db ``use test_db`` и посмотрел список всех БД командой - ``SHOW TABLES;``, результат выполнения:
    * ![task1-1](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture3/task1-1.png)
  * Ввел запрос ``select * from orders where price>300;`` и получил ответ:
    * ![task1-2](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture3/task1-2.png)
 2. Распишу по пунктам:
  * Создал пользователя командой - 
  ```
  CREATE USER 'test'@'localhost'``
  IDENTIFIED WITH mysql_native_password BY 'test-pass'
  WITH MAX_CONNECTIONS_PER_HOUR 100``
  PASSWORD EXPIRE INTERVAL 180 DAY``
  FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME UNBOUNDED``
  ATTRIBUTE '{"FName":"James", "LName":"Pretty"}';
  ```
