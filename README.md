# Exploring DVD Rental Dataset Using SQL

### Project Overview

This project involved investigating the Salike Movie database provided by Udacity in the Learn Data Science nanodegree course. We wrote queries that answered questions about business decisions and then visualized our findings.
![image](https://github.com/Tascha98/Exploring-DVD-Rental-Dataset-Using-SQL/assets/135465320/14afacbf-8c3d-477f-bb13-d3e29913d7fa)


### Tools
-SQL server

-Excel for graphs

-Powerpoint for presentation


### Data Analysis
```sql
#Q1. Rank Top 10 actors by the number of films they have appeared in and their prefered category of film (ie the most appearance in such category)

WITH ActorFilmCounts AS (
    SELECT
        a.actor_id,
        CONCAT(a.first_name, ' ', a.last_name) AS actor_name,
        COUNT(fa.film_id) AS film_count,
        RANK() OVER (ORDER BY COUNT(fa.film_id) DESC) AS actor_rank
    FROM
        actor a
    JOIN
        film_actor fa ON a.actor_id = fa.actor_id
    GROUP BY
        a.actor_id,
        actor_name
  LIMIT 10
  
)

SELECT
    afc.actor_name,
    afc.film_count,
    afc.actor_rank,
    (
        SELECT
            c.name
        FROM
            film_category fc
        JOIN
            category c ON fc.category_id = c.category_id
        WHERE
            fc.film_id IN (
                SELECT
                    fa.film_id
                FROM
                    film_actor fa
                WHERE
                    fa.actor_id = afc.actor_id
            )
        GROUP BY
            c.name
        ORDER BY
            COUNT(*) DESC
        LIMIT 1
    ) AS preferred_category
FROM
    ActorFilmCounts afc
ORDER BY
    actor_rank;

#Q2. The number of rental orders each store has by every month for all the years in the dataset


WITH t1 AS (SELECT r.rental_id,r.rental_date,i.store_id,DATE_PART('month',rental_date) AS month, DATE_PART('year',rental_date) AS year 
FROM rental r 
JOIN inventory i 
ON r.inventory_id = i.inventory_id)

SELECT t1.month, t1.year, t1.store_id, COUNT(t1.rental_id) 
FROM t1 
GROUP BY 1,2,3 
ORDER BY 4 DESC

#Q3. We would like to know who were our top 10 paying customers in terms of total amount paid, how many payments they made on a monthly basis during 2007

 WITH t1 AS (SELECT DATE_TRUNC('month', p.payment_date) AS pay_mon, CONCAT(c.first_name, ' ', c.last_name) AS fullname, SUM(p.amount)AS pay_amount,COUNT(p.amount) AS pay_countpermonth

 FROM payment p 

JOIN customer c 

ON p.customer_id = c.customer_id 
GROUP BY 1, 2 
ORDER BY 2,1 ),

t2 AS (SELECT t1.pay_mon, t1.fullname,t1.pay_amount,t1.pay_countpermonth, SUM(t1.pay_amount) OVER (PARTITION BY t1.fullname)AS total_amount
 FROM t1),

t3 AS (SELECT t2.pay_mon, t2.fullname, t2.pay_amount, t2.pay_countpermonth, t2.total_amount, DENSE_RANK() OVER (ORDER BY t2.total_amount DESC) AS customer_rank 
FROM t2)

SELECT t3.pay_mon, t3.fullname, t3.pay_countpermonth 
FROM t3 
WHERE t3.customer_rank BETWEEN 1 AND 10 
ORDER BY t3.fullname, t3.pay_mon

#Q4. What is the total rental count of movies for each category?

WITH CategoryRentalCounts AS (
    SELECT
        c.name AS category,
        COUNT(r.rental_id) AS total_rentals
    FROM category c
    JOIN  film_category fc 

    ON c.category_id = fc.category_id
    JOIN film f 
    ON f.film_id = fc.film_id
    JOIN inventory i 
    ON f.film_id = i.film_id
    JOIN rental r 
    ON r.inventory_id = i.inventory_id
    GROUP BY c.name
)

SELECT
    category,
    total_rentals

FROM  CategoryRentalCounts

ORDER BY category;


```






