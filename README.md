Mobile Manufacturer Analysis
This repository contains a collection of SQL queries designed to analyze data from a mobile phone sales database. The dataset includes information about customers, transactions, cellphones, and manufacturers across various states and years. The queries in this repository address various business questions and perform in-depth analysis on mobile sales trends.

Features
List all states in which we have customers who have bought cellphones from 2005 to today.
Identify the state in the US with the highest number of 'Samsung' cellphone purchases.
Show the number of transactions for each model per zip code and per state.
Find the cheapest cellphone and display its price.
Calculate the average price for each model in the top 5 manufacturers by sales quantity, ordered by average price.
List customer names and their average spending in 2009, where the average is higher than $500.
Identify if there is any model that was in the top 5 in terms of quantity in 2008, 2009, and 2010.
Find the manufacturer with the 2nd highest sales in 2009 and the 2nd highest sales in 2010.
Show manufacturers that sold cellphones in 2010 but did not sell any in 2009.
Find the top 100 customers, their average spend, average quantity purchased by year, and the percentage change in their spending.
Requirements
Database: SQL-based database (e.g., MySQL, PostgreSQL, etc.)
Tables:
customers
transactions
cellphones
manufacturers
states
Ensure the database schema supports the necessary relationships between these entities, such as:
A customer_id linking customers to transactions
A model_id linking cellphones to transactions
A manufacturer_id linking cellphones to manufacturers
Transaction details including quantity and price
Example Queries
1. List all states with customers who bought cellphones from 2005 to today:

SELECT DISTINCT state
FROM customers
JOIN transactions ON customers.customer_id = transactions.customer_id
JOIN cellphones ON transactions.model_id = cellphones.model_id
WHERE transactions.date >= '2005-01-01';

2. State in the US buying the most 'Samsung' cellphones:

SELECT state, COUNT(*) AS samsung_sales
FROM transactions
JOIN cellphones ON transactions.model_id = cellphones.model_id
WHERE cellphones.manufacturer = 'Samsung'
GROUP BY state
ORDER BY samsung_sales DESC
LIMIT 1;

3. Number of transactions for each model per zip code per state:

SELECT state, zip_code, model, COUNT(*) AS num_transactions
FROM transactions
JOIN customers ON transactions.customer_id = customers.customer_id
JOIN cellphones ON transactions.model_id = cellphones.model_id
GROUP BY state, zip_code, model;

4. Cheapest cellphone:

SELECT model, MIN(price) AS cheapest_price
FROM cellphones
GROUP BY model
ORDER BY cheapest_price ASC
LIMIT 1;
5. Average price for each model in the top 5 manufacturers by sales quantity:

WITH top_manufacturers AS (
  SELECT manufacturer_id
  FROM transactions
  JOIN cellphones ON transactions.model_id = cellphones.model_id
  GROUP BY manufacturer_id
  ORDER BY COUNT(*) DESC
  LIMIT 5
)
SELECT cellphones.model, AVG(cellphones.price) AS avg_price
FROM cellphones
WHERE cellphones.manufacturer_id IN (SELECT manufacturer_id FROM top_manufacturers)
GROUP BY cellphones.model
ORDER BY avg_price;

6. List customers and their average spending in 2009 where the average is higher than $500:

SELECT customers.name, AVG(transactions.amount) AS avg_spend
FROM customers
JOIN transactions ON customers.customer_id = transactions.customer_id
WHERE YEAR(transactions.date) = 2009
GROUP BY customers.name
HAVING avg_spend > 500;

7. Find if any model was in the top 5 in 2008, 2009, and 2010:

WITH top_models_2008 AS (
  SELECT model_id
  FROM transactions
  WHERE YEAR(date) = 2008
  GROUP BY model_id
  ORDER BY COUNT(*) DESC
  LIMIT 5
),
top_models_2009 AS (
  SELECT model_id
  FROM transactions
  WHERE YEAR(date) = 2009
  GROUP BY model_id
  ORDER BY COUNT(*) DESC
  LIMIT 5
),
top_models_2010 AS (
  SELECT model_id
  FROM transactions
  WHERE YEAR(date) = 2010
  GROUP BY model_id
  ORDER BY COUNT(*) DESC
  LIMIT 5
)
SELECT model_id
FROM top_models_2008
INTERSECT
SELECT model_id
FROM top_models_2009
INTERSECT
SELECT model_id
FROM top_models_2010;

8. Manufacturer with the 2nd highest sales in 2009 and 2010:

SELECT manufacturer_id, SUM(transactions.amount) AS total_sales
FROM transactions
JOIN cellphones ON transactions.model_id = cellphones.model_id
WHERE YEAR(transactions.date) = 2009
GROUP BY manufacturer_id
ORDER BY total_sales DESC
LIMIT 1 OFFSET 1;

9. Manufacturers that sold in 2010 but not in 2009:

SELECT DISTINCT manufacturer_id
FROM transactions
JOIN cellphones ON transactions.model_id = cellphones.model_id
WHERE YEAR(transactions.date) = 2010
AND manufacturer_id NOT IN (
    SELECT DISTINCT manufacturer_id
    FROM transactions
    JOIN cellphones ON transactions.model_id = cellphones.model_id
    WHERE YEAR(transactions.date) = 2009
);

10. Top 100 customers and their average spend, average quantity by year, and percentage change in spending:

WITH customer_stats AS (
  SELECT customer_id, YEAR(date) AS year, AVG(amount) AS avg_spend, AVG(quantity) AS avg_quantity
  FROM transactions
  GROUP BY customer_id, year
)
SELECT customer_id, year, avg_spend, avg_quantity,
       (avg_spend - LAG(avg_spend) OVER (PARTITION BY customer_id ORDER BY year)) / LAG(avg_spend) OVER (PARTITION BY customer_id ORDER BY year) * 100 AS percent_change
FROM customer_stats
ORDER BY avg_spend DESC
LIMIT 100;

Import the SQL queries into your SQL-based database environment.

Run the individual queries to retrieve data based on the questions outlined above.

Contributing
Feel free to submit pull requests if you have improvements or additional queries. If you have any specific questions or need help with any part of the code, please open an issue.

License
This project is licensed under the MIT License - see the LICENSE file for details.
