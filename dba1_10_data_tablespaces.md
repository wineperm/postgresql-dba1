# Практика

---

## Задание

---

Почему при создании базы данных без предложения
TABLESPACE табличным пространством по умолчанию
становится pg_default?

1. Создайте новое табличное пространство.
2. Измените табличное пространство по умолчанию для базы
   данных template1 на созданное пространство.
3. Создайте новую базу данных.
   Проверьте, какое табличное пространство по умолчанию
   установлено для новой базы данных.
4. Посмотрите в файловой системе символьную ссылку
   в PGDATA на каталог табличного пространства.
5. Удалите созданное табличное пространство.

## Решение

## 1. Новое табличное пространство

```
sudo -u postgres mkdir /var/lib/postgresql/ts_dir
sudo -i -u postgres
psql
CREATE TABLESPACE ts LOCATION '/var/lib/postgresql/ts_dir';
```

![Alt text]()

## 2. Табличное пространство по умолчанию для template1

```
ALTER DATABASE template1 SET TABLESPACE ts;
```

![Alt text]()

## 3. Новая база данных и проверка

```
CREATE DATABASE db;
SELECT spcname FROM pg_tablespace WHERE oid = (SELECT dattablespace FROM pg_database WHERE datname = 'db');
```

![Alt text]()

## 4. Символическая ссылка

```
SELECT oid AS tsoid FROM pg_tablespace WHERE spcname = 'ts';
\q
exit
sudo -u postgres ls -l /var/lib/postgresql/17/main/pg_tblspc/24639
```

![Alt text]()

## 5. Удаление табличного пространства

```
sudo -i -u postgres
psql
ALTER DATABASE template1 SET TABLESPACE pg_default;
DROP DATABASE db;
DROP TABLESPACE ts;
\q
exit
sudo -u postgres rm -rf /var/lib/postgresql/ts_dir
```

![Alt text]()

# Практика +

---

## Задание

---

1. Установите параметр random_page_cost в значение 1.1
   для табличного пространства pg_default.

## Решение

## 1. Установка random_page_cost для табличного пространства

```
sudo -i -u postgres
psql
\dconfig *page_cost
ALTER TABLESPACE pg_default SET (random_page_cost = 1.1);
\db+
```

![Alt text]()
