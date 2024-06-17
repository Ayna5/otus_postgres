# Настройка резервного копирования barman

## Характеристики тестового стенда

|Характеристика|ВМ с PostgreSQL|ВМ с бекапом|
|--------------|---------------|---------------|
|Hostname|	otus-postgres| otus-pgbackup |
|CPU|   2|   2|
|RAM| 4| 4|
|Тип накопителя| SSD| SSD|
|PostgreSQL| 15.7||

## Настройка тестового стенда

Создадим VM и установим PostgreSQL и создадим новый кластер.

```
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
```

<img width="940" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/fdca9c55-5351-496f-b594-4c0f272c5f71">
<img width="923" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/dafd5362-ded1-4e4a-b8bb-5e2e3cb1276b">

Включила archive_mode
```sql
alter system set archive_mode = on;
```

<img width="377" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/d1d8d3c6-1b43-4df8-b393-27ce1cc54fa5">

```sql
alter system set archive_command = 'gzip < %p > /var/lib/postgresql/archive/%f.gz';
select pg_reload_conf();
```

<img width="696" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/83d1b004-3e12-4de7-b448-27cd8fc29ede">

Создадим бд project и 1 таблицу. Заполним таблицу случанными данными на 1 млн. Размер бд = 73 MB.
```sql
CREATE DATABASE project;
\c project
CREATE TABLE tab1 (id SERIAL, text TEXT);
INSERT INTO tab1 (text) SELECT md5(random()::text) FROM generate_series(1, 1000000);
SELECT pg_size_pretty(pg_database_size('project'));
```

<img width="477" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/e5491679-e708-48fb-9956-7f7ce6703f18">
<img width="695" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/0f8872bd-c341-44de-aceb-b47bfa232148">
<img width="471" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/190f9cb9-d395-4a11-a12b-9a64a8ffa2bd">

## barman

Создадим VM и установим barman
```
sudo apt update && sudo apt install barman
```

<img width="858" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/37aca2e0-ad9b-4b9f-ac21-28fe4af64cf8">
<img width="927" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/1c9c9cdd-84a4-4aa8-8a97-d1cbe3322579">
<img width="940" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/eeb8ef44-dda8-42e2-b7f6-f40bcf7f6e5b">

Создаю пользователей в PostgreSQL
```
createuser --interactive -P barman
createuser --replication -P streaming_barman
```

<img width="533" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/f62272c9-ba01-4f19-afad-03697898644d">

Записываю креды в файл .pgpass у пользователя barman на сервере бекапов
```
echo "158.160.170.203:5432:*: streaming_barman:passpass" > ~/.pgpass
vim .pgpass
echo "158.160.170.203:5432:*:barman:pass" >> ~/.pgpass
```

<img width="709" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/ddcc1f84-23d0-4ccc-80d7-8a9a9fd72ed6">

Добавляю разрешение на подключени в pg_hba.conf
```
sudo nano /etc/postgresql/15/main/pg_hba.conf
```

<img width="665" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/f8131826-95e7-4e7f-86a6-f799f2569d6b">

Есть глобальный конфиг /etc/barman.conf. Он содержит общие конфигурации резервного копирования, такие как файл журнала, пользователя резервного копирования, каталог резервного копирования. Его оставляем неизменным.
Файлы конфигурации сервера по умолчанию расположены в каталоге /etc/barman.d. В этом каталоге доступны файлы конфигурации шаблона для методов резервного копирования (потоковая передача, ssh и rsync). Делаю копию готового шаблона и правлю
```
cd /etc/barman.d/
sudo cp streaming-server.conf-template otus-postgres.conf
sudo nano otus-postgres.conf
```

<img width="692" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/a3ae3887-0bd7-4a6c-ac01-79df40160691">

Конфиг

<img width="557" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/9fbf9874-74cd-4086-afcb-673acb85b238">

Проверим доступ созданного нами бармена с помощью команды ниже
```
psql -c 'SELECT version()' -U barman -h 158.160.170.203 postgres
```

<img width="921" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/242ce1cd-bc22-40ed-b110-e1c681384de4">

Проверяем статус barman
```
sudo su - barman
barman check otus-postgres
```

<img width="724" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/3b84dc1d-b1f2-4655-b33b-bb80d9a944e0">

Отредактируем файл, рестартанем кластер и проверим статус barman вновь
```
listen_addresses = '*'
sudo systemctl restart postgresql@15-main
```

<img width="940" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/46530bd9-0d8f-4e45-9b7f-a25a346e5b6e">

Создадим слот otus-postgres
```
barman receive-wal --create-slot otus-postgres
barman check otus-postgres
```

<img width="760" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/bf1eac7e-331b-4598-9506-5d5bdaff70e9">

```
barman cron
barman check otus-postgres
```

<img width="747" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/360f6f7f-d8fe-4da5-8f01-f6e6e6e82fef">

```
barman receive-wal otus-postgres
```

<img width="694" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/cb5581cd-d752-482e-a343-0060aeea93a3">

Из-за несовместимости pg_receivewal версии с PostgreSQL версии не смогла продвинуться далее :(

Если бы конфигурация barman была настроена успешно, то нам оставалось бы только выполнить бекап:

```barman backup otus-postgres```

## pg_probackup
