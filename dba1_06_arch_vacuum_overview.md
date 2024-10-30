# Практика

---

## Задание

1. Отключите процесс автоочистки и убедитесь, что он
   не работает.
2. В новой базе данных создайте таблицу с одним числовым
   столбцом и индекс по этой таблице. Вставьте в таблицу
   100 000 случайных чисел.
3. Несколько раз измените половину строк таблицы,
   контролируя на каждом шаге размер таблицы и индекса.
4. Выполните полную очистку.
5. Повторите пункт 4, вызывая после каждого изменения
   обычную очистку. Сравните результаты.
6. Включите процесс автоочистки.

---

## Решение

## 1. Отключение автоочистки

```
sudo -i -u postgres
psql
SELECT pid, backend_start, backend_type FROM pg_stat_activity WHERE backend_type = 'autovacuum launcher';
ALTER SYSTEM SET autovacuum = off;
SELECT pg_reload_conf();
SELECT pid, backend_start, backend_type FROM pg_stat_activity WHERE backend_type = 'autovacuum launcher';
```

![Alt text]()

## 2. База данных, таблица и индекс

```
CREATE DATABASE arch_vacuum_overview;
\c arch_vacuum_overview
CREATE TABLE t(n numeric);
CREATE INDEX t_n on t(n);
INSERT INTO t SELECT random() FROM generate_series(1,100_000);
```

![Alt text]()

## 3. Изменение строк без очистки

```
\set SIZE 'SELECT pg_size_pretty(pg_table_size(''t'')) table_size, pg_size_pretty(pg_indexes_size(''t'')) index_size\\g (footer=off)'
:SIZE
UPDATE t SET n=n WHERE n < 0.5;
:SIZE
UPDATE t SET n=n WHERE n < 0.5;
:SIZE
UPDATE t SET n=n WHERE n < 0.5;
:SIZE
```

![Alt text]()

## 4. Полная очистка

```
VACUUM FULL t;
:SIZE
```

![Alt text]()

## 5. Изменение строк с очисткой

```
UPDATE t SET n=n WHERE n < 0.5;
VACUUM t;
:SIZE
UPDATE t SET n=n WHERE n < 0.5;
VACUUM t;
:SIZE
UPDATE t SET n=n WHERE n < 0.5;
VACUUM t;
:SIZE
```

![Alt text]()

## 6. Восстанавливаем автоочистку

```
ALTER SYSTEM RESET autovacuum;
SELECT pg_reload_conf();
```

![Alt text]()
