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
