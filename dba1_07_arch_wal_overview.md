# Практика

---

## Задание

---

1. Средствами операционной системы найдите процессы,
   отвечающие за работу буферного кеша и журнала WAL.
2. Остановите PostgreSQL в режиме fast; снова запустите его.
   Просмотрите журнал сообщений сервера.
3. Теперь остановите в режиме immediate и снова запустите.
   Просмотрите журнал сообщений сервера и сравните
   с предыдущим разом.

## Решение

## 1. Процессы операционной системы

```
sudo cat /var/lib/postgresql/17/main/postmaster.pid
sudo ps -o pid,command --ppid 1046
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_07_arch_wal_overview/Operating_system_processes.jpg)

## 2. Остановка в режиме fast

```
sudo rm /var/log/postgresql/postgresql-17-main.log
sudo pg_ctlcluster 17 main stop
sudo pg_ctlcluster 17 main start
cat /var/log/postgresql/postgresql-17-main.log
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_07_arch_wal_overview/Stop_in_fast_mode.jpg)

## 3. Остановка в режиме immediate

```
sudo rm /var/log/postgresql/postgresql-17-main.log
sudo pg_ctlcluster 17 main stop -m immediate --skip-systemctl-redirect
sudo pg_ctlcluster 17 main start
cat /var/log/postgresql/postgresql-17-main.log
```

![Alt text](https://github.com/wineperm/postgresql-dba1/blob/main/dba1_07_arch_wal_overview/Stop_in_immediate_mode.jpg)
