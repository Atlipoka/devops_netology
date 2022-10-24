1. Поднял контейнер с 13 версией postgres, зашел на контейнер, внутри контейнера подключился к БД и ввел команду \?, далее расписываю по шагам:
  * вывода списка БД - ``\l``
  * подключения к БД - ``\deu``
  * вывода списка таблиц - ``\dtS``
  * вывода описания содержимого таблиц - ``\d``
  * выхода из psql - ``\q``
2. Распишу по пунктам:
  * Создал БД ``create database test_database;``
  * Посмотел бэекап и создал юзера postgres ``create user postgres;``
  * Восстановился из бэкапа ``psql -U maxim -d test_database -f /home/vagrant/testdb/backup/test_dump.sql``
  * Зашел в контейнер и подключился к БД test_database ``psql -U maxim -d test_database``
  * Использовал ANALYZE на таблицу orders ``analyze orders;``
  * Вычислил столбец с максимальным значением еспользуя pg_stats - ``select avg_width,attname from pg_stats where tablename='orders' order by avg_width desc limit 1;``, результат - ![task2](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture4/task2.png)
3. Результаты работы по шардированию таблицы orders:
  * ```--создаем первую БД orders_1 с условием price>499
create table orders_1 (like orders);
insert into orders_1 select * from orders where price>499;

--создаем вторую БД orders_2 с условием price<=499
create table orders_2 (like orders);
insert into orders_2 select * from orders where price<=499;

--Удаляем данные из orders по условиям price>499 и price<=499
delete from orders where price>499;
delete from orders where price<=499;```
