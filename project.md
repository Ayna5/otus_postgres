# Сравнение производительности PostgreSQL и MySQL

## Характеристики тестового стенда

|Характеристика|ВМ с PostgreSQL| ВМ с MySQL |
|--------------|-----------------|---------------|
|Hostname|	otus-postgres|	otus-mysql|
|CPU|   2|   2|
|RAM| 4|	4|
|Тип накопителя|	HDD|	HDD|
|Версия БД| 	15.7|	8.0.36|

## Настройка тестового стенда

Создала 2 VM. В одной установила бд postgres, в другой mysql.

<img width="765" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/c49b442f-f01e-4e06-9234-efc8e841da79">
<img width="764" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/ae924d9f-3458-4aef-89ee-58f12ea14329">

Зальем данные в бд postgres (https://edu.postgrespro.ru/demo-big.zip)

```
wget https://edu.postgrespro.ru/demo-big.zip && sudo apt install unzip && unzip demo-big.zip
pwd
ls -la
sudo -u postgres psql < /home/postgres/demo-big-20170815.sql
```

<img width="919" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/11920e57-517e-45a3-8efd-63b9a03362d9">
<img width="939" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/0e7783dc-5689-47a7-8fd9-d2a8a9dcab96">
<img width="941" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/ef89ae71-3e5f-416f-a209-1344f91bc9f2">

Зальем данные в бд mysql (https://edu.postgrespro.ru/demo-big.zip)

```
-- конвертируем дамп бд postgres в mysql
./pg2mysql.pl < demo-big-20170815.sql > mysql.sql
pwd

-- скопируем полученный файл в VM с базой данных mysql
scp /Users/aynagulagataeva/Downloads/mysql.sql mysql@158.160.153.106:/home/mysql
ls -la
sudo -u mysql psql < /home/mysql/mysql.sql
mysql -u mysql -p mysql < /home/mysql/mysql.sql
sqlite3 mysql < /home/mysql/mysql.sql
```

<img width="926" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/1306f680-4c56-4713-90bf-7a25e0a33bcd">
<img width="935" alt="image" src="https://github.com/Ayna5/otus_postgres/assets/42717899/b16a9ae3-9b7c-4d92-b18f-6e13065ff7ec">


