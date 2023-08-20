# Домашнее задание к занятию «Индексы»

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение 1

Создан следующий запрос:

```sql
SELECT TABLE_SCHEMA, sum(DATA_LENGTH), sum(INDEX_LENGTH), ROUND(sum(INDEX_LENGTH)/sum(DATA_LENGTH)*100) as '%'
FROM INFORMATION_SCHEMA.TABLES
WHERE  TABLE_SCHEMA = 'sakila' ;
```
Пример работы:

![1](https://github.com/SKA1010/hw_db_5/assets/125235217/2984e715-983e-4308-a3ce-ad892ee59200)


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение 2

До оптимизации:

![2-1](https://github.com/SKA1010/hw_db_5/assets/125235217/d3e5e926-dc2c-454e-8f23-56f4298bf834)

Узкие места текущего запроса:

- Отсутствуют индексы, создал индекс payment_index в таблице payment, на поле payment_date (CREATE INDEX payment_index ON payment(payment_date);)
- Вместо перечисления таблиц в FROM (таблица film f лишняя) переписал запрос с использованием join 
- Вместо over() проводим группировку через partition by, по полям c.customer_id, f.title
- Меняем DISTINCT на GROUP BY

Оптимизированный запрос:

```sql

SELECT concat(c.last_name, ' ', c.first_name) as FI, SUM(p.amount) as 'SUM'
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN customer c ON r.customer_id = c.customer_id 
JOIN inventory i ON r.inventory_id = i.inventory_id 
WHERE p.payment_date >= '2005-07-30' AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY FI;

```
Послек оптимизации:

![image](https://github.com/SKA1010/hw_db_5/assets/125235217/ce6c3039-110f-4b17-b442-61cbf56cbe32)
