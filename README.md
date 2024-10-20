# Movie Rentals SQL Database Exploration Project

## Overview

This project involves exploring the Sakila DVD Rental database using SQL. The Sakila Database holds information about a company that rents movie DVDs. The goal is to query the database to gain insights into the customer base, such as rental patterns across different customer groups, payment earnings, and store performance.

You will write SQL queries, run them to obtain data, and create visualizations to answer specific questions related to the database. The project culminates in a presentation containing questions of interest, the SQL queries used, visualizations of the results, and brief explanations.

## Project Submission Requirements

Your submission should include:
- A set of **four slides**, each containing:
  - A question of interest.
  - A SQL query answering the question.
  - A visualization of the query's output.
  - A brief summary explaining the results.
- A text file containing the SQL queries used to generate the visualizations.

## SQL Query Examples

The following are examples of SQL queries used in the project, along with explanations of the techniques used.

### Example Query 1: Monthly Rental Orders by Store

This query compares the number of rental orders between two stores, grouped by year and month. It aggregates the count of rental orders per store for each month across the available years of data.

```sql
SELECT 
  YEAR(rental_date) AS year, 
  MONTH(rental_date) AS month, 
  store.store_id, 
  COUNT(rental.rental_id) AS rental_orders
FROM rental
JOIN inventory ON rental.inventory_id = inventory.inventory_id
JOIN store ON inventory.store_id = store.store_id
GROUP BY year, month, store.store_id
ORDER BY year, month, rental_orders DESC;
```

### Example Query 2: Top 10 Paying Customers in 2007

This query identifies the top 10 paying customers and calculates the total amount they paid per month in 2007. The query uses a subquery to limit the result to the top 10 customers and concatenates customer first and last names.

```sql
SELECT 
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    MONTH(p.payment_date) AS month,
    YEAR(p.payment_date) AS year,
    SUM(p.amount) AS total_payment
FROM 
    payment p
JOIN 
    customer c ON p.customer_id = c.customer_id
WHERE 
    YEAR(p.payment_date) = 2007
GROUP BY 
    customer_name, month, year
ORDER BY 
    total_payment DESC
LIMIT 10;
```

### Example Query 3: Comparing Monthly Payments for Top Customers

This query calculates the difference in monthly payments for the top 10 customers across 2007, helping to identify customers with the highest variation in payment amounts. A window function calculates the difference between consecutive months.

```sql
WITH customer_payments AS (
  SELECT 
    customer_id, 
    MONTH(payment_date) AS month, 
    SUM(amount) AS monthly_total
  FROM payment
  WHERE YEAR(payment_date) = 2007
  GROUP BY customer_id, month
)
SELECT 
  customer_id, 
  month, 
  monthly_total, 
  LEAD(monthly_total) OVER (PARTITION BY customer_id ORDER BY month) - monthly_total AS payment_difference
FROM customer_payments;
```

### Example Query 4: Total Inventory by Film

This query finds the total inventory of each film by counting the number of available copies in the inventory table, grouped by film ID.

```sql
SELECT 
  film_id, 
  COUNT(inventory_id) AS inventory_count
FROM inventory
GROUP BY film_id
ORDER BY inventory_count DESC;
```

## Instructions for Visualizing Data
To create visualizations based on the query outputs:

1. Run the SQL queries and export the result as a CSV file.
2. Import the CSV file into a spreadsheet tool like Excel or Google Sheets.
3. Use the tool to create visualizations such as bar charts, line graphs, or tables.
4. Include these visualizations in the presentation slides along with the corresponding SQL query and a brief explanation.

## Additional Guidelines

1) **Use of Joins**: Joins should be between the primary key and foreign key. For example:

```sql
ON inventory.inventory_id = rental.inventory_id
```

2) **Aggregations**: Be cautious when using aggregations. Every column in the SELECT clause, apart from the aggregated values, must be included in the GROUP BY clause.

```sql
SELECT film_id, COUNT(*)
FROM inventory
GROUP BY film_id;
```

3) **Window Functions**: At least one query should use a window function to calculate an aggregation over a subset of rows. In the provided example query, the LEAD function is used to calculate the difference between consecutive rows.

4) **Subqueries**: Use subqueries only when necessary to refine the data being returned.

## Conclusion

This project helps you explore a movie rental database using SQL, applying key techniques like joins, aggregations, subqueries, and window functions. By querying the database, you'll gain valuable insights and use visualizations to showcase your findings.

## Tips

Ensure you follow the submission guidelines closely, and remember that each query should include at least one join and aggregation. Additionally, use subqueries or common table expressions (CTEs) where necessary, and make sure one of your visualizations includes a column generated by a window function.

## License
This project was done as part of the Udacity SQL Nano degree program.
