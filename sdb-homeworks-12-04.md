Домашнее задание к занятию «SQL. Часть 2» - `Эсадов Роман`
---
### Задание 1
```
SELECT s.last_name, s.first_name, c.city, COUNT(cr.store_id) as clinet_count FROM staff s 
LEFT JOIN address a ON a.address_id = s.address_id 
LEFT JOIN city c ON c.city_id = a.city_id
LEFT JOIN customer cr ON cr.store_id = s.store_id
GROUP BY s.staff_id
HAVING COUNT(cr.store_id) > 300;
```
![Задание 1](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL4_1.png)

---
### Задание 2
```
SELECT COUNT(*)  FROM film f
WHERE `length` > (SELECT AVG(`length`) FROM film f2);
```
![Задание 2](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL4_2.png)

---
### Задание 3
```
SELECT YEAR(payment_date) as `year`, MONTH(payment_date) AS `month`, SUM(amount) AS paym_sum, COUNT(rental_id) as rental_cnt 
FROM payment
GROUP BY YEAR(payment_date), MONTH(payment_date)
ORDER BY paym_sum DESC
LIMIT 1;
```
Вариант с JOIN rental
```
SELECT 
	most_profit.`year`,
    most_profit.`month`, 
    most_profit.paym_sum, 
    rental_counts.rental_cnt
FROM (
    SELECT `year`, `month`, paym_sum
    FROM (
        SELECT YEAR(payment_date) as `year`, MONTH(payment_date) AS `month`, SUM(amount) AS paym_sum
        FROM payment
        GROUP BY YEAR(payment_date), MONTH(payment_date)
    ) AS most_profit
    WHERE paym_sum = (
        SELECT MAX(paym_sum)
        FROM (
            SELECT MONTH(payment_date) AS `month`, SUM(amount) AS paym_sum
            FROM payment
            GROUP BY MONTH(payment_date)
        ) AS max_paym_sum
    )
) AS most_profit
LEFT JOIN (
    SELECT r_month, rental_cnt
    FROM (
        SELECT MONTH(rental_date) AS r_month, count(rental_id) as rental_cnt 
        FROM rental
        GROUP BY r_month
    ) AS r_cnt
    GROUP BY r_month
) AS rental_counts
ON most_profit.`month` = rental_counts.r_month;
```
![Задание 3](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL4_3.png)

---
