# Zomato SQL Project.

create table customers
(
	customer_id int not null,
	customer_name varchar(25),
	reg_date date,
	primary key(customer_id)

create table restaurants
(
	restaurant_id int not null,
	restaurant_name varchar(50),
	city varchar(20),
	opening_hours varchar(60),
	primary key(restaurant_id)
);
drop table restaurants;

create table orders
(
	order_id int primary key,
	customer_id int,
	restaurant_id int,
	order_item varchar(60),
	order_date date,
	order_time time,
	order_status varchar(50),
	total_amount Float
);

create table riders
(
	rider_id int primary key,
	rider_name varchar(50),
	sign_up date
);

drop table deliveries;
create table deliveries
(
	delivery_id int,
	order_id int,
	delivery_status varchar(35),
	delivery_time time,
	rider_id int,
	constraint fk_orders foreign key(order_id) 
	references orders(order_id),
	constraint fk_rider foreign key(rider_id)
	references riders(rider_id)
);

alter table orders
add constraint fk_customer
foreign key (customer_id)
references customers(customer_id);

alter table orders
add constraint fk_restaurants
foreign key (restaurant_id)
references restaurants(restaurant_id);

select *
from customers;

select *
from deliveries;
select *
from orders;
select *
from restaurants;
select *
from riders;

select *
from deliveries
where delivery_time is NULL;

--Business problems

**--Write a query to find the top 5 most frequently ordered dishes by the customer "Arjun Mehta" in
--the last 1 year.**

SELECT
	CUSTOMER_NAME,
	ORDER_ITEM,
	TOTAL_ORDERS
FROM
	(
		SELECT
			C.CUSTOMER_ID,
			C.CUSTOMER_NAME,
			O.ORDER_ITEM,
			COUNT(*) AS TOTAL_ORDERS,
			DENSE_RANK() OVER (
				ORDER BY
					COUNT(*) DESC
			) AS RANK
		FROM
			ORDERS AS O
			JOIN CUSTOMERS AS C ON C.CUSTOMER_ID = O.CUSTOMER_ID
		WHERE
			O.ORDER_DATE >= CURRENT_DATE - INTERVAL '1.4 Year'
			AND CUSTOMER_NAME = 'Arjun Mehta'
		GROUP BY
			1,
			2,
			3
		ORDER BY
			1,
			4 DESC
	) AS T1
WHERE
	RANK <= 5



****--Identify the time slots during which the most orders are placed, based on 2-hour intervals.**
**Solution 1.**
SELECT
	FLOOR(
		EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) / 2
	) * 2 AS START_TIME,
	FLOOR(
		EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) / 2
	) * 2 + 2 AS END_TIME,
	COUNT(*) AS TOTAL_ORDER
FROM
	ORDERS
GROUP BY
	1,
	2
ORDER BY
	3 DESC

**Solution 2.**
SELECT
	CASE
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 0 AND 1  THEN '00:00 - 02:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 2 AND 3  THEN '02:00 - 04:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 4 AND 5  THEN '04:00 - 06:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 6 AND 7  THEN '06:00 - 08:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 8 AND 9  THEN '08:00 - 10:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 10 AND 11  THEN '10:00 - 12:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 12 AND 13  THEN '12:00 - 14:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 14 AND 15  THEN '14:00 - 16:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 16 AND 17  THEN '16:00 - 18:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 18 AND 19  THEN '18:00 - 20:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 20 AND 21  THEN '20:00 - 22:00'
		WHEN EXTRACT(
			HOUR
			FROM
				ORDER_TIME
		) BETWEEN 22 AND 23  THEN '22:00 - 24:00'
	END AS TIME_SLOT,
	COUNT(ORDER_ID) AS COUNT
FROM
	ORDERS
GROUP BY
	TIME_SLOT
ORDER BY
	COUNT DESC;

**--Find the average order value (AOV) per customer who has placed more than 750 orders.
--Return: customer_name, aov (average order value).**

SELECT
	*
FROM
	(
		SELECT
			C.CUSTOMER_ID,
			C.CUSTOMER_NAME,
			AVG(O.TOTAL_AMOUNT) AS AVG_PURCHARSE,
			COUNT(O.ORDER_ID) AS TOTAL
		FROM
			CUSTOMERS AS C
			JOIN ORDERS AS O ON C.CUSTOMER_ID = O.CUSTOMER_ID
		GROUP BY
			1,
			2
		ORDER BY
			4 DESC
	) AS T1
WHERE
	TOTAL > 750;

SELECT
	*
FROM


**--List the customers who have spent more than 100K in total on food orders.
--Return: customer_name, customer_id.**

SELECT
	C.CUSTOMER_ID,
	C.CUSTOMER_NAME,
	SUM(O.TOTAL_AMOUNT) AS TOTAL
FROM
	CUSTOMERS AS C
	JOIN ORDERS AS O ON C.CUSTOMER_ID = O.CUSTOMER_ID
GROUP BY
	1,
	2
HAVING
	SUM(O.TOTAL_AMOUNT) > 100000;

**--Write a query to find orders that were placed but not delivered.
--Return: restaurant_name, city, and the number of not delivered orders.**
**Slolution 1**
SELECT
	R.RESTAURANT_NAME,
	R.CITY,
	O.ORDER_STATUS,
	COUNT(O.ORDER_ID) AS TOTAL
FROM
	RESTAURANTS AS R
	JOIN ORDERS AS O ON R.RESTAURANT_ID = O.RESTAURANT_ID
GROUP BY
	1,
	2,
	3
HAVING
	O.ORDER_STATUS = 'Not Fulfilled'
ORDER BY
	R.RESTAURANT_NAME DESC;
**Solution 2**
SELECT
	R.RESTAURANT_NAME,
	R.CITY,
	COUNT(O.ORDER_ID) AS NOT_DELIVERED
FROM
	ORDERS AS O
	LEFT JOIN RESTAURANTS AS R ON R.RESTAURANT_ID = O.RESTAURANT_ID
	LEFT JOIN DELIVERIES AS D ON D.ORDER_ID = O.ORDER_ID
WHERE
	D.DELIVERY_ID IS NULL
GROUP BY
	1,
	2
ORDER BY
	R.RESTAURANT_NAME DESC;


**--Rank restaurants by their total revenue from the last year.
--Return: restaurant_name, total_revenue, and their rank within their city.**

WITH
	RANKING_TABLE AS (
		SELECT
			R.CITY,
			R.RESTAURANT_NAME,
			SUM(O.TOTAL_AMOUNT) AS TOTAL_REVENUE,
			RANK() OVER (
				PARTITION BY
					R.CITY
				ORDER BY
					SUM(O.TOTAL_AMOUNT) DESC
			) AS RANK
		FROM
			ORDERS AS O
			JOIN RESTAURANTS AS R ON O.RESTAURANT_ID = R.RESTAURANT_ID
		WHERE
			O.ORDER_DATE = CURRENT_DATE - INTERVAL '1 Year'
		GROUP BY
			1,
			2
	)
SELECT
	*
FROM
	RANKING_TABLE
WHERE
	RANK = 1;
	
**--Identify the most popular dish in each city based on the number of orders.**

WITH
	MY_CTE AS (
		SELECT
			R.CITY,
			O.ORDER_ITEM,
			COUNT(O.ORDER_ITEM),
			RANK() OVER (
				PARTITION BY
					R.CITY
				ORDER BY
					COUNT(O.ORDER_ITEM) DESC
			) AS TOT
		FROM
			ORDERS AS O
			JOIN RESTAURANTS AS R ON O.RESTAURANT_ID = R.RESTAURANT_ID
		GROUP BY
			1,
			2
	)
SELECT
	*
FROM
	MY_CTE
WHERE
	TOT = 1;

**--Find customers who haven’t placed an order in 2024 but did in 2023.**

SELECT DISTINCT
	CUSTOMER_ID
FROM
	ORDERS
WHERE
	EXTRACT(
		YEAR
		FROM
			ORDER_DATE
	) = 2023
	AND CUSTOMER_ID NOT IN (
		SELECT DISTINCT
			CUSTOMER_ID
		FROM
			ORDERS
		WHERE
			EXTRACT(
				YEAR
				FROM
					ORDER_DATE
			) = 2024
	);SELECT DISTINCT
	CUSTOMER_ID
FROM
	ORDERS
WHERE
	EXTRACT(
		YEAR
		FROM
			ORDER_DATE
	) = 2023
	AND CUSTOMER_ID NOT IN (
		SELECT DISTINCT
			CUSTOMER_ID
		FROM
			ORDERS
		WHERE
			EXTRACT(
				YEAR
				FROM
					ORDER_DATE
			) = 2024
	);
**--Calculate and compare the order cancellation rate for each restaurant between the current year
--and the previous year.**

WITH
	CANCEL_RATIO_23 AS (
		SELECT
			O.RESTAURANT_ID,
			COUNT(O.ORDER_ID) AS TOTAL_ORDERS,
			COUNT(
				CASE
					WHEN D.DELIVERY_ID IS NULL THEN 1
				END
			) AS NOT_DELIVERED
		FROM
			ORDERS AS O
			LEFT JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
		WHERE
			EXTRACT(
				YEAR
				FROM
					O.ORDER_DATE
			) = 2023
		GROUP BY
			O.RESTAURANT_ID
	),
	CANCEL_RATIO_24 AS (
		SELECT
			O.RESTAURANT_ID,
			COUNT(O.ORDER_ID) AS TOTAL_ORDERS,
			COUNT(
				CASE
					WHEN D.DELIVERY_ID IS NULL THEN 1
				END
			) AS NOT_DELIVERED
		FROM
			ORDERS AS O
			LEFT JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
		WHERE
			EXTRACT(
				YEAR
				FROM
					O.ORDER_DATE
			) = 2024
		GROUP BY
			O.RESTAURANT_ID
	),
	LAST_YEAR_DATE AS (
		SELECT
			RESTAURANT_ID,
			TOTAL_ORDERS,
			NOT_DELIVERED,
			ROUND(
				(NOT_DELIVERED::NUMERIC / TOTAL_ORDERS::NUMERIC) * 100,
				2
			) AS CANCEL_RATIO
		FROM
			CANCEL_RATIO_23
	),
	CURRENT_YEAR_DATA AS (
		SELECT
			RESTAURANT_ID,
			TOTAL_ORDERS,
			NOT_DELIVERED,
			ROUND(
				(NOT_DELIVERED::NUMERIC / TOTAL_ORDERS::NUMERIC) * 100,
				2
			) AS CANCEL_RATIO
		FROM
			CANCEL_RATIO_24
	)
SELECT
	C.RESTAURANT_ID AS RESTAURANT_ID,
	C.CANCEL_RATIO AS CURRENT_YEAR_CANCEL_RATIO,
	L.CANCEL_RATIO AS LAST_YEAR_CANCEL_RATIO
FROM
	CURRENT_YEAR_DATA AS C
	JOIN LAST_YEAR_DATE AS L ON C.RESTAURANT_ID = L.RESTAURANT_ID;

**--Determine each rider's average delivery time.**

SELECT
	O.ORDER_ID,
	O.ORDER_TIME,
	D.DELIVERY_TIME,
	D.DELIVERY_TIME - O.ORDER_TIME AS TIME_DIFFERENCE,
	EXTRACT(
		EPOCH
		FROM
			(
				D.DELIVERY_TIME - O.ORDER_TIME + CASE
					WHEN D.DELIVERY_TIME < O.ORDER_TIME THEN INTERVAL '1 day'
					ELSE INTERVAL '0 day'
				END
			)
	) / 60 AS TIME_DIFF
FROM
	ORDERS AS O
	JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
WHERE
	D.DELIVERY_STATUS = 'Delivered'
GROUP BY
	1,
	2,
	3;

**--Calculate each restaurant's growth ratio based on the total number of delivered orders since its
--joining.**

SELECT
	O.RESTAURANT_ID,
	TO_CHAR(O.ORDER_DATE, 'mm-yy') AS MONTH,
	COUNT(O.ORDER_ID) AS ORDERS_TOTAL
FROM
	ORDERS AS O
	JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
WHERE
	D.DELIVERY_STATUS = 'Delivered'
GROUP BY
	1,
	2 ORDER BY
	1,
	2;


**--Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the
--average order value (AOV). If a customer's total spending exceeds the AOV, label them as
--'Gold'; otherwise, label them as 'Silver'.
--Return: The total number of orders and total revenue for each segment.**

SELECT
	CUSTOMER_CATEGORY,
	SUM(TOTAL),
	SUM(TOTAL_COUNT)
FROM
	(
		SELECT
			CUSTOMER_ID,
			SUM(TOTAL_AMOUNT) AS TOTAL,
			COUNT(ORDER_ID) AS TOTAL_COUNT,
			CASE
				WHEN SUM(TOTAL_AMOUNT) > (
					SELECT
						AVG(TOTAL_AMOUNT)
					FROM
						ORDERS
				) THEN 'Gold'
				ELSE 'Silver'
			END AS CUSTOMER_CATEGORY
		FROM
			ORDERS
		GROUP BY
			1
	) AS T1
GROUP BY
	1;

**--Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.**

SELECT
	D.RIDER_ID,
	TO_CHAR(O.ORDER_DATE, 'mm-yy') AS MONTH,
	SUM(O.TOTAL_AMOUNT) AS TOTAL_REVENUE,
	SUM(O.TOTAL_AMOUNT) * 0.08 AS RIDER_REVENUE
FROM
	ORDERS AS O
	JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
GROUP BY
	1,
	2
ORDER BY
	1,
	3 DESC;


**--Find the number of 5-star, 4-star, and 3-star ratings each rider has.
--Riders receive ratings based on delivery time:
--● 5-star: Delivered in less than 15 minutes
--● 4-star: Delivered between 15 and 20 minutes
--● 3-star: Delivered after 20 minutes**

SELECT
	RIDER_ID,
	STARS,
	COUNT(*) AS TOTAL_STARS
FROM
	(
		SELECT
			RIDER_ID,
			DELIVERY_TIME_TOOK,
			CASE
				WHEN DELIVERY_TIME_TOOK < 15 THEN '5 star'
				WHEN DELIVERY_TIME_TOOK BETWEEN 15 AND 20  THEN '4 star'
				ELSE '3 star'
			END AS STARS
		FROM
			(
				SELECT
					O.ORDER_ID,
					O.ORDER_TIME,
					D.DELIVERY_TIME,
					EXTRACT(
						EPOCH
						FROM
							(
								D.DELIVERY_TIME - O.ORDER_TIME + CASE
									WHEN D.DELIVERY_TIME < O.ORDER_TIME THEN INTERVAL '1 day'
									ELSE INTERVAL '0 day'
								END
							)
					) / 60 AS DELIVERY_TIME_TOOK,
					D.RIDER_ID
				FROM
					ORDERS AS O
					JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
				WHERE
					D.DELIVERY_STATUS = 'Delivered'
			) AS T1
	) AS T2
GROUP BY
	1,
	2
ORDER BY
	1,
	3 DESC;

**--Analyze order frequency per day of the week and identify the peak day for each restaurant.**

SELECT
	*
FROM
	(
		SELECT
			R.RESTAURANT_NAME,
			TO_CHAR(O.ORDER_DATE, 'Day') AS DAY,
			COUNT(O.ORDER_ID) AS TOTAL_ORDERS,
			RANK() OVER (
				PARTITION BY
					R.RESTAURANT_NAME
				ORDER BY
					COUNT(ORDER_ID) DESC
			) AS RANK_1
		FROM
			ORDERS AS O
			JOIN RESTAURANTS AS R ON O.RESTAURANT_ID = R.RESTAURANT_ID
		GROUP BY
			1,
			2
		ORDER BY
			1,
			4
	) AS T1
WHERE
	RANK_1 = 1;
**--Calculate the total revenue generated by each customer over all their orders.**
SELECT
	C.CUSTOMER_ID,
	C.CUSTOMER_NAME,
	SUM(O.TOTAL_AMOUNT)
FROM
	ORDERS AS O
	JOIN CUSTOMERS AS C ON O.CUSTOMER_ID = C.CUSTOMER_ID
GROUP BY
	1,
	2
ORDER BY3 DESC;
**--Identify sales trends by comparing each month's total sales to the previous month.**

SELECT
	EXTRACT(
		YEAR
		FROM
			ORDER_DATE
	) AS YEAR,
	EXTRACT(
		MONTH
		FROM
			ORDER_DATE
	) AS MONTH,
	SUM(TOTAL_AMOUNT) AS TOTAL_REVENUE,
	LAG(SUM(TOTAL_AMOUNT), 1) OVER (
		ORDER BY
			EXTRACT(
				YEAR
				FROM
					ORDER_DATE
			),
			EXTRACT(
				MONTH
				FROM
					ORDER_DATE
			)
	) AS PREVIOUS_MONTH
FROM
	ORDERS
GROUP BY
	1,
	2

**--Evaluate rider efficiency by determining average delivery times and identifying those with the
--lowest and highest**

WITH
	NEW_TABLE AS (
		SELECT
			D.RIDER_ID AS RIDER_ID,
			EXTRACT(
				EPOCH
				FROM
					(
						D.DELIVERY_TIME - O.ORDER_TIME + CASE
							WHEN D.DELIVERY_TIME < O.ORDER_TIME THEN INTERVAL '1 day'
							ELSE INTERVAL '0 day'
						END
					)
			) / 60 AS TIME_TAKEN
		FROM
			ORDERS AS O
			JOIN DELIVERIES AS D ON O.ORDER_ID = D.ORDER_ID
		WHERE
			D.DELIVERY_STATUS = 'Delivered'
	),
	RIDER_TIME AS (
		SELECT
			RIDER_ID,
			AVG(TIME_TAKEN) AS AVG_TIME
		FROM
			NEW_TABLE
		GROUP BY
			1
	)
SELECT
	MIN(AVG_TIME),
	MAX(AVG_TIME)
FROM
	RIDER_TIME

**--Track the popularity of specific order items over time and identify seasonal demand spikes**

SELECT
	ORDER_ITEM,
	SEASONS,
	COUNT(ORDER_ID) AS TOTAL
FROM
	(
		SELECT
			*,
			EXTRACT(
				MONTH
				FROM
					ORDER_DATE
			) AS MONTH,
			CASE
				WHEN EXTRACT(
					MONTH
					FROM
						ORDER_DATE
				) BETWEEN 4 AND 6  THEN 'spring'
				WHEN EXTRACT(
					MONTH
					FROM
						ORDER_DATE
				) > 6
				AND EXTRACT(
					MONTH
					FROM
						ORDER_DATE
				) < 9 THEN 'summer'
				ELSE 'winter'
			END AS SEASONS
		FROM
			ORDERS
	) AS T1
GROUP BY
	1,
	2


**--Rank each city based on the total revenue for the last year (2023).**

SELECT
	R.CITY,
	SUM(O.TOTAL_AMOUNT) AS TOTAL_REVENUE,
	RANK() OVER (
		ORDER BY
			SUM(O.TOTAL_AMOUNT) DESC
	) AS CITY_RANK
FROM
	ORDERS AS O
	JOIN RESTAURANTS AS R ON O.RESTAURANT_ID = O.RESTAURANT_ID
GROUP BY
	1;




















