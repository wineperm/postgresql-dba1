# Практика +

---

## Задание

Установите PostgreSQL из исходных кодов так, как это
показано в демонстрации.
Создайте кластер баз данных, запустите сервер.
Убедитесь, что сервер работает.
Остановите сервер.

---

## Решение

## Сборка PostgreSQL из исходных кодов.

## Распаковка архива

```
mkdir postgresql
cd postgresql
wget https://ftp.postgresql.org/pub/source/v17.0/postgresql-17.0.tar.gz
tar -xzf /home/vagrant/postgresql/postgresql-17.0.tar.gz
rm postgresql-17.0.tar.gz
cd postgresql-17.0/
ll
```

![Alt text]()

## Создание конфигурации

```
sudo apt-get update
sudo apt-get install -y bison
sudo apt-get install -y flex
sudo apt-get install -y libreadline-dev
sudo apt-get install -y zlib1g-dev
./configure --prefix=/home/vagrant/postgresql/pgsql17 --with-pgport=5555 --without-icu
```

![Alt text]()

## Сборка и установка PostgreSQL

```
make
make install
cd contrib
make
make install
```

## Создание кластера

```
mkdir /home/vagrant/postgresql/pgsql17/data
export PGDATA=/home/vagrant/postgresql/pgsql17/data
export PATH=/home/vagrant/postgresql/pgsql17/bin:$PATH
initdb -U postgres -k
```

![Alt text]()

## Управление сервером

```
pg_ctl start -l /home/vagrant/postgresql/logfile
psql -U postgres -p 5555 -c 'SELECT now();'
pg_ctl stop
```

![Alt text]()

---

## Практика

---

## Задание

Включение расчета контрольных сумм в кластере.

1. Остановите сервер.
2. Проверьте, рассчитываются ли контрольные суммы
   в кластере.
3. Включите расчет контрольных сумм.
4. Запустите сервер.

---

## Решение

```
sudo pg_ctlcluster 17 main status
sudo pg_ctlcluster 17 main stop
sudo pg_ctlcluster 17 main status
sudo /usr/lib/postgresql/17/bin/pg_checksums --check -D /var/lib/postgresql/17/main
sudo /usr/lib/postgresql/17/bin/pg_checksums --enable -D /var/lib/postgresql/17/main
sudo /usr/lib/postgresql/17/bin/pg_checksums --check -D /var/lib/postgresql/17/main
sudo pg_ctlcluster 17 main start
sudo pg_ctlcluster 17 main status
```

![Alt text]()
