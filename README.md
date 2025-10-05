# Домашнее задание к занятию "`Репликация и масштабирование. Часть 1`" - `Евдокимов Андрей`

## Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.
## Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.
Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.
### Создано 2 контейнера - master и replica
<img width="1363" height="465" alt="image" src="https://github.com/user-attachments/assets/e54ee9d8-a3b0-465d-b88b-ddf1372b2b9f" />

**Состояния и режимы работы**
```sql
SHOW BINARY LOG STATUS\G
```
<img width="445" height="148" alt="image" src="https://github.com/user-attachments/assets/a418577c-8397-49da-af89-b65ae382bf39" />
```sql
SHOW REPLICA STATUS\G
```
<img width="638" height="1110" alt="image" src="https://github.com/user-attachments/assets/7832038f-2450-4b08-a517-04a7502548ab" />
```sql
Source_Host: master
Source_Port: 3306
Replica_IO_Running: Yes
```
Из этих параметров видно, что соединение с master, который на порту 3306 активно.

# Домашнее задание к занятию "`Индексы`" - `Евдокимов Андрей`

## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.
```sql
SELECT ((SUM(INDEX_LENGTH) / (SUM(INDEX_LENGTH) + SUM(DATA_LENGTH))) * 100) as index_percentage
FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = 'sakila';
```
<img width="758" height="203" alt="image" src="https://github.com/user-attachments/assets/16a8dea1-1c98-4c17-9d86-adffcf475be7" />

## Задание 2
Выполнен explain analyze следующего запроса:
```sql
explain analyze select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
### Резульат explain analyze:
-> Table scan on <temporary> (cost=2.5..2.5 rows=0) (actual time=2345..2345 rows=391 loops=1)\n<br>
-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=2345..2345 rows=391 loops=1)\n<br>
-> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1498..2246 rows=642000 loops=1)\n<br>
-> Sort: c.customer_id, f.title  (actual time=1498..1538 rows=642000 loops=1)\n<br>
-> Stream results  (cost=23.1e+6 rows=16.5e+6) (actual time=7.94..1153 rows=642000 loops=1)\n<br>
-> Nested loop inner join  (cost=23.1e+6 rows=16.5e+6) (actual time=7.81..995 rows=642000 loops=1)\n<br>
-> Nested loop inner join  (cost=21.5e+6 rows=16.5e+6) (actual time=1.04..860 rows=642000 loops=1)\n<br>
-> Nested loop inner join  (cost=19.8e+6 rows=16.5e+6) (actual time=1.02..725 rows=642000 loops=1)\n<br>
-> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.755..28 rows=634000 loops=1)\n<br>
-> Filter: (cast(p.payment_date as date) = \'2005-07-30\')  (cost=1.72 rows=16500) (actual time=0.143..3.51 rows=634 loops=1)\n<br>
-> Table scan on p  (cost=1.72 rows=16500) (actual time=0.134..2.46 rows=16044 loops=1)\n<br>
-> Hash\n<br>
-> Covering index scan on f using idx_title  (cost=103 rows=1000) (actual time=0.317..0.55 rows=1000 loops=1)\n<br>
-> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=1 rows=1) (actual time=671e-6..987e-6 rows=1.01 loops=634000)\n<br>
-> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=83.4e-6..103e-6 rows=1 loops=642000)\n<br>
-> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.001 rows=1) (actual time=85.6e-6..106e-6 rows=1 loops=642000)\n<br>

### Решение:
Из результата explain analyze можно видеть, что запрос довольно медленный и с высоким cost. Для оптимизации его можно откорректировать в зависимости от желаемого итогового результата. Исходя из вида исходного запроса можно рассмотреть два случая: 
### 1) Цель: получить данные обо всех тратах конкретного покупателя за 2005-07-30.
В этом случае обращение к таблицам inventory и film излишни, но join с них без условия фильтрации тратит мног ресурсов, что видно по строке "Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.755..28 rows=634000 loops=1)\n". Поэтому запрос можно исправить так, что так же и оптимизирует его: 
```sql
select distinct  concat(c.last_name, ' ', c.first_name ), sum(p.amount) over (partition by c.customer_id )
from payment p, rental r, customer c
where p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-30 23:59:59'  and p.payment_date = r.rental_date and r.customer_id = c.customer_id
```
### Резульат explain analyze:
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=6.37..6.41 rows=391 loops=1)\n    <br>
-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=6.37..6.37 rows=391 loops=1)\n <br>       
-> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=5.31..6.21 rows=642 loops=1)\n     <br>       
-> Sort: c.customer_id  (actual time=5.29..5.32 rows=642 loops=1)\n            <br>    
-> Stream results  (cost=4276 rows=1835) (actual time=0.237..5.17 rows=642 loops=1)\n     <br>               
-> Nested loop inner join  (cost=4276 rows=1835) (actual time=0.23..5 rows=642 loops=1)\n    <br>                    
-> Nested loop inner join  (cost=3633 rows=1835) (actual time=0.221..4.54 rows=642 loops=1)\n     <br>                       
-> Filter: ((p.payment_date >= TIMESTAMP\'2005-07-30 00:00:00\') and (p.payment_date < TIMESTAMP\'2005-07-30 23:59:59\'))  (cost=1674 rows=1833) (actual time=0.204..3.48 rows=634 loops=1)\n     <br>                           
-> Table scan on p  (cost=1674 rows=16500) (actual time=0.192..2.64 rows=16044 loops=1)\n                            <br>
-> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1) (actual time=0.00113..0.00154 rows=1.01 loops=634)\n       <br>                 
-> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=579e-6..602e-6 rows=1 loops=642)\n'<br>


### 2) Цель: получить данные тратах конкретного покупателя на конкретный фильм. 
Во этом случае обращения к таблицам inventory и film необходимы. Но нужно указать условие фильтрации "", иначе будут создаваться дуликаты для всех 1000 фильмов из film, что негативно скажется на производительности и не даст желаемого результата. <br>
Итоговый запрос:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id and i.film_id = f.film_id
```
### Резульат explain analyze:
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=7.03..7.08 rows=602 loops=1)\n<br>
-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=7.03..7.03 rows=602 loops=1)\n<br>
-> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=5.98..6.88 rows=642 loops=1)\n<br>
-> Sort: c.customer_id, f.title  (actual time=5.96..6 rows=642 loops=1)\n<br>
-> Stream results  (cost=36656 rows=16520) (actual time=0.187..5.76 rows=642 loops=1)\n<br>
-> Nested loop inner join  (cost=36656 rows=16520) (actual time=0.182..5.54 rows=642 loops=1)\n<br>
-> Nested loop inner join  (cost=30875 rows=16520) (actual time=0.179..4.97 rows=642 loops=1)\n<br>
-> Nested loop inner join  (cost=25093 rows=16520) (actual time=0.176..4.41 rows=642 loops=1)\n<br>
-> Nested loop inner join  (cost=19311 rows=16520) (actual time=0.17..3.99 rows=642 loops=1)\n<br>
-> Filter: (cast(p.payment_date as date) = \'2005-07-30\')  (cost=1674 rows=16500) (actual time=0.156..3.06 rows=634 loops=1)\n<br>
-> Table scan on p  (cost=1674 rows=16500) (actual time=0.148..2.31 rows=16044 loops=1)\n<br>
-> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1) (actual time=981e-6..0.00136 rows=1.01 loops=634)\n<br>
-> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=521e-6..541e-6 rows=1 loops=642)\n<br>
-> Single-row index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.25 rows=1) (actual time=746e-6..767e-6 rows=1 loops=642)\n<br>
-> Single-row index lookup on f using PRIMARY (film_id=i.film_id)  (cost=0.25 rows=1) (actual time=753e-6..774e-6 rows=1 loops=642)\n<br>

### Исследуем влияние на оптимизацию запроса добавления индекса для 'payment_date' на примере первого запроса
<img width="1051" height="282" alt="image" src="https://github.com/user-attachments/assets/92eb9f09-bbbf-42c0-ac77-09284985e67d" />

```sql
select distinct  concat(c.last_name, ' ', c.first_name ), sum(p.amount) over (partition by c.customer_id )
from payment p, rental r, customer c
where p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-30 23:59:59'  and p.payment_date = r.rental_date and r.customer_id = c.customer_id
```

### Резульат explain analyze:
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4.01..4.04 rows=391 loops=1)\n<br>    
-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4.01..4.01 rows=391 loops=1)\n<br>        
-> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=3.34..3.9 rows=642 loops=1)\n<br>            
-> Sort: c.customer_id  (actual time=3.32..3.34 rows=642 loops=1)\n<br>                
-> Stream results  (cost=582 rows=661) (actual time=0.0877..3.19 rows=642 loops=1)\n<br>                    
-> Nested loop inner join  (cost=582 rows=661) (actual time=0.0834..3.03 rows=642 loops=1)\n<br>                        
-> Nested loop inner join  (cost=351 rows=634) (actual time=0.0696..0.779 rows=634 loops=1)\n<br>                            
-> Filter: ((r.rental_date >= TIMESTAMP\'2005-07-30 00:00:00\') and (r.rental_date < TIMESTAMP\'2005-07-30 23:59:59\'))  (cost=129 rows=634) (actual time=0.0578..0.212 rows=634 loops=1)\n<br>                                
-> Covering index range scan on r using rental_date over (\'2005-07-30 00:00:00\' <= rental_date < \'2005-07-30 23:59:59\')  (cost=129 rows=634) (actual time=0.0555..0.144 rows=634 loops=1)\n<br>                            
-> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=767e-6..787e-6 rows=1 loops=634)\n<br>                        
-> Index lookup on p using idx_payment_date (payment_date=r.rental_date)  (cost=0.261 rows=1.04) (actual time=0.00309..0.00338 rows=1.01 loops=634)\n<br>

### Вывод: 
Оба варианта оптимизации, основанные на верном понимании целеполагания формирования выборки значительно ускоряют получение данных. Добавление индекса смогло ускорить запрос 6.37 мс до 4.04 мс.

# Домашнее задание к занятию "`SQL. Часть 2`" - `Евдокимов Андрей`

## Задание 1
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.
```sql
WITH customer_counts AS (
    SELECT store_id, COUNT(*) as customer_count 
    FROM sakila.customer 
    GROUP BY store_id 
    HAVING COUNT(*) > 300
)

SELECT 
    st.first_name,
    st.last_name,
    city.city,
    cc.customer_count
FROM customer_counts cc
JOIN sakila.store s ON cc.store_id = s.store_id
JOIN sakila.staff st ON s.manager_staff_id = st.staff_id
JOIN sakila.address a ON s.address_id = a.address_id
JOIN sakila.city city ON a.city_id = city.city_id;
```
<img width="472" height="461" alt="image" src="https://github.com/user-attachments/assets/6f3bffb8-02f8-4ac9-b977-ce1290a00d61" />

## Задание 2
Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.
```sql
SELECT COUNT(*)
FROM sakila.film 
WHERE length > (SELECT AVG(length) FROM sakila.film);
```
<img width="451" height="220" alt="image" src="https://github.com/user-attachments/assets/0906a434-fee2-46ef-827d-2d73df577498" />

## Задание 3
Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.
```sql
WITH total_month AS (
    SELECT
        MONTHNAME(payment_date) AS month_name,
        SUM(amount) AS total_sales,
        COUNT(rental_id) AS rental_count
    FROM sakila.payment 
    GROUP BY MONTH(payment_date), MONTHNAME(payment_date)
)

SELECT * FROM total_month
ORDER BY total_sales DESC 
LIMIT 1;
```
<img width="479" height="351" alt="image" src="https://github.com/user-attachments/assets/98416563-1c3c-4170-9770-9d241d72e1a3" />


# Домашнее задание к занятию "`SQL. Часть 1`" - `Евдокимов Андрей`

## Задание 1
Получите уникальные названия районов из таблицы с адресами, которые начинаются на “K” и заканчиваются на “a” и не содержат пробелов.
```sql
SELECT DISTINCT district FROM address WHERE LEFT(district, 1) = 'K' AND RIGHT(district, 1) = 'a' AND POSITION(' ' IN district) = 0;
```
<img width="746" height="352" alt="image" src="https://github.com/user-attachments/assets/2a319128-0453-493b-9cf3-e59f2c605d40" />

## Задание 2
Получите из таблицы платежей за прокат фильмов информацию по платежам, которые выполнялись в промежуток с 15 июня 2005 года по 18 июня 2005 года включительно и стоимость которых превышает 10.00.
```sql
SELECT * FROM payment WHERE payment_date BETWEEN '2005-06-15' AND '2005-06-18 23:59:59' AND amount > 10.00;
```
<img width="829" height="368" alt="image" src="https://github.com/user-attachments/assets/cf7cef9f-5c9f-4590-ba4f-b3b8b109bf41" />

## Задание 3
```sql
SELECT * FROM rental ORDER BY rental_id DESC LIMIT 5;
```
<img width="668" height="342" alt="image" src="https://github.com/user-attachments/assets/035a6186-3918-4530-a0b0-563449e4b405" />

## Задание 4
Одним запросом получите активных покупателей, имена которых Kelly или Willie.
Сформируйте вывод в результат таким образом:
- все буквы в фамилии и имени из верхнего регистра переведите в нижний регистр,
- замените буквы 'll' в именах на 'pp'.
```sql
SELECT 
    customer_id,
    LOWER(first_name) as first_name,
    LOWER(last_name) as last_name,
    REPLACE(LOWER(first_name), 'll', 'pp') as first_name,
    active
FROM customer WHERE active = 1 AND (first_name = 'Kelly' OR first_name = 'Willie');
```
<img width="661" height="358" alt="image" src="https://github.com/user-attachments/assets/fa29e1cf-7f2c-4e38-9e78-b3dd4e7abbf2" />

# Домашнее задание к занятию "`Работа с данными (DDL/DML)`" - `Евдокимов Андрей`

### Задание 1
1.1. Поднимите чистый инстанс MySQL версии 8.0+. Можно использовать локальный сервер или контейнер Docker.

1.2. Создайте учётную запись sys_temp. 
```sql
CREATE USER 'sys_temp'@'localhost' IDENTIFIED BY 'password';
```
1.3. Выполните запрос на получение списка пользователей в базе данных. (скриншот)
```sql
SELECT user, host FROM mysql.user;
```
<img width="620" height="278" alt="image" src="https://github.com/user-attachments/assets/7ab6908e-3b11-4cb9-bd51-8629f95c117f" />

1.4. Дайте все права для пользователя sys_temp.
```sql
GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'localhost';
FLUSH PRIVILEGES;
```
1.5. Выполните запрос на получение списка прав для пользователя sys_temp. (скриншот)
```sql
SHOW GRANTS FOR 'sys_temp'@'localhost';
```
<img width="2550" height="396" alt="image" src="https://github.com/user-attachments/assets/13c0de7d-318f-4063-a58e-7cd583e4c042" />

1.6. Переподключитесь к базе данных от имени sys_temp.<br>
<img width="253" height="120" alt="image" src="https://github.com/user-attachments/assets/bc44abe1-ada4-4368-820e-5db6331c0b82" />

Для смены типа аутентификации с sha2 используйте запрос: 
```sql
ALTER USER 'sys_test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
1.6. По ссылке https://downloads.mysql.com/docs/sakila-db.zip скачайте дамп базы данных.

1.7. Восстановите дамп в базу данных.

1.8. При работе в IDE сформируйте ER-диаграмму получившейся базы данных. При работе в командной строке используйте команду для получения всех таблиц базы данных. (скриншот)
<img width="2011" height="1379" alt="image" src="https://github.com/user-attachments/assets/05e4b325-08bb-4a75-8497-ea2440c2a351" />

*Результатом работы должны быть скриншоты обозначенных заданий, а также простыня со всеми запросами.*

### Задание 2
Составьте таблицу, используя любой текстовый редактор или Excel, в которой должно быть два столбца: в первом должны быть названия таблиц восстановленной базы, во втором названия первичных ключей этих таблиц. Пример: (скриншот/текст)
```
Название таблицы | Название первичного ключа
customer         | customer_id
```
#### Запрос:
```sql
SELECT TABLE_NAME AS 'Название таблицы', COLUMN_NAME AS 'Название первичного ключа' FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE TABLE_SCHEMA = 'sakila' AND CONSTRAINT_NAME = 'PRIMARY' ORDER BY TABLE_NAME;
```
#### Результат:
```
+---------------------------------+--------------------------------------------------+
| Название таблицы                | Название первичного ключа                        |
+---------------------------------+--------------------------------------------------+
| actor                           | actor_id                                         |
| address                         | address_id                                       |
| category                        | category_id                                      |
| city                            | city_id                                          |
| country                         | country_id                                       |
| customer                        | customer_id                                      |
| film                            | film_id                                          |
| film_actor                      | actor_id                                         |
| film_actor                      | film_id                                          |
| film_category                   | film_id                                          |
| film_category                   | category_id                                      |
| film_text                       | film_id                                          |
| inventory                       | inventory_id                                     |
| language                        | language_id                                      |
| payment                         | payment_id                                       |
| rental                          | rental_id                                        |
| staff                           | staff_id                                         |
| store                           | store_id                                         |
+---------------------------------+--------------------------------------------------+
```
# Домашнее задание к занятию "`Базы данных`" - `Евдокимов Андрей`

`При необходимости прикрепитe сюда скриншоты

![db-scheme](https://raw.githubusercontent.com/EvdokimovAndrey/sys-pattern-homework-12-01-hw/refs/heads/main/img/db-schema.png)`
