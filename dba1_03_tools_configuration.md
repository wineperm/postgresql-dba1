# Практика

---

## Задание

1. Получите список параметров, для изменения которых
   требуется перезапуск сервера.
2. В дополнительном подключаемом файле конфигурации
   сделайте ошибку при изменении параметра max_connections.
   Перезапустите сервер. Убедитесь, что сервер не стартует,
   и проверьте журнал сообщений.
   Исправьте ошибку и запустите сервер.

---

## Решение

## 1. Параметры, изменение которых требует перезапуска сервера

```
sudo -i -u postgres
psql
SELECT name, setting, unit FROM pg_settings WHERE context = 'postmaster';
```

![Alt text]()

## 2. Установка параметра max_connections

```
\dconfig max_conn*
\q
echo max_connections=5O | tee /etc/postgresql/17/main/conf.d/max_connections.conf
psql
SELECT * FROM pg_file_settings WHERE name = 'max_connections'\gx
\q
sudo pg_ctlcluster 17 main restart
tail -n 5 /var/log/postgresql/postgresql-17-main.log
echo max_connections=50 | sudo tee /etc/postgresql/17/main/conf.d/max_connections.conf
sudo pg_ctlcluster 17 main start
psql
SHOW max_connections;
```

![Alt text]()

# Практика +

---

## Задание

1. Установите параметр work_mem = 32MB в командной строке  
   запуска утилиты psql.
2. В пакетном дистрибутиве для Ubuntu файл postgresql.conf
   находится не в каталоге PGDATA. Каким образом сервер
   находит этот файл конфигурации при запуске?

---

## Решение

## 1. Установка параметров при запуске приложения

```
psql "options='-c work_mem=32MB'" -c 'SHOW work_mem'
```

![Alt text]()

## 2. Где определяется config_file

```
psql
\dconfig (config_file|data_directory)
\q
sudo cat /var/lib/postgresql/17/main/postmaster.pid | head -n 1
ps -p 356299 -ho command
```

![Alt text]()
