# Практика

---

## Задание

1. Запустите psql и проверьте информацию о текущем
   подключении.
2. Выведите список баз данных в подробном виде.
3. По умолчанию psql использует команду «less» для
   постраничного просмотра результатов запроса. Замените ее
   на команду «less -XS» и снова выведите подробный список
   баз данных.
4. По умолчанию приглашение psql показывает имя базы
   данных. Настройте приглашение так, чтобы дополнительно
   выводилась информация о пользователе: роль@база=#
5. Настройте psql так, чтобы для всех команд выводилась
   длительность выполнения. Убедитесь, что при повторном
   запуске эта настройка сохраняется.

---

## Решение

## 1. Запуск psql и просмотр информации о подключении

```
sudo -i -u postgres
psql
\conninfo
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/launch.jpg)

## 2. Вывод список баз данных в подробном виде

```
\l+
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/database_information.jpg)

## 3. Замена на команду «less -XS»

```
echo "\setenv PSQL_PAGER 'less -XS'" > ~/.psqlrc
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/less-XS.jpg)

## 4. Настройка приглашения

```
echo "\set PROMPT1 '%n@%/%R%x%# '" >> ~/.psqlrc
echo "\set PROMPT2 '%n@%/%R%x%# '" >> ~/.psqlrc
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/invite.jpg)

## 5. Вывод длительности выполнения команд SQL

```
echo "\timing on" >> ~/.psqlrc
```

```
cat ~/.psqlrc
```

```
psql
SELECT * FROM your_table;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/timing.jpg)

# Практика +

---

## Задание

1. Откройте транзакцию и выполните команду, которая
   завершается любой ошибкой. Убедитесь, что продолжить
   работу в этой транзакции невозможно.
2. Задайте переменной ON_ERROR_ROLLBACK значение on
   и убедитесь, что после ошибки можно выполнять команды
   внутри транзакции.

---

## Решение

## 1. Утилита psql и обработка ошибок внутри транзакций

```
sudo -i -u postgres
psql
BEGIN;
CREATE TABLE t (id int);
INSERTINTO t VALUES(1);
INSERT INTO t VALUES(1);
COMMIT;
SELECT * FROM t;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/error_handling.jpg)

## 2. Переменная ON_ERROR_ROLLBACK

```
sudo -i -u postgres
psql
\set ON_ERROR_ROLLBACK on
BEGIN;
CREATE TABLE t (id int);
INSERTINTO t VALUES(1);
INSERT INTO t VALUES(1);
COMMIT;
SELECT * FROM t;
DROP TABLE t;
SELECT * FROM t;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_02_tools_psql/on_error_rollback.jpg)
