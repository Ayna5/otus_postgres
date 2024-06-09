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
