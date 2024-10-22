# Практика

---

## Задание

1. Создайте таблицу с одной строкой.
   Начните первую транзакцию на уровне изоляции Read
   Committed и выполните запрос к таблице.
   Во втором сеансе удалите строку и зафиксируйте изменения.
   Сколько строк увидит первая транзакция, выполнив тот же
   запрос повторно? Проверьте.
   Завершите первую транзакцию.
2. Повторите то же самое, но пусть теперь транзакция работает
   на уровне изоляции Repeatable Read:
   BEGIN ISOLATION LEVEL REPEATABLE READ;
   Объясните отличия.

---

## Решение

## 1. Уровень изоляции Read Committed

```
sudo -i -u postgres
psql
CREATE TABLE t(n integer);
INSERT INTO t VALUES (42);
BEGIN;
SELECT * FROM t;
| sudo -i -u postgres
| psql
| DELETE FROM t;
SELECT * FROM t;
COMMIT;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_05_arch_mvcc_overview/read_committed.jpg)

## 2. Уровень изоляции Repeatable Read

```
sudo -i -u postgres
psql
INSERT INTO t VALUES (42);
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM t;
| sudo -i -u postgres
| psql
| DELETE FROM t;
SELECT * FROM t;
COMMIT;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_05_arch_mvcc_overview/repeatable_read.jpg)

# Практика +

---

## Задание

1.  Начните транзакцию и создайте новую таблицу с одной
    строкой. Не завершая транзакцию, откройте второй сеанс
    и выполните в нем запрос к таблице. Проверьте, что увидит
    транзакция во втором сеансе.
    Зафиксируйте транзакцию в первом сеансе и повторите
    запрос к таблице во втором сеансе.
2.  Повторите задание 1, но откатите, а не зафиксируйте
    транзакцию в первом сеансе. Что изменилось?
3.  В первом сеансе начните транзакцию и выполните запрос
    к созданной ранее таблице. Получится ли удалить эту
    таблицу во втором сеансе, пока первая транзакция не
    завершена? Проверьте.

---

## Решение

## 1. Транзакции и команды DDL — фиксация

```
BEGIN;
CREATE TABLE t1(n integer);
INSERT INTO t1 VALUES (42);
| SELECT * FROM t1;
COMMIT;
| SELECT * FROM t1;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_05_arch_mvcc_overview/DDL_fixing.jpg)

## 2. Транзакции и команды DDL — откат

```
BEGIN;
CREATE TABLE t2(n integer);
INSERT INTO t2 VALUES (42);
| SELECT * FROM t2;
ROLLBACK;
SELECT * FROM t2;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_05_arch_mvcc_overview/DDL_rollback.jpg)

## 3. Блокировки таблиц

```
BEGIN;
SELECT * FROM t1;
| DROP TABLE t1;
COMMIT;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_05_arch_mvcc_overview/table_locks.gif)
