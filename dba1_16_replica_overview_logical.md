# Практика

---

## Задание

---

1. Настройте логическую репликацию произвольной таблицы
   в другую на том же самом сервере.
2. Настройте двунаправленную логическую репликацию одной
   и той же таблицы на двух разных серверах.

## Решение

## 1. Репликация на одном сервере

```
psql
CREATE DATABASE replica_overview_logical_dba;
\c replica_overview_logical_dba
CREATE TABLE test (id int);
\c student
CREATE DATABASE replica_overview_logical_dba2 TEMPLATE replica_overview_logical_dba;
ALTER SYSTEM SET wal_level = logical;
\q
sudo pg_ctlcluster 16 main restart
psql replica_overview_logical_dba
|| psql replica_overview_logical_dba2
CREATE PUBLICATION test FOR TABLE test;
SELECT pg_create_logical_replication_slot('test_slot','pgoutput');
|| CREATE SUBSCRIPTION test CONNECTION 'user=student dbname=replica_overview_logical_dba' PUBLICATION test WITH (slot_name = test_slot, create_slot = false);
INSERT INTO test SELECT * FROM generate_series(1,100);
|| SELECT count(*) FROM test;
DROP PUBLICATION test;
|| DROP SUBSCRIPTION test;
DROP DATABASE replica_overview_logical_dba2 (FORCE);
```

![Alt text]()

## 2. Двунаправленная репликация

```
pg_basebackup --pgdata=/home/student/tmp/backup --checkpoint=fast
sudo pg_ctlcluster 16 replica stop
sudo rm -rf /var/lib/postgresql/16/replica
sudo mv /home/student/tmp/backup /var/lib/postgresql/16/replica
sudo chown -R postgres: /var/lib/postgresql/16/replica
sudo pg_ctlcluster 16 replica start
psql -p 5432 replica_overview_logical_dba
|| psql -p 5433 replica_overview_logical_dba
|| SHOW wal_level;
|| SELECT count(*) FROM test;
CREATE PUBLICATION test FOR TABLE test;
|| CREATE PUBLICATION test FOR TABLE test;
CREATE SUBSCRIPTION test CONNECTION 'port=5433 user=student dbname=replica_overview_logical_dba' PUBLICATION test WITH (copy_data = false, origin = none);
|| CREATE SUBSCRIPTION test CONNECTION 'port=5432 user=student dbname=replica_overview_logical_dba' PUBLICATION test WITH (copy_data = false, origin = none);
UPDATE test SET id = id + 100;
ALTER TABLE test ADD PRIMARY KEY (id);
|| ALTER TABLE test ADD PRIMARY KEY (id);
UPDATE test SET id = id + 100;
|| UPDATE test SET id = id + 100;
SELECT min(id), max(id) FROM test;
|| SELECT min(id), max(id) FROM test;
```

![Alt text]()
