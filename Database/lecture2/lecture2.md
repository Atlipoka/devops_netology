1. Команда для создания контейнера с 2 volume:
 * ``docker run --name testdb -p 5432:5432 -e POSTGRES_USER=maxim -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=testdb \
-e PGDATA=/home/vagrant/testdb/data/pgdata -e PG_BASEBACKUP=/home/vagrant/testdb/backup/pgbackup \
-v "/home/vagrant/testdb/data/pgdata":/var/lib/postgresql/data -v "/home/vagrant/testdb/backup/pgbackup":/var/lib/postgresql/backup \
-d postgres:12.12``
2. Задание вополнял частично в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
 * Итоговый список БД - ``testdb-# \l
                             List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | Access privileges
-----------+-------+----------+------------+------------+-------------------
 postgres  | maxim | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | maxim | UTF8     | en_US.utf8 | en_US.utf8 | =c/maxim         +
           |       |          |            |            | maxim=CTc/maxim
 template1 | maxim | UTF8     | en_US.utf8 | en_US.utf8 | =c/maxim         +
           |       |          |            |            | maxim=CTc/maxim
 testdb    | maxim | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)``
 * 
