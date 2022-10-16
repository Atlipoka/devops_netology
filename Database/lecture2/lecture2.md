1. Команда для создания контейнера с 2 volume:
 * ``docker run --name testdb -p 5432:5432 -e POSTGRES_USER=maxim -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=testdb \
-e PGDATA=/home/vagrant/testdb/data/pgdata -e PG_BASEBACKUP=/home/vagrant/testdb/backup/pgbackup \
-v "/home/vagrant/testdb/data/pgdata":/var/lib/postgresql/data -v "/home/vagrant/testdb/backup/pgbackup":/var/lib/postgresql/backup \
-d postgres:12.12``
2. Задание вополнял частично в DBeaver, поэтому иногда могу присылать скриншоту не из консоли. Теперь по пунктам:
 * Итоговый список БД - ![task2-1]()
 * Описание таблиц - 
 * Запрос на получение списка пользователей с правами - ``SELECT grantee, table_catalog, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');``
 * Полученный список - 
