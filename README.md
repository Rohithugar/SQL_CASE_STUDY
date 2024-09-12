# CASE_STUDY-1
## Bike rental shop - SQL Case study

## Introduction:
Emily is the shop owner, and she would like to gather data to help her grow the
business. She has hired you as an SQL specialist to get the answers to her
business questions such as How many bikes does the shop own by category?
What was the rental revenue for each month? etc. The answers are hidden in the
database. You just need to figure out how to get them out using SQL.

## Understanding the database:
The shop's database consists of 5 tables:
customer
bike
rental
membership_type
membership

## Problem Statements

1. Emily would like to know how many bikes the shop owns by category. Can
you get this for her? 
Display the category name and the number of bikes the shop owns in
each category (call this column number_of_bikes ). Show only the categories
where the number of bikes is greater than 2 .
``` sql
SELECT category, COUNT(id) AS number_of_bikes
FROM bike
GROUP BY category
HAVING COUNT(id) > 2;
``` 

2. Emily needs a list of customer names with the total number of memberships purchased by each.
For each customer, display the customer's name and the count of
memberships purchased (call this column membership_count ). Sort the
results by membership_count , starting with the customer who has purchased
the highest number of memberships.
``` sql
SELECT c.name, COUNT(m.id) AS membership_count 
FROM membership m
RIGHT JOIN customer c ON c.id=m.customer_id
GROUP BY c.name
ORDER BY COUNT(m.id) DESC;
``` 
3. Emily is working on a special offer for the winter months. Can you help her prepare a list of new rental prices?
For each bike, display its ID, category, old price per hour (call this column 
old_price_per_hour ), discounted price per hour (call it new_price_per_hour ), old
price per day (call it old_price_per_day ), and discounted price per day (call it new_price_per_day ).
Electric bikes should have a 10% discount for hourly rentals and a 20%
discount for daily rentals. Mountain bikes should have a 20% discount for
hourly rentals and a 50% discount for daily rentals. All other bikes should
have a 50% discount for all types of rentals. Round the new prices to 2 decimal digits.
``` sql
SELECT id, category, price_per_hour AS old_price_per_hour,
	CASE 
		WHEN category = 'electric' THEN ROUND(price_per_hour - (price_per_hour*0.1) ,2)
		WHEN category = 'mountain bike' THEN ROUND(price_per_hour - (price_per_hour*0.2) ,2)
		ELSE ROUND(price_per_hour - (price_per_hour*0.5) ,2)
	END AS new_price_per_hour, price_per_day AS old_price_per_day,
	CASE 
		WHEN category = 'electric' THEN ROUND(price_per_day - (price_per_day*0.2) ,2)
		WHEN category = 'mountain bike' THEN ROUND(price_per_day - (price_per_day*0.5) ,2)
		ELSE ROUND(price_per_day - (price_per_day*0.5) ,2)
	END AS new_price_per_day
FROM bike;
```
4. Emily is looking for counts of the rented bikes and of the available bikes in each category.
Display the number of available bikes (call this column 
available_bikes_count ) and the number of rented bikes (call this column rented_bikes_count ) by bike category.
``` sql
SELECT category,
COUNT(CASE WHEN status ='available' THEN 1 END) AS available_bikes_count,
COUNT(CASE WHEN status ='rented' THEN 1 END) AS rented_bikes_count
FROM bike
GROUP BY category;
``` 
5. Emily is preparing a sales report. She needs to know the total revenue
from rentals by month, the total by year, and the all-time across all the years. 
Bike rental shop Display the total revenue from rentals for each month, the total for each
year, and the total across all the years. Do not take memberships into account. There should be 3 columns: year , month , and revenue .
Sort the results chronologically. Display the year total after all the month totals for the corresponding year. Show the all-time total as the last row.
``` sql
SELECT 
    YEAR(start_timestamp) AS year,
    MONTH(start_timestamp) AS month,
    SUM(total_paid) AS revenue
FROM rental
GROUP BY YEAR(start_timestamp), MONTH(start_timestamp)
UNION ALL
SELECT 
    YEAR(start_timestamp) AS year,
    NULL AS month,
    SUM(total_paid) AS revenue
FROM rental
GROUP BY YEAR(start_timestamp)
UNION ALL
SELECT 
    NULL AS year,
    NULL AS month,
    SUM(total_paid) AS revenue
FROM rental;
``` 
6. Emily has asked you to get the total revenue from memberships for each combination of year, month, and membership type.
Display the year, the month, the name of the membership type (call this column membership_type_name ), and the total revenue (call this column 
total_revenue ) for every combination of year, month, and membership type. Sort the results by year, month, and name of membership type.
``` sql
SELECT
    YEAR(start_date) AS year,
    MONTH(start_date) AS month,
    mt.name AS membership_type_name,
    SUM(total_paid) AS total_revenue
FROM membership m
JOIN membership_type mt ON m.membership_type_id = mt.id
GROUP BY YEAR(start_date), MONTH(start_date), mt.name
ORDER BY year, month, mt.name;
```
7. Next, Emily would like data about memberships purchased in 2023, with
subtotals and grand totals for all the different combinations of membership types and months.
Display the total revenue from memberships purchased in 2023 for each combination of month and membership type. Generate subtotals and
grand totals for all possible combinations. There should be 3 columns:  membership_type_name , month , and total_revenue .
Sort the results by membership type name alphabetically and then  chronologically by month.
``` sql
SELECT
  mt.name AS membership_type_name,
  month(start_date) AS month,
  SUM(CASE WHEN month(start_date) THEN total_paid ELSE 0 END) AS total_revenue
FROM membership m
JOIN membership_type mt ON m.membership_type_id = mt.id
WHERE YEAR(start_date) = 2023
GROUP BY membership_type_name, month(start_date)
WITH ROLLUP
ORDER BY membership_type_name, month(start_date);
```
8. Emily wants to segment customers based on the number of rentals and see the count of customers in each segment. Use your SQL skills to get this!
Categorize customers based on their rental history as follows: 
Customers who have had more than 10 rentals are categorized as 'more than 10' .
Customers who have had 5 to 10 rentals (inclusive) are categorized as 'between 5 and 10' .
Customers who have had fewer than 5 rentals should be categorized as 'fewer than 5' .
Calculate the number of customers in each category. Display two columns: 
rental_count_category (the rental count category) and customer_count (the number of customers in each category).
``` sql
WITH cte AS 
	(SELECT customer_id, COUNT(1),
		CASE WHEN COUNT(1) > 10 THEN 'more than 10' 
		WHEN COUNT(1) BETWEEN 5 AND 10 THEN 'between 5 and 10'
		ELSE 'fewer than 5'
	END AS category
	FROM rental
	GROUP BY customer_id)
SELECT category AS rental_count_category,
COUNT(*) AS customer_count
FROM cte 
GROUP BY category
ORDER BY customer_count;
```
