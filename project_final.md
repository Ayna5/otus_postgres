# Настройка различных видов резервного копирования сторонними средствами

## Характеристики тестового стенда

|Характеристика|ВМ с PostgreSQL|
|--------------|---------------|
|Hostname|	otus-vm|
|CPU|   2|
|RAM| 4|
|Тип накопителя|	HDD|
|PostgreSQL| 	15.7|

## Настройка тестового стенда

Создадим VM и установим PostgreSQL.

```
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
```

<img width="940" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/fdca9c55-5351-496f-b594-4c0f272c5f71">

Создадим бд project и 5 таблиц. Заполним таблицы случанными данными по 10 млн. Размер бд = 3264 MB.
```sql
CREATE DATABASE project;
\c project
CREATE TABLE table1 (id SERIAL, text TEXT);
INSERT INTO table1 (text) SELECT md5(random()::text) FROM generate_series(1, 10000000);
CREATE TABLE table2 (id SERIAL, text TEXT);
INSERT INTO table2 (text) SELECT md5(random()::text) FROM generate_series(1, 10000000);
CREATE TABLE table3 (id SERIAL, text TEXT);
INSERT INTO table3 (text) SELECT md5(random()::text) FROM generate_series(1, 10000000);
CREATE TABLE table4 (id SERIAL, text TEXT);
INSERT INTO table4 (text) SELECT md5(random()::text) FROM generate_series(1, 10000000);
CREATE TABLE table5 (id SERIAL, text TEXT);
INSERT INTO table5 (text) SELECT md5(random()::text) FROM generate_series(1, 10000000);
SELECT pg_size_pretty(pg_database_size('project'));
```

<img width="747" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/a02feb88-3d9c-4509-a3e7-e8663835f7d1">

## barman

Создадим VM и установим barman
```
sudo apt update && sudo apt install barman
```

<img width="881" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/1cefe3fa-6482-41be-9aef-216f75427416">
<img width="977" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/7f51de99-a9c2-417a-9bbd-14116338891c">

Создаю пользователя barman в postgres
```
createuser --interactive -P barman
createuser --replication -P streaming_barman
```

<img width="397" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/c68a7e53-6f26-47da-87de-e7f3d6c40d76">
<img width="517" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/5e4248d6-10db-41ac-8be7-7382860d0bcc">

Записываю креды в файл .pgpass у пользователя barman на сервере бекапов
```
echo "158.160.130.154:5432:*:barman:pass" >> ~/.pgpass
vim .pgpass
```

<img width="383" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/10e07138-832b-42b3-ba96-8567e9232652">

Добавляю разрешение на подключени в pg_hba.conf

<img width="806" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/ed87d091-b984-41d5-98e8-bd8906290363">



