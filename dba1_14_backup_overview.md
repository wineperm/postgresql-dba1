# Практика

---

## Задание

---

1. Создайте базу данных и таблицу в ней с несколькими
   строками.
2. Сделайте логическую копию базы данных с помощью
   утилиты pg_dump.
   Удалите базу данных и восстановите ее из сделанной копии.
3. Сделайте автономную физическую резервную копию
   кластера с помощью утилиты pg_basebackup.
   Измените таблицу.
   Восстановите новый кластер из сделанной резервной копии
   и проверьте, что база данных не содержит более поздних
   изменений.

## Решение

## 1. База данных и таблица

```
psql
CREATE DATABASE backup_overview;
\c backup_overview
CREATE TABLE t(n integer);
INSERT INTO t VALUES (1), (2), (3);
```

![Alt text]()

## 2. Логическая резервная копия

```
\q
pg_dump -f /home/student/tmp/backup_overview.dump -d backup_overview --create
psql
\c postgres
DROP DATABASE backup_overview;
\q
psql -f /home/student/tmp/backup_overview.dump
psql
\c backup_overview
SELECT * FROM t;
```

![Alt text]()

## 3. Физическая автономная резервная копия

```
rm -rf /home/student/tmp/backup
pg_basebackup --pgdata=/home/student/tmp/backup --checkpoint=fast
sudo pg_ctlcluster 16 replica status
sudo rm -rf /var/lib/postgresql/16/replica
sudo mv /home/student/tmp/backup /var/lib/postgresql/16/replica
sudo chown -R postgres:postgres /var/lib/postgresql/16/replica
|| DELETE FROM t;
sudo pg_ctlcluster 16 replica start
psql -p 5433 -d backup_overview
SELECT * FROM t;
```

![Alt text]()

# Практика +

---

## Задание

---

1. Организуйте потоковую архивацию кластера main
   с помощью утилиты pg_receivewal.
2. Сделайте автономную резервную копию кластера main
   (без WAL) с помощью утилиты pg_basebackup.
3. В кластере main создайте базу данных и таблицу в ней.
4. Выполните восстановление кластера replica из базовой копии
   с использованием архива. Убедитесь, что база и таблица
   тоже восстановились.

## Решение

## 1. Потоковый архив

```
sudo -i -u postgres
mkdir /var/lib/postgresql/archive
pg_receivewal --create-slot --slot=archive
pg_receivewal -D /var/lib/postgresql/archive --slot=archive
|| sudo ls -l /var/lib/postgresql/archive
```

![Alt text]()

## 2. Базовая физическая копия без журнала

```
|| pg_basebackup --wal-method=none --pgdata=/home/student/tmp/backup --checkpoint=fast
|| ls -l /home/student/tmp/backup
```

![Alt text]()

## 3. Новые база данных и таблица

```
|| psql
|| CREATE DATABASE backup_overview;
|| \c backup_overview
|| CREATE TABLE t(n integer);
|| INSERT INTO t VALUES (1), (2), (3);
```

![Alt text]()

## 4. Настройка восстановления

```
|| \q
|| sudo pg_ctlcluster 16 replica status
|| sudo rm -rf /var/lib/postgresql/16/replica
|| sudo mv /home/student/tmp/backup /var/lib/postgresql/16/replica
|| echo "restore_command = 'cp /var/lib/postgresql/archive/%f %p || cp /var/lib/postgresql/archive/%f.partial %p'" | sudo tee /var/lib/postgresql/16/replica/postgresql.auto.conf
|| touch /var/lib/postgresql/16/replica/recovery.signal
|| sudo chown -R postgres:postgres /var/lib/postgresql/16/replica
|| sudo pg_ctlcluster 16 replica start
|| psql -p 5433 -d backup_overview
|| SELECT * FROM t;
|| sudo pkill pg_receivewal
pg_receivewal --drop-slot --slot=archive

```

![Alt text]()
