1. Команда для создания контейнера с 2 volume:
 * ``docker run --name testdb -p 5432:5432 -e POSTGRES_USER=maxim -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=testdb \
-e PGDATA=/home/vagrant/testdb/data/pgdata -e PG_BASEBACKUP=/home/vagrant/testdb/backup/pgbackup \
-v "/home/vagrant/testdb/data/pgdata":/var/lib/postgresql/data -v "/home/vagrant/testdb/backup/pgbackup":/var/lib/postgresql/backup \
-d postgres:12.12``
