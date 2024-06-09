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

Установка barman
```
sudo apt update && sudo apt install barman
```

<img width="803" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/099d9726-45f7-424e-bab5-6d80d0f28e04">
