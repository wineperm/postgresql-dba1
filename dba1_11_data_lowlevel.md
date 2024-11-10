# Практика

---

## Задание

---

1. Создайте нежурналируемую таблицу в пользовательском
   табличном пространстве и убедитесь, что для таблицы
   существует слой init.
   Удалите созданное табличное пространство.
2. Создайте таблицу со столбцом типа text.
   Какая стратегия хранения применяется для этого столбца?
   Измените стратегию на external и вставьте в таблицу
   короткую и длинную строки.
   Проверьте, попали ли строки в toast-таблицу, выполнив
   прямой запрос к ней. Объясните, почему.

## Решение

## 1. Нежурналируемая таблица

```
sudo -u postgres mkdir /var/lib/postgresql/ts_dir
sudo -i -u postgres
CREATE TABLESPACE ts LOCATION '/var/lib/postgresql/ts_dir';
CREATE DATABASE data_lowlevel;
\c data_lowlevel
CREATE UNLOGGED TABLE u(n integer) TABLESPACE ts;
INSERT INTO u(n) SELECT n FROM generate_series(1,1000) n;
SELECT pg_relation_filepath('u');
\q
ls -l /var/lib/postgresql/17/main/pg_tblspc/24641/PG_17_202406281/24642/24643*
psql
\c data_lowlevel
DROP TABLE u;
DROP TABLESPACE ts;
\q
exit
sudo -u postgres rm -rf /var/lib/postgresql/ts_dir
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_11_data_lowlevel/Non-journalable_table.jpg)

## 2. Таблица с текстовым столбцом

```
CREATE TABLE t(s text);
\d+ t
ALTER TABLE t ALTER COLUMN s SET STORAGE external;
INSERT INTO t(s) VALUES ('Короткая строка.');
INSERT INTO t(s) VALUES (repeat('A',3456));
SELECT relname FROM pg_class WHERE oid = (SELECT reltoastrelid FROM pg_class WHERE relname='t');
SELECT chunk_id, chunk_seq, length(chunk_data) FROM pg_toast.pg_toast_24652
ORDER BY chunk_id, chunk_seq;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_11_data_lowlevel/A_table_with_a_text_column.jpg)

Объяснение
Короткая строка: Если строка короткая (менее 2000 байт), она будет храниться в основной таблице, даже если стратегия хранения установлена на external.
Длинная строка: Если строка длинная (более 2000 байт), она будет храниться в toast-таблице.
Таким образом, короткая строка не попадет в toast-таблицу, а длинная строка попадет.

# Практика +

---

## Задание

---

1. Создайте базу данных.
   Сравните размер базы данных, возвращаемый функцией
   pg_database_size, с общим размеров всех таблиц в этой базе.
   Объясните полученный результат.
2. В TOAST поддерживаются два метода сжатия: pglz и lz4.
   Проверьте средствами SQL, был ли PostgreSQL
   скомпилирован с поддержкой этих методов.
3. Создайте текстовый файл размером больше 10 Мбайт.
   Загрузите его содержимое в таблицу с текстовым полем,
   сначала без сжатия, а затем используя каждый из
   алгоритмов. Сравните итоговый размер таблицы и время
   загрузки данных для трех вариантов.

## Решение

## 1. Сравнение размеров базы данных и таблиц в ней

```
CREATE DATABASE data_lowlevel;
\c data_lowlevel
SELECT sum(pg_total_relation_size(oid)) FROM pg_class WHERE NOT relisshared AND relkind = 'r';
SELECT pg_database_size('data_lowlevel');
SELECT oid FROM pg_database WHERE datname = 'data_lowlevel';
\q
ls -l /var/lib/postgresql/17/main/base/24658/[^0-9]*
```

WHERE NOT relisshared -- локальные объекты базы
AND relkind = 'r'; -- обычные таблицы

pg_filenode.map — отображение oid некоторых таблиц в имена файлов;
pg_internal.init — кеш системного каталога;
PG_VERSION — версия PostgreSQL

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_11_data_lowlevel/Comparing_the_size_of_the_database_and_the_tables_in_it.jpg)

## 2. Поддержка методов сжатия TOAST

```
SELECT * FROM (SELECT string_to_table(setting, ''' ''') AS setting FROM pg_config WHERE name = 'CONFIGURE') WHERE setting ~ '(lz|zs)';
\dconfig *toast*
SELECT setting, enumvals FROM pg_settings WHERE name = 'default_toast_compression';
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_11_data_lowlevel/Support_for_TOAST_compression_methods.jpg)

## 3. Сравнение методов сжатия

```
sudo cat /usr/lib/postgresql/17/bin/postgres | base32 -w0 > /tmp/gram.input
ls -l --block-size=K /tmp/gram.input
sudo -i -u postgres
psql
CREATE TABLE t (txt text STORAGE EXTERNAL);
\timing on
COPY t FROM '/tmp/gram.input';
\timing off
SELECT pg_table_size('t')/1024;
TRUNCATE TABLE t;
ALTER TABLE t ALTER COLUMN txt SET STORAGE EXTENDED, ALTER COLUMN txt SET COMPRESSION pglz;
\timing on
COPY t FROM '/tmp/gram.input';
\timing off
SELECT pg_table_size('t')/1024;
TRUNCATE TABLE t;
ALTER TABLE t ALTER COLUMN txt SET COMPRESSION lz4;
\timing on
COPY t FROM '/tmp/gram.input';
\timing off
SELECT pg_table_size('t')/1024;
\q
exit
sudo rm -f /tmp/gram.input
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_11_data_lowlevel/Comparison_of_compression_methods.jpg)
