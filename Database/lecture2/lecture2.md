1. Команда для создания контейнера с 2 volume:
  * ``docker run --name testdb-1 -p 5432:5432 -e POSTGRES_USER=maxim -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=testdb -v "/home/data/":/home/vagrant/testdb/data/ -v "/home/backup/":/home/vagrant/testdb/backup/ -d postgres:12.12``
2. Задание выполнял частично в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
  * Итоговый список БД
    * ![task2-1](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task2-1.png)
  * Описание таблиц
    * ![task2-2](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task2-2.png)
  * Запрос на получение списка пользователей с правами - ``SELECT grantee, table_catalog, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');``
  * Полученный список
    * ![task2-4](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task2-4.png)
3. Задание выполнял в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
  * Скрипты для заполнения таблиц и получения кол-ва записей из таблиц
    * ![task3-1](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task3-1.png)
    * При выполнении подсчета результат по 5 строк в каждой таблице.
4. Задание выполнял в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
  * Для обновления записей использовал запросы:
    * ``UPDATE orders SET наименование='Книга' WHERE id=(select id from clients where фамилия = 'Иванов Иван Иванович');``
    * ``UPDATE orders SET наименование='Монитор' WHERE id=(select id from clients where фамилия = 'Петров Петр Петрович');``
    * ``UPDATE orders SET наименование='Гитара' WHERE id=(select id from clients where фамилия = 'Иоганн Себастьян Бах');``
  * Для получения списка записей использовал скрипт:
    * ``select c.фамилия,o.наименование from clients c
        join orders o on c.заказ=o.id
        where o.наименование is not null``
   * Результат выполнения скпира выше - ![Task4](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task4.png)
5. Задание выполнял в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
  * Результат выполнения запроса с использованием explain. Explain анализирует результат выполнения запроса, указывает, как запрос обрабатывается СУБД и предлагает варианты оптимизации запроса.
   * ![Task5](https://github.com/Atlipoka/devops_netology/blob/main/Database/lecture2/task5.png)
6. Распишу по пордяку:
  * Сохраняем бэкап - ``pg_dump -U maxim testdb > /home/backup/pgbackup/testdb.backup``
  * Стопим контейнер - ``docker stop testdb``
  * Поднимаем новый контейнер - ``docker run --name testdb-1 -p 5432:5432 -e POSTGRES_USER=maxim -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=testdb -v "/home/data/":/home/vagrant/testdb-1/data/ -v "/home/backup/":/home/vagrant/testdb-1/backup/ -d postgres:12.12``
  * Копируем бэкап - ``docker cp testdb:/home/vagrant/testdb/backup/pgbackup/testdb.backup /home/vagrant/testdb/backup/pgbackup && docker cp /home/vagrant/testdb/backup/pgbackup testdb-1:/home/vagrant/testdb-1/backup/``
  * Поднимаем БД из бэкапа - ``psql -U maxim -d testdb -f /home/vagrant/testdb-1/backup/pgbackup``
