Домашнее задание к занятию «Индексы» - `Эсадов Роман`
---
### Задание 1
```
SELECT sum(DATA_LENGTH) as schema_weight, sum(INDEX_LENGTH) as index_weight, (sum(INDEX_LENGTH)/sum(DATA_LENGTH))*100 as index_weight
FROM information_schema.tables
WHERE TABLE_SCHEMA = 'sakila';
```
![Задание 1](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL5_1.png)

---
### Задание 2
`
Результат Explain analyze:
-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=6885..6885 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=6885..6885 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=6885..6885 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=2876..6670 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=2876..2960 rows=642000 loops=1)
                    -> Stream results  (cost=22.2e+6 rows=16.7e+6) (actual time=0.369..2039 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=22.2e+6 rows=16.7e+6) (actual time=0.363..1714 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=20.5e+6 rows=16.7e+6) (actual time=0.359..1509 rows=642000 loops=1)
                                -> Nested loop inner join  (cost=18.9e+6 rows=16.7e+6) (actual time=0.354..1289 rows=642000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1.61e+6 rows=16.1e+6) (actual time=0.34..65 rows=634000 loops=1)
                                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.68 rows=16086) (actual time=0.0289..7.55 rows=634 loops=1)
                                            -> Table scan on p  (cost=1.68 rows=16086) (actual time=0.0192..5.17 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0446..0.228 rows=1000 loops=1)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.04) (actual time=0.00123..0.00178 rows=1.01 loops=634000)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=164e-6..190e-6 rows=1 loops=642000)
                            -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=144e-6..169e-6 rows=1 loops=642000)
Общее время выполнения запроса: 7.258s`
![Задание 2.1](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL5_2.png)
`Исходя из анализа дедупликация с distinct, окнная функция over, не явно прописанные джоины, фильтрация с функцией date занимают много времени.
Предложение по ускорению выборки:`
1. исполльзование группировки по customer_name и customer_id вместо distict и оконной функции over
2. явно прописанные джоины, а также 
3. фильтрация по столбцу p.payment_date с указанием конечной даты без функции date
4. дополнительной индексация в таблице payment по payment_date

`Индекс создавался командой:`
```
CREATE INDEX idx_payment_date ON payment(payment_date);
```
`Итоговый запрос:`
```
select concat(c.last_name, ' ', c.first_name) as customer_name, sum(p.amount) as total_amount
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
join inventory i on r.inventory_id = i.inventory_id
join film f on i.film_id = f.film_id
where p.payment_date >= '2005-07-30 00:00:00' and p.payment_date < '2005-07-31 00:00:00'
group by c.customer_id, customer_name;
```
`
Результат Explain analyze после совершения всех манипуляций:
-> Limit: 200 row(s)  (actual time=5.53..5.56 rows=200 loops=1)
    -> Table scan on <temporary>  (actual time=5.53..5.55 rows=200 loops=1)
        -> Aggregate using temporary table  (actual time=5.53..5.53 rows=391 loops=1)
            -> Nested loop inner join  (cost=1021 rows=645) (actual time=0.0478..4.77 rows=642 loops=1)
                -> Nested loop inner join  (cost=798 rows=645) (actual time=0.0445..4.07 rows=642 loops=1)
                    -> Nested loop inner join  (cost=575 rows=645) (actual time=0.0417..3.37 rows=642 loops=1)
                        -> Nested loop inner join  (cost=349 rows=634) (actual time=0.0296..1.18 rows=634 loops=1)
                            -> Filter: ((r.rental_date >= TIMESTAMP'2005-07-30 00:00:00') and (r.rental_date < TIMESTAMP'2005-07-31 00:00:00'))  (cost=127 rows=634) (actual time=0.0207..0.372 rows=634 loops=1)
                                -> Covering index range scan on r using rental_date over ('2005-07-30 00:00:00' <= rental_date < '2005-07-31 00:00:00')  (cost=127 rows=634) (actual time=0.0188..0.273 rows=634 loops=1)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.00112..0.00114 rows=1 loops=634)
                        -> Index lookup on p using idx_payment_date (payment_date=r.rental_date)  (cost=0.254 rows=1.02) (actual time=0.00282..0.00329 rows=1.01 loops=634)
                    -> Single-row index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.246 rows=1) (actual time=934e-6..953e-6 rows=1 loops=642)
                -> Single-row covering index lookup on f using PRIMARY (film_id=i.film_id)  (cost=0.246 rows=1) (actual time=924e-6..944e-6 rows=1 loops=642)
Общее время выполнения запроса: 10ms
`
![Задание 2.2](https://github.com/BeastieBoy93/sdb-homeworks/blob/sdbsql-24/SQL5_3.png)

---
