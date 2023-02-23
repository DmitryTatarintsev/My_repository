### Здесь примеры моих запросов в PostgreSQL во время учебы. Это должно показывать мои скилы и заодно моя шпаргалка, так как после учебы практиковать sql мне негде :'(

| #  | Тип данных | Псевдоним | Что входит |
|:-|:-|:-|:-|
|Числовые типы	|integer	|int,int4	|Целые числа|
| |real	|float4	|Вещественные числа|
|Символьные типы	|character	|char(n)	|Строка фиксированной длины|
| |character varying|	|varchar(n)	|Строка нефиксированной длины|
| |text|		|Строка любой длины|
|Типы для работы с датой и временем|	timestamp without timezone|	timestamp|	Дата и время без данных о часовом поясе|
| |timestamp with timezone|	timestamptz|	Дата и время с данными о часовом поясе|
| |date|		|Дата в разных форматах|
| |time|		|Время 00:00:00 до 24:00:00|
| |interval|		|Интервал между датами|
|Логические типы	|boolean	|bool	|TRUE и аналоги: 'true', 't', 'yes', 'y', 'on', 1. FALSE и аналоги: 'false', 'f', 'no', 'n', 'off', 0.|


```sql
SELECT CAST(milliseconds AS varchar),
       CAST(bytes AS varchar)
FROM track;


SELECT billing_city,
       billing_address       
FROM invoice
WHERE total > 2
  AND NOT billing_country = 'USA'
  AND NOT billing_country = 'Brazil'
  AND CAST(invoice_date AS date)  >= date('2009-09-01')
  AND CAST(invoice_date AS date) <= date('2009-09-30');
```

|Выражение	|Что попадёт	|Пример кода	|Результат|
|:-|:-|:-|:-|
|'text%'	|Значения, которые начинаются с text	|WHERE quotes LIKE 'кот%'	|'кот просил еды' 'котелок не варит'|
|'%text'	|Значения, которые заканчиваются на text	|WHERE quotes LIKE '%кот'	|'куда пропал кот' 'двойной апперкот'|
|'%text%'	|Значения, в которых text занимает любую позицию	|WHERE quotes LIKE '%кот%'	|'кота зовут василий' 'они танцевали котильон'|
|'te%xt'	|Значения, которые начинаются на te и заканчиваются на xt	|WHERE quotes LIKE 'ко%т'	|'кот играет' 'консультант'|

```sql
SELECT name
FROM track
WHERE (milliseconds > 300000
      AND composer LIKE '%Bono%'
      AND genre_id BETWEEN 7 AND 10)
      OR bytes > 1000000000
      
      
SELECT total,
       customer_id
FROM invoice
WHERE billing_city IN ('Dublin',
                       'London',
                       'Paris',
                       'Boston',
                       'Berlin',
                       'Stuttgart'); 
		       
		       
SELECT *
FROM client
WHERE fax = NULL; 


SELECT *
FROM movie
WHERE release_year > 2010
  AND rental_duration > 4
  OR rental_duration < 5
  AND length > 90; 
  
  
SELECT billing_country, 
       COUNT(*) -- укажите нужные поля
FROM invoice
WHERE billing_postal_code IS NULL
      AND (billing_address LIKE '%Street%'
           OR billing_address LIKE '%Way%'
           OR billing_address LIKE '%Road%'
           OR billing_address LIKE '%Drive%') -- пропишите условие
GROUP BY billing_country -- сгруппируйте данные
HAVING COUNT(*) > 6; -- пропишите условие;


SELECT a.actor_id,
       a.first_name,
       a.last_name,
       c.first_name,
       c.last_name
FROM actor AS a
FULL OUTER JOIN client AS c ON a.last_name = c.last_name
LIMIT 10; 


SELECT i.billing_country,
       COUNT(i.total) AS total_purchases
FROM invoice AS i
WHERE i.billing_country IN ('USA',
                            'Germany',
                            'Brazil')
  AND EXTRACT(YEAR
              FROM cast(invoice_date AS date)) = 2009
GROUP BY i.billing_country
UNION
SELECT i.billing_country,
       COUNT(i.total) AS total_purchases
FROM invoice AS i
WHERE i.billing_country IN ('USA',
                            'Germany',
                            'Brazil')
  AND EXTRACT(YEAR
              FROM cast(invoice_date AS date)) = 2013
GROUP BY i.billing_country; 


SELECT DISTINCT c.name
FROM film_category AS fc
INNER JOIN film_actor AS fa ON fc.film_id=fa.film_id
INNER JOIN actor AS a ON fa.actor_id=a.actor_id
INNER JOIN category AS c ON fc.category_id=c.category_id
WHERE a.first_name = 'Uma' 
      AND a.last_name = 'Wood'
      
      
SELECT *
FROM (SELECT i.customer_id
      FROM invoice AS i 
      INNER JOIN client AS c ON i.customer_id = c.customer_id
      GROUP BY i.customer_id
      ORDER BY SUM(total) DESC
      LIMIT 10) AS c_1
ORDER BY c_1.customer_id; 


SELECT customer_id
FROM client
WHERE customer_id IN (SELECT customer_id
                      FROM invoice
                      GROUP BY customer_id
                      ORDER BY SUM(total) DESC
                      LIMIT 10);
		      
		      
SELECT *, 
       DENSE_RANK() OVER (ORDER BY user_id)
FROM online_store.orders 
WHERE user_id IN (300768196,
                  840452722,
                  59432616)
ORDER BY user_id; 


SELECT *, 
       RANK() OVER (ORDER BY user_id)
FROM online_store.orders
WHERE user_id IN (300768196,
                  840452722,
                  59432616)
ORDER BY user_id; 


SELECT *, 
       ROW_NUMBER() OVER (ORDER BY user_id)
FROM online_store.orders
WHERE user_id IN (300768196,
                  840452722,
                  59432616)
ORDER BY user_id; 



WITH 
-- первый подзапрос с псевдонимом i
i AS (SELECT billing_country AS country,
             COUNT(total) AS total_invoice
      FROM invoice
      GROUP BY billing_country
      ORDER BY total_invoice DESC
      LIMIT 5), -- подзапросы разделяют запятыми
-- второй подзапрос с псевдонимом c
 c AS (SELECT country AS country,
              COUNT(customer_id) AS total_clients
              FROM client
              GROUP BY country)

-- основной запрос, в котором указаны псевдонимы для подзапросов
SELECT i.country,
       i.total_invoice,
       c.total_clients
FROM i LEFT OUTER JOIN c ON i.country=c.country
ORDER BY i.total_invoice DESC; 


WITH 
costs AS
(SELECT DISTINCT DATE_TRUNC('month',created_at)::date AS month,
        SUM(costs) OVER(PARTITION BY DATE_TRUNC('month',created_at)::date) AS costs
FROM tools_shop.costs
ORDER BY month)

SELECT *,
       costs - LAG(costs, 1, costs) OVER(ORDER BY month)
FROM costs


SELECT user_id,
       event_dt,
       LAG(event_dt) OVER (PARTITION BY user_id ORDER BY event_dt) AS previous_order_dt,
       LEAD(event_dt) OVER (PARTITION BY user_id ORDER BY event_dt) AS next_order_dt
FROM online_store.orders
WHERE user_id IN (300768196,
                  840452722,
                  59432616)
ORDER BY user_id; 
```
```
user_id	event_dt	previous_order_dt	next_order_dt
59432616	2020-06-17	[NULL]		[NULL]
300768196	2020-06-25	[NULL]		2020-06-27
300768196	2020-06-27	2020-06-25	[NULL]
840452722	2020-06-19	[NULL]		2020-06-21
840452722	2020-06-21	2020-06-19	2020-06-24
840452722	2020-06-24	2020-06-21	[NULL]
```
```sql


SELECT total,
       CASE
           WHEN total < 5 THEN 'маленький'
           WHEN total >= 5 AND total < 10 THEN 'средний'
           WHEN total >= 10 THEN 'крупный'
       END
FROM invoice
LIMIT 10; 
```
```
total	case
1.98	маленький
3.96	маленький
5.94	средний
8.91	средний
13.86	крупный
0.99	маленький
1.98	маленький
1.98	маленький
3.96	маленький
5.94	средний
```

```sql
WITH 
fund AS 
    (SELECT EXTRACT(MONTH FROM funding_round.funded_at) AS month,
            COUNT(DISTINCT fund.id) AS count_of_fund
    FROM funding_round
        LEFT JOIN investment ON investment.funding_round_id = funding_round.id
        LEFT JOIN fund ON fund.id = investment.fund_id
    WHERE EXTRACT(YEAR FROM funding_round.funded_at) BETWEEN 2010 AND 2013 
          AND fund.country_code = 'USA'
    GROUP BY EXTRACT(MONTH FROM funding_round.funded_at)),

acquired AS
    (SELECT EXTRACT(MONTH FROM acquisition.acquired_at) AS month,
            COUNT(acquisition.acquired_company_id) AS count_of_acquired,
            SUM(acquisition.price_amount) AS sum_of_acquired
    FROM acquisition
    WHERE EXTRACT(YEAR FROM acquisition.acquired_at) BETWEEN 2010 AND 2013
    GROUP BY EXTRACT(MONTH FROM acquisition.acquired_at))

SELECT fund.month, fund.count_of_fund, acquired.count_of_acquired, acquired.sum_of_acquired
FROM fund
JOIN acquired ON fund.month = acquired.month


WITH education AS
(SELECT person_id,
       COUNT(instituition)
FROM education
WHERE person_id IN
                   (SELECT DISTINCT id
                   FROM people
                   WHERE company_id in 
                                       (SELECT DISTINCT id
                                        FROM company
                                        WHERE name = 'Facebook'))
GROUP BY education.person_id)

SELECT AVG(count)
FROM education


SELECT DISTINCT people.id,
       education.instituition
FROM company
JOIN funding_round ON funding_round.company_id = company.id
JOIN people ON people.company_id = company.id
JOIN education ON education.person_id = people.id
WHERE status = 'closed'
      AND is_first_round = 1
      AND is_last_round = 1
      
      
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM founded_at::date) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) > 0
ORDER BY avg DESC, country_code ASC
LIMIT 10


WITH funding_round AS
(SELECT funded_at::date,
       MIN(raised_amount),
       MAX(raised_amount)
FROM funding_round
GROUP BY funded_at::date)

SELECT *
FROM funding_round
WHERE min != 0
      AND min != max
```

```
funded_at	min	max
2012-08-22	40000	7.5e+07
2010-07-25	3.27825e+06	9e+06
2002-03-01	2.84418e+06	8.95915e+06
2010-10-11	28000	2e+08
2007-01-18	5.5e+06	2.3e+07
2007-02-27	1.29e+06	3.6e+07
2006-01-05	8.9e+06	2.65e+07
2011-10-31	35000	2.5e+07
2012-10-27	500000	9.3e+06
2007-08-16	2.51989e+06	9e+06
2013-09-10	50000	4.48e+08
2010-03-14	600000	3.5e+06
2006-01-13	4.5e+06	1.3e+07
2010-04-20	40000	3.1496e+08
2007-07-12	606672	4.13e+07
2005-12-12	192000	9.5e+06
2008-06-24	124000	3.8e+07
2008-03-09	1.99e+06	9.5e+06
2005-08-16	1.61e+06	2.5e+07
2008-05-29	3e+06	2e+07
2012-03-10	40000	1.068e+06
2007-06-14	811544	9.847e+07
2009-06-22	400000	5.2575e+08
2012-09-08	33102	1.2e+07
2006-12-18	1e+06	2e+07
2012-10-13	725000	4.4e+06
2000-04-01	1.01e+06	1.4e+07
2010-11-30	25996	7.7e+08
2008-05-26	2e+06	7.6e+06
2009-10-22	76000	1.2e+07
2013-01-06	147372	390360
2005-04-26	1.3e+06	2e+07
2005-11-10	1.49e+06	3.18e+07
2007-01-19	427000	5.33e+06
2012-09-09	10000	640000
2013-01-25	500000	3.29006e+07
2007-05-07	200000	2e+07
2005-05-04	2e+06	1.2e+07
2011-11-30	50000	3e+07
2009-06-25	575002	1.5e+07
2013-05-25	120000	1.25e+08
2010-05-14	40000	3.5e+07
2010-02-19	45000	4.5e+07
2000-02-14	35000	2e+06
2009-08-24	1e+06	4.5e+07
2007-02-11	1e+06	2.2e+07
2006-07-19	1.5e+06	1e+07
2005-06-10	500000	1.2e+07
2007-02-14	810000	2.5e+07
2006-12-15	350000	1.3106e+06
```
