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
