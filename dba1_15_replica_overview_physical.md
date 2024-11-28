# Практика

---

## Задание

---

1. Настройте физическую потоковую репликацию
   между двумя серверами в синхронном режиме.
   Проверьте работу репликации. Убедитесь, что при
   остановленной реплике фиксация не завершается.
2. По умолчанию применение конфликтующих записей
   на реплике откладывается максимум на 30 секунд.
   Отключите откладывание применения и убедитесь, что
   долгий запрос, выполняющийся на реплике, будет прерван,
   если необходимые ему версии строк удаляются и очищаются
   на мастере.
   Включите обратную связь и убедитесь, что теперь запрос
   не прерывается из-за того, что на мастере откладывается
   очистка.

## Решение

## 1. Синхронная репликация

```
pg_basebackup --pgdata=/home/student/tmp/backup -R --checkpoint=fast
sudo pg_ctlcluster 16 replica stop
sudo rm -rf /var/lib/postgresql/16/replica
sudo mv /home/student/tmp/backup /var/lib/postgresql/16/replica
sudo chown -R postgres: /var/lib/postgresql/16/replica
sudo pg_ctlcluster 16 replica start
psql
SHOW synchronous_commit;
SHOW synchronous_standby_names;
|| psql -p 5433
|| SHOW cluster_name;
ALTER SYSTEM SET synchronous_standby_names = '"16/replica"';
SELECT pg_reload_conf();
SELECT sync_state FROM pg_stat_replication;
CREATE DATABASE replica_overview_physical;
\c replica_overview_physical
|| \q
|| sudo pg_ctlcluster 16 replica stop
CREATE TABLE test(n integer);
|| sudo pg_ctlcluster 16 replica start
```

![Alt text]()

## 2. Конфликтующие записи

```
psql -d replica_overview_physical
ALTER SYSTEM SET max_standby_streaming_delay = 0;
SELECT pg_reload_conf();
INSERT INTO test(n) SELECT id FROM generate_series(1,10) AS id;
|| psql -p 5433 -d replica_overview_physical
|| SELECT pg_sleep(5), count(*) FROM test;
DELETE FROM test;
VACUUM VERBOSE test;
-----
ALTER SYSTEM SET hot_standby_feedback = on;
SELECT pg_reload_conf();
INSERT INTO test(n) SELECT id FROM generate_series(1,10) AS id;
|| SELECT pg_sleep(5), count(*) FROM test;
DELETE FROM test;
VACUUM VERBOSE test;
ALTER SYSTEM RESET synchronous_standby_names;
SELECT pg_reload_conf();
```

![Alt text]()
