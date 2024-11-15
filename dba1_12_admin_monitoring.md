# Практика

---

## Задание

---

1. В новой базе данных создайте таблицу, выполните вставку
   нескольких строк, а затем удалите все строки.
   Посмотрите статистику обращений к таблице и сопоставьте
   цифры (n_tup_ins, n_tup_del, n_live_tup, n_dead_tup)
   с вашей активностью.
   Выполните очистку (vacuum), снова проверьте статистику
   и сравните с предыдущими цифрами.
2. Создайте ситуацию взаимоблокировки двух транзакций.
   Посмотрите, какая информация записывается при этом
   в журнал сообщений сервера.

## Решение

## 1. Статистика обращений к таблице

```
sudo -i -u postgres
psql
CREATE DATABASE admin_monitoring;
\c admin_monitoring
CREATE TABLE t(n numeric);
INSERT INTO t SELECT 1 FROM generate_series(1,1000);
DELETE FROM t;
SELECT * FROM pg_stat_all_tables WHERE relid = 't'::regclass \gx
VACUUM;
SELECT * FROM pg_stat_all_tables WHERE relid = 't'::regclass \gx
```

![Alt text]()

## 2. Взаимоблокировка

```
INSERT INTO t VALUES (1),(2);
| sudo -i -u postgres
| psql
| \c admin_monitoring
| BEGIN;
| UPDATE t SET n = 10 WHERE n = 1;
|| sudo -i -u postgres
|| psql
|| \c admin_monitoring
|| BEGIN;
|| UPDATE t SET n = 200 WHERE n = 2;
| UPDATE t SET n = 20 WHERE n = 2;
|| UPDATE t SET n = 100 WHERE n = 1;
\q
exit
sudo tail -n 8 /var/log/postgresql/postgresql-17-main.log
```

![Alt text]()

# Практика +

---

## Задание

---

1. Установите расширение pg_stat_statements.
   Выполните несколько произвольных запросов.
   Посмотрите, какую информацию показывает представление
   pg_stat_statements.

## Решение

## 1. Расширение pg_stat_statements

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';
\q
sudo pg_ctlcluster 17 main restart
psql
CREATE DATABASE admin_monitoring;
\c admin_monitoring
CREATE EXTENSION pg_stat_statements;
CREATE TABLE t(n numeric);
SELECT format('INSERT INTO t VALUES (%L)', x) FROM generate_series(1,5) AS x \gexec
DELETE FROM t;
DROP TABLE t;
SELECT query, calls, total_exec_time FROM pg_stat_statements ORDER BY calls DESC LIMIT 1;
ALTER SYSTEM RESET shared_preload_libraries;
\q
sudo pg_ctlcluster 16 main restart
```

![Alt text]()
