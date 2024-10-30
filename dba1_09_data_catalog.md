# Практика

---

## Задание

---

1. Получите описание таблицы pg_class.
2. Получите подробное описание представления pg_tables.
3. Создайте базу данных и временную таблицу в ней.
   Получите полный список схем в базе, включая системные.
4. Получите список представлений в схеме information_schema.
5. Какие запросы выполняет следующая команда psql?\d+ pg_views

## Решение

## 1. Описание pg_class

```
sudo -i -u postgres
psql
\d pg_class
```

![Alt text]()

## 2. Подробное описание pg_tables

```
\d+ pg_tables
```

![Alt text]()

## 3. Полный список схем

```
CREATE DATABASE data_catalog;
\c data_catalog
CREATE TEMP TABLE t(n integer);
\dnS
SELECT pg_my_temp_schema()::regnamespace;
SELECT * FROM pg_temp.t;
```

![Alt text]()

## 4. Список представлений в information_schema

```
\dv information_schema.*
```

![Alt text]()

## 5. Запросы к системному каталогу

```
\set ECHO_HIDDEN on
\d+ pg_views
\set ECHO_HIDDEN off
```

![Alt text]()
