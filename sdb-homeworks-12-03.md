Домашнее задание к занятию «SQL. Часть 1» - `Эсадов Роман`
---
### Задание 1
```
SELECT DISTINCT  district
FROM address a
WHERE district LIKE 'K%a'
AND district NOT LIKE 'K% %a';
```
![Задание 1](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL_1.png)

---
### Задание 2
```
SELECT * FROM payment p
WHERE payment_date >= '2005-06-15' AND payment_date < '2005-06-19'
AND amount > 10.00;
```
![Задание 2](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL_22.png)

---
### Задание 3
```
SELECT * FROM rental r ORDER BY  rental_date DESC LIMIT 5;
```
![Задание 3](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL_3.png)

---
### Задание 4
```
SELECT customer_id, store_id, LOWER(REPLACE(first_name, 'LL', 'PP')), LOWER(last_name), email, address_id, active, create_date, last_update
FROM customer c 
WHERE active = 1 
and first_name = 'Kelly' 
or first_name = 'Willie';
```
![Задание 4](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL_4.png)
