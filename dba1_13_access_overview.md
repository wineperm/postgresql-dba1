# Практика

---

## Задание

---

Настройте привилегии таким образом, чтобы одни пользователи
имели полный доступ к таблицам, а другие могли только
запрашивать, но не изменять информацию.

1. Создайте новую базу данных и двух пользователей: writer и reader.
2. Отзовите у роли public все привилегии на схему public, выдайте роли
   writer обе привилегии, а роли reader — только usage.
3. Настройте привилегии по умолчанию так, чтобы роль reader получала
   доступ на чтение к таблицам, принадлежащим writer в схеме public.
4. Создайте пользователя w1, включив его в роль writer, и пользователя
   r1, включив его в reader.
5. Под ролью writer создайте таблицу.
6. Убедитесь, что r1 имеет доступ к таблице только на чтение, а w1
   имеет к ней полный доступ, включая удаление.

## Решение

## 1. База данных и роли

```
psql
CREATE DATABASE access_overview;
CREATE USER writer;
CREATE USER reader;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Database_and_roles.jpg)

## 2. Привилегии

```
\c access_overview
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO writer;
GRANT USAGE ON SCHEMA public TO reader;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Privileges.jpg)

## 3. Привилегии по умолчанию

```
ALTER DEFAULT PRIVILEGES FOR ROLE writer IN SCHEMA public GRANT SELECT ON TABLES TO reader;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Default_privileges.jpg)

## 4. Пользователи

```
CREATE ROLE w1 LOGIN IN ROLE writer;
CREATE ROLE r1 LOGIN IN ROLE reader;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Users.jpg)

## 5. Таблица

```
\c - writer
CREATE TABLE t(n integer);
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Table.jpg)

## 6. Проверка

```
\c - w1
INSERT INTO t VALUES (42);
\c - r1
SELECT * FROM t;
UPDATE t SET n = n + 1;
\c - w1
DROP TABLE t;
\c postgres postgres
DROP DATABASE access_overview;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Check.jpg)

# Практика +

---

## Задание

---

1. Зарегистрируйте пользовательские роли alice и bob.
2. Отредактируйте файл pg_hba.conf так, чтобы беспарольный
   вход был разрешен лишь для пользователей postgres и
   student. Убедитесь, что доступ для alice и bob запрещен.
3. Для пользователей alice и bob включите разрешение входить
   в сеанс, используя метод peer. Проверьте невозможность
   входа в сеанс без создания отображения на пользователя ОС.
   Для alice создайте такое отображение.
4. Проверьте возможность использовать одно и то же
   отображение для разных ролей.

## Решение

## 1. Добавление ролей

```
CREATE ROLE alice LOGIN;
CREATE ROLE bob LOGIN;
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Adding_roles%2B.jpg)

## 2. Ограничение использования trust

```
\q
sudo sed -i 's/^local.*all.*all.*trust.*$/local all postgres,student trust\n/' /etc/postgresql/16/main/pg_hba.conf
psql
SELECT type,database,user_name,address,auth_method,error FROM pg_hba_file_rules ORDER BY rule_number;
SELECT pg_reload_conf();
\q
psql -l -U alice
psql -l -U bob
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Restriction_of_use_trust%2B.jpg)

## 3. Метод аутентификации peer

```
sudo sed -i '/^local.*all.*postgres,student.*$/alocal all alice,bob peer' /etc/postgresql/16/main/pg_hba.conf
psql
SELECT type,database,user_name,address,auth_method,error FROM pg_hba_file_rules ORDER BY rule_number;
SELECT pg_reload_conf();
\q
psql -l -U alice
psql -l -U bob
echo 'stmap student alice' | sudo tee -a /etc/postgresql/16/main/pg_ident.conf
sudo sed -i 's/peer.*$/peer map=stmap/' /etc/postgresql/16/main/pg_hba.conf
psql
SELECT type,database,user_name,address,auth_method,options,error FROM pg_hba_file_rules ORDER BY rule_number;
SELECT pg_reload_conf();
\q
psql -c '\conninfo' -U alice -d student
psql -c '\conninfo' -U bob -d student
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/Peer_authentication_method%2B.jpg)

## 4. Одно отображение для нескольких ролей

```
echo 'stmap student bob' | sudo tee -a /etc/postgresql/16/main/pg_ident.conf
sudo tail -n2 /etc/postgresql/16/main/pg_ident.conf
psql
SELECT pg_reload_conf();
\q
psql -c '\conninfo' -U alice -d student
psql -c '\conninfo' -U bob -d student
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_13_access_overview/One_mapping_for_multiple_roles%2B.jpg)
