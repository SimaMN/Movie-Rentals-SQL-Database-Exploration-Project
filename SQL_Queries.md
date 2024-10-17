--Query 1 - query used for first insight 
```sql

SELECT 
    f.title, 
    c.name AS category, 
    COUNT(*) AS rental_count
FROM 
    category c
JOIN 
    film_category fc ON c.category_id = fc.category_id
JOIN 
    film f ON fc.film_id = f.film_id
JOIN 
    inventory i ON i.film_id = f.film_id
JOIN 
    rental r ON i.inventory_id = r.inventory_id
WHERE 
    c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
GROUP BY 
1, 2
ORDER BY 
2, 1;

--Query 2 - query used for second insight 

SELECT 
    f.title, 
    c.name AS category, 
    f.rental_duration,
    NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
FROM 
    category c
JOIN 
    film_category fc ON c.category_id = fc.category_id
JOIN 
    film f ON fc.film_id = f.film_id
WHERE 
    c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music');

-- Query 3 - query used for third insight

SELECT category, standard_quartile,
COUNT(*)
FROM
    (SELECT c.name category, f.rental_duration,
     NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile
     FROM category c
     JOIN film_category fc
     ON c.category_id = fc.category_id
     JOIN film f
     ON fc.film_id = f.film_id
     WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')) t1
GROUP BY category, standard_quartile
ORDER BY category, standard_quartile


-- Query 4 - query used for fourth insight

SELECT  
    DATE_PART('year', r.rental_date) AS year, 
    DATE_PART('month', r.rental_date) AS month, 
    store.store_id, 
    COUNT(*) AS count_rentals
FROM 
    store
JOIN 
    staff ON store.store_id = staff.store_id
JOIN 
    rental r ON r.staff_id = staff.staff_id
GROUP BY 
    year, 
    month, 
    store.store_id
ORDER BY 
    count_rentals DESC;


-- Query 5 - query used for fifth insight

WITH top10 AS (
    SELECT 
        c.customer_id, 
        SUM(p.amount) AS total_payments
    FROM 
        customer c
    JOIN 
        payment p ON p.customer_id = c.customer_id
    GROUP BY 
        c.customer_id
    ORDER BY 
        total_payments DESC
    LIMIT 10
)
SELECT 
    DATE_TRUNC('month', p.payment_date) AS pay_mon, 
    (c.first_name || ' ' || c.last_name) AS full_name, 
    COUNT(p.amount) AS pay_count_per_mon, 
    SUM(p.amount) AS pay_amount
FROM 
    top10
JOIN 
    customer c ON top10.customer_id = c.customer_id
JOIN 
    payment p ON p.customer_id = c.customer_id
WHERE 
    p.payment_date >= '2007-01-01' AND 
    p.payment_date < '2008-01-01'
GROUP BY 
    pay_mon, 
    full_name
ORDER BY 
    full_name, 
    pay_mon;


-- Query 6 - query used for six insight 
WITH top10 AS (
    SELECT 
        c.customer_id, 
        SUM(p.amount) AS total_payments
    FROM 
        customer c
    JOIN 
        payment p ON p.customer_id = c.customer_id
    GROUP BY 
        c.customer_id
    ORDER BY 
        total_payments DESC
    LIMIT 10
),

t2 AS (
    SELECT 
        DATE_TRUNC('month', p.payment_date) AS pay_mon, 
        (c.first_name || ' ' || c.last_name) AS full_name, 
        SUM(p.amount) AS pay_amount
    FROM 
        top10
    JOIN 
        customer c ON top10.customer_id = c.customer_id
    JOIN 
        payment p ON p.customer_id = c.customer_id
    WHERE 
        p.payment_date >= '2007-01-01' AND 
        p.payment_date < '2008-01-01'
    GROUP BY 
        pay_mon, 
        full_name
)

SELECT 
    *, 
    LAG(t2.pay_amount) OVER (PARTITION BY full_name ORDER BY pay_mon) AS lag_amount, 
    (pay_amount - COALESCE(LAG(t2.pay_amount) OVER (PARTITION BY full_name ORDER BY pay_mon), 0)) AS diff
FROM 
    t2
ORDER BY 
    diff DESC;


