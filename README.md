# Zomato SQL Project

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

select customer_name,order_item,total_orders
from
(select c.customer_id , c.customer_name , o.order_item,count(*) as total_orders,
DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS RANK
from orders as o
join customers as c
	on c.customer_id = o.customer_id
	where o.order_date >= CURRENT_DATE - INTERVAL '1.4 Year'
	and customer_name = 'Arjun Mehta'
	group by 1,2,3
	order by 1,4 DESC) as t1
	where rank <=5


****--Identify the time slots during which the most orders are placed, based on 2-hour intervals.**
**Solution 1.**
select **
floor(extract(hour from order_time)/2)* 2 as start_time,
floor(extract(hour from order_time)/2)* 2 + 2 as end_time,
count(*) as total_order
from orders
group by 1,2
order by 3 DESC

**Solution 2.**
select 
	case 
		when extract(hour from order_time) between 0 and 1 then '00:00 - 02:00'
		when extract(hour from order_time) between 2 and 3 then '02:00 - 04:00'
		when extract(hour from order_time) between 4 and 5 then '04:00 - 06:00'
		when extract(hour from order_time) between 6 and 7 then '06:00 - 08:00'
		when extract(hour from order_time) between 8 and 9 then '08:00 - 10:00'
		when extract(hour from order_time) between 10 and 11 then '10:00 - 12:00'
		when extract(hour from order_time) between 12 and 13 then '12:00 - 14:00'
		when extract(hour from order_time) between 14 and 15 then '14:00 - 16:00'
		when extract(hour from order_time) between 16 and 17 then '16:00 - 18:00'
		when extract(hour from order_time) between 18 and 19 then '18:00 - 20:00'
		when extract(hour from order_time) between 20 and 21 then '20:00 - 22:00'
		when extract(hour from order_time) between 22 and 23 then '22:00 - 24:00'
	end as time_slot,
count(order_id) as count
from orders
group by time_slot
order by count DESC;

**--Find the average order value (AOV) per customer who has placed more than 750 orders.
--Return: customer_name, aov (average order value).**

select *
from
(select c.customer_id,c.customer_name, avg(o.total_amount) as avg_purcharse,
count(o.order_id) as total
from customers as c
join orders as o
	on c.customer_id = o.customer_id
	group by 1,2
	order by 4 DESC) as t1
where total > 750;
select *
from


**--List the customers who have spent more than 100K in total on food orders.
--Return: customer_name, customer_id.**

select c.customer_id , c.customer_name,sum(o.total_amount) as total
from customers as c
join orders as o
	on c.customer_id = o.customer_id
	group by 1,2
	having sum(o.total_amount) > 100000;

**--Write a query to find orders that were placed but not delivered.
--Return: restaurant_name, city, and the number of not delivered orders.**
**Slolution 1**
select r.restaurant_name , r.city ,o.order_status, count(o.order_id) as total
from restaurants as r
join orders as o
	on r.restaurant_id = o.restaurant_id
	group by 1,2,3
	having o.order_status = 'Not Fulfilled'
	order by r.restaurant_name DESC;
**Solution 2**
select r.restaurant_name, r.city,count(o.order_id) as not_delivered
from orders as o
left join restaurants as r
	on r.restaurant_id = o.restaurant_id
left join deliveries as d
	on d.order_id = o.order_id
	where d.delivery_id is NULL
	group by 1,2
	order by r.restaurant_name DESC;


**--Rank restaurants by their total revenue from the last year.
--Return: restaurant_name, total_revenue, and their rank within their city.**

with ranking_table as
(select r.city,r.restaurant_name, sum(o.total_amount) as total_revenue,
RANK() over(partition by r.city order by sum(o.total_amount) DESC) as rank
from orders as o
join restaurants as r
	on o.restaurant_id = r.restaurant_id
	where o.order_date = CURRENT_DATE - INTERVAL '1 Year'
	group by 1,2
	)
select *
from ranking_table
where rank = 1;
	
**--Identify the most popular dish in each city based on the number of orders.**

with my_cte as
(select r.city , o.order_item , count(o.order_item),
rank() over (partition by r.city order by count(o.order_item) DESC ) as tot
from orders as o
join restaurants as r
	on o.restaurant_id = r.restaurant_id
	group by 1,2)
select *
from my_cte
where tot = 1;

**--Find customers who haven’t placed an order in 2024 but did in 2023.**

select distinct customer_id
from orders
where extract(year from order_date) = 2023
and customer_id not in(select distinct customer_id
					from orders
						where extract(year from order_date) = 2024);

**--Calculate and compare the order cancellation rate for each restaurant between the current year
--and the previous year.**

with cancel_ratio_23 as
(
	select o.restaurant_id,
	count(o.order_id) as total_orders,
	count(case when d.delivery_id is null then 1 end) as not_delivered
from orders as o
left join deliveries as d
	on o.order_id = d.order_id
	where extract(year from o.order_date) = 2023
	group by o.restaurant_id
),
cancel_ratio_24 as 
(
	select o.restaurant_id,
	count(o.order_id) as total_orders,
	count(case when d.delivery_id is null then 1 end) as not_delivered
from orders as o
left join deliveries as d
	on o.order_id = d.order_id
	where extract(year from o.order_date) = 2024
	group by o.restaurant_id
),
last_year_date as 
(
	select restaurant_id,
	total_orders,
	not_delivered,
	round((not_delivered::numeric / total_orders::numeric) * 100, 2) as cancel_ratio
	from cancel_ratio_23
),
current_year_data as
(
	select restaurant_id,
	total_orders,
	not_delivered,
	round((not_delivered::numeric / total_orders::numeric) * 100, 2) as cancel_ratio
	from cancel_ratio_24
)
select c.restaurant_id as restaurant_id , c.cancel_ratio as current_year_cancel_ratio,
l.cancel_ratio as last_year_cancel_ratio
from current_year_data as c
join last_year_date as l
on c.restaurant_id = l.restaurant_id;

**--Determine each rider's average delivery time.**

select o.order_id,o.order_time , d.delivery_time,
d.delivery_time - o.order_time as time_difference,
extract(epoch from(d.delivery_time - o.order_time + case when d.delivery_time < o.order_time then interval '1 day'
else interval'0 day'End))/60 as time_diff
from orders as o
join deliveries as d
	on o.order_id = d.order_id
	where d.delivery_status ='Delivered'
	group by 1,2,3;

**--Calculate each restaurant's growth ratio based on the total number of delivered orders since its
--joining.**

select o.restaurant_id,
to_char(o.order_date,'mm-yy') as month,
count(o.order_id) as orders_total
from orders as o
join deliveries as d
	on o.order_id = d.order_id
	where d.delivery_status = 'Delivered'
	group by 1,2
	order by 1,2;


**--Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the
--average order value (AOV). If a customer's total spending exceeds the AOV, label them as
--'Gold'; otherwise, label them as 'Silver'.
--Return: The total number of orders and total revenue for each segment.**

select customer_category,sum(total),sum(total_count)
from
(select customer_id,sum(total_amount) as total,count(order_id) as total_count,
case when sum(total_amount) > (select avg(total_amount) from orders) then 'Gold'
else 'Silver'
end as customer_category
from orders
group by 1) as t1
group by 1;

**--Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.**

select d.rider_id,
to_char(o.order_date,'mm-yy') as month,
sum(o.total_amount) as total_revenue,
sum(o.total_amount) * 0.08 as rider_revenue
from orders as o
join deliveries as d
	on o.order_id = d.order_id
	group by 1,2
	order by 1,3 DESC;

**--Find the number of 5-star, 4-star, and 3-star ratings each rider has.
--Riders receive ratings based on delivery time:
--● 5-star: Delivered in less than 15 minutes
--● 4-star: Delivered between 15 and 20 minutes
--● 3-star: Delivered after 20 minutes**

select rider_id,stars,count(*) as total_stars
from

	(select rider_id,
	delivery_time_took,
	case 
		when delivery_time_took < 15 then '5 star'
		when delivery_time_took between 15 and 20 then '4 star'
		else '3 star'
		end as stars
	from
		(select o.order_id , o.order_time,d.delivery_time,
		extract(Epoch from (d.delivery_time - o.order_time +
		case when d.delivery_time < o.order_time then Interval '1 day'
		else Interval '0 day' End))/60 as delivery_time_took,
		d.rider_id
		from orders as o
		join deliveries as d
			on o.order_id = d.order_id
		where d.delivery_status = 'Delivered') as t1
	) as t2
group by 1,2
order by 1,3 DESC;

**--Analyze order frequency per day of the week and identify the peak day for each restaurant.**

select *
from
(select r.restaurant_name,
to_char(o.order_date ,'Day') as day,
count(o.order_id) as total_orders,
rank() over (partition by r.restaurant_name order by count(order_id) DESC) as rank_1
from orders as o
join restaurants as r
	on o.restaurant_id = r.restaurant_id
	group by 1,2
	order by 1,4) as t1
where rank_1 = 1
;

**--Calculate the total revenue generated by each customer over all their orders.**

select c.customer_id,c.customer_name,sum(o.total_amount)
from orders as o
join customers as c
	on o.customer_id =  c.customer_id
	group by 1,2
	order by 3 DESC;

**--Identify sales trends by comparing each month's total sales to the previous month.**

select 
extract(year from order_date) as year,
extract(month from order_date ) as month,
sum(total_amount) as total_revenue,
lag(sum(total_amount),1) over(order by extract(year from order_date),extract(month from order_date )) as previous_month
from orders
group by 1,2

**--Evaluate rider efficiency by determining average delivery times and identifying those with the
--lowest and highest**

with new_table as
(
	select d.rider_id as rider_id,
	extract(Epoch from (d.delivery_time - o.order_time +
		case when d.delivery_time < o.order_time then Interval '1 day'
		else Interval '0 day' End))/60 as time_taken
	from orders as o
	join deliveries as d
	on o.order_id = d.order_id
	where d.delivery_status = 'Delivered'
),
rider_time as 
(
	select rider_id,
	avg(time_taken) as avg_time
	from new_table
	group by 1
)
select
min(avg_time),
max(avg_time)
from rider_time

**--Track the popularity of specific order items over time and identify seasonal demand spikes**

select order_item , 
seasons,
count(order_id) as total
from
(select *,
extract(month from order_date) as month,
case
	when extract(month from order_date) between 4 and 6 then 'spring'
	when extract(month from order_date) > 6 and 
	extract(month from order_date) < 9 then 'summer'
	else 'winter'
end as seasons
from orders) as t1
group by 1,2


**--Rank each city based on the total revenue for the last year (2023).**

select r.city,sum(o.total_amount) as total_revenue,
rank() over (order by sum(o.total_amount) DESC) as city_rank
from orders as o
join restaurants as r
	on o.restaurant_id = o.restaurant_id
	group by 1;




















