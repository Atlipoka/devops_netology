1.Распишу по пунктам:
  * Развернул контейнер с MySQL ver.8 - ``docker run --name testdb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v "/home/data/":/home/vagrant/testdb/data/ -v "/home/backup/":/home/vagrant/testdb/backup/ -d mysql:8.``
  * Подключился к контейнеру и внутри контенера к mysql ``mysql -u root -p``. Создал БД - CREATE DATABASE test_db; и после восстановился из бэкапа - ``mysql -u root -p test_db < /home/vagrant/testdb/backup/test_dump.sql``
  * Используя команду ``\s`` получил информацию о версии БД ``Server version: 8.0.31 MySQL Community Server - GPL``
  * Подключился к БД test_db ``use test_db`` и посмотрел список всех БД командой - ``SHOW TABLES;``, результат выполнения: 
   * ``mysql> SHOW tables;
   * +-------------------+
   * | Tables_in_test_db |
   * +-------------------+
   * | orders            |
   * +-------------------+
   * 1 row in set (0.01 sec)``
