# Практика

---

## Задание

---

1. Создайте новую базу данных и подключитесь к ней.
2. Проверьте размер созданной базы данных.
3. Создайте две схемы: app и названную так же, как
   пользователь.
   Создайте несколько таблиц в обеих схемах и наполните их
   какими-нибудь данными.
4. Проверьте, на сколько увеличился размер базы данных.
5. Установите путь поиска так, чтобы при подключении к БД
   таблицы из обеих схем были доступны по
   неквалифицированному имени; приоритет должна иметь
   «пользовательская» схема.

## Решение

## 1. База данных

```
sudo -i -u postgres
psql
CREATE DATABASE data_databases;
\c data_databases
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Database.jpg)

## 2. Размер БД

```
SELECT pg_size_pretty(pg_database_size('data_databases'));
SELECT pg_database_size('data_databases') AS oldsize \gset
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Database_size.jpg)

## 3. Схемы и таблицы

```
CREATE SCHEMA app;
CREATE SCHEMA student;
SELECT current_schema();
CREATE TABLE a(s text);
INSERT INTO a VALUES ('student');
CREATE TABLE b(s text);
INSERT INTO b VALUES ('student');

CREATE TABLE app.a(s text);
INSERT INTO app.a VALUES ('app');
CREATE TABLE app.c(s text);
INSERT INTO app.c VALUES ('app');
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Diagrams_and_tables.jpg)

## 4. Изменение размера БД

```
SELECT pg_size_pretty(pg_database_size('data_databases'));
SELECT pg_database_size('data_databases') AS newsize \gset
SELECT pg_size_pretty(:newsize::bigint - :oldsize::bigint);
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Changing_the_database_size.jpg)

## 5. Путь поиска

```
SELECT * FROM a;
SELECT * FROM b;
SELECT * FROM c;
ALTER DATABASE data_databases SET search_path = "$user",app,public;
\c
SHOW search_path;
SELECT * FROM a;
SELECT * FROM b;
SELECT * FROM c;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Search_path.jpg)

# Практика +

---

## Задание

---

1. Создайте базу данных. Для всех сеансов этой базы данных
   установите значение параметра temp_buffers, в четыре раза
   превышающее значение по умолчанию.

## Решение

## 1. Установка temp_buffers

```
CREATE DATABASE data_databases;
\c data_databases
SELECT name, setting, unit, boot_val, reset_val FROM pg_settings WHERE name = 'temp_buffers' \gx
ALTER DATABASE data_databases SET temp_buffers = '32MB';
\c
SHOW temp_buffers;
\drds
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_08_data_databases/Installing_temp_buffers.jpg)
