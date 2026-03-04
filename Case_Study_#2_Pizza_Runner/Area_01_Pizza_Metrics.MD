This area consiste of 10 questions;
<ol>
<li>How many pizzas were ordered?</li>
<li>How many unique customer orders were made?</li>
<li>How many successful orders were delivered by each runner?</li>
<li>How many of each type of pizza was delivered?</li>
<li>How many Vegetarian and Meatlovers were ordered by each customer?</li>
<li>What was the maximum number of pizzas delivered in a single order?</li>
<li>For each customer, how many delivered pizzas had at least 1 change and how many had no changes?</li>
<li>How many pizzas were delivered that had both exclusions and extras?</li>
<li>What was the total volume of pizzas ordered for each hour of the day?</li>
<li>What was the volume of orders for each day of the week?</li>
</ol>

***

# 01.How many pizzas were ordered?
The data set to be used for this question is customer_orders, but it has inconsistencies in data, so the data set is cleaned first. Please find the queries for data cleaning andstandardizing here. 

#### SQL Query

````sql
select count(pizza_id) as Pizza_order_count
from customer_orders;
````
<ol>
  <p> ->The query is fairly direct. SELECT statement is used with COUNT to find the total number of pizzas</p>
</ol>

#### Answer:

<img width="143" height="60" alt="Image" src="https://github.com/user-attachments/assets/5dfcb97a-cc50-4044-a9c8-fadeb76d203e" />

- A total of 14 pizzas were ordered

***

#02.How many unique customer orders were made?

#### SQL Query

````sql
select count(distinct order_id) as unique_order_count
from customer_orders;
````
<ol>
  <p> ->The order numbers may be repeated as one order may contain several pizzas with varying extras and exclusions</p>
  <p> ->DISTINCT function is used to count the number of unique orders</p>
</ol>

#### Answer:

<img width="152" height="68" alt="Image" src="https://github.com/user-attachments/assets/0b8df91c-cbb3-4c8b-a7a4-d74290975e06" />

- A total of 10 unique orders were made 

***

#03.How many successful orders were delivered by each runner?

#### SQL Query

````sql
select count(*) 
from runner_orders
where cancellation is null;
````

<ol>
  <p> ->The standardized runner_orders table has been used. </p>
  <p> ->IS NULL function has been used to filter orders with no update to the cancellation column</p>
</ol>

#### Answer:

<img width="146" height="59" alt="Image" src="https://github.com/user-attachments/assets/be3d4e9e-698b-4c65-80c4-b5bb57133053" />

- There a total of 8 successful orders 

***

#04.How many of each type of pizza was delivered?

#### SQL Query

````sql
select distinct n.pizza_id, any_value (n.pizza_name) as pizza_type, count(c.pizza_id) as delivered_count  
from
	customer_orders c
join
pizza_names n on c.pizza_id = n.pizza_id
join 
runner_orders r on c.order_id = r.order_id 
where r.cancellation is null
group by n.pizza_id;
````

<ol>
  <p> ->Customer_orders, pizza_names and runner_orders tables have been combined with 2 JOIN statements back to back in order to get the name, count and the cancellation factor in one query. </p>
  <p> ->Then successful orders have been counted as per the category</p>
</ol>

#### Answer:

<img width="265" height="83" alt="Image" src="https://github.com/user-attachments/assets/c13a78cd-1f59-4adc-945c-f873b6298627" />

- 9 "Meatlovers" pizzas were delivered
- 3 "Vegetarian" pizzas were delivered

***

#05.How many Vegetarian and Meatlovers were ordered by each customer?

#### SQL Query

````sql
select c.customer_id, n.pizza_name, count(c.pizza_id) as order_count
from customer_orders c, pizza_names n
where c.pizza_id = n.pizza_id
group by c.customer_id, n.pizza_name
order by c.customer_id;
````

<ol>
  <p> ->The pizzas have been counted and grouped by pizza name. </p>
  <p> ->WHERE clause has been used instead of a join statement.</p>
  <p> ->By grouping the results as per the customer ID, we can clearlsy see how many of each category were ordered. 
  </ol>

#### Answer:

<img width="260" height="164" alt="Image" src="https://github.com/user-attachments/assets/20b4eb93-bccd-4cf7-9add-9f7c45d23797" />

- As queried previously we know that there were 14 orders. Out of which;
- Customer no.101 ordered 2 "Meatlovers" pizzas and 1 "Vegetarian" pizza
- Customer no.102 ordered 2 "Meatlovers" pizzas and 1 "Vegetarian" pizza
- Customer no.103 ordered 3 "Meatlovers" pizzas and 1 "Vegetarian" pizza
- Customer no.104 ordered 3 "Meatlovers" pizzas 
- Customer no.105 ordered 1 "Vegetarian" pizza

***

#06.What was the maximum number of pizzas delivered in a single order?

#### SQL Query

````sql
with delivered as
(
select c.order_id,c.customer_id,c.extras,c.exclusions,count(customer_id) as total_delivered
from customer_orders c
join runner_orders r
on c.order_id = r.order_id
where r.cancellation is null
group by order_id,customer_id,extras,exclusions
)
select order_id, customer_id,sum(total_delivered) as max_pizzas
from delivered
group by order_id,customer_id
order by max_pizzas desc
limit 1;
````

<ol>
  <p> ->A CTE is used to filter out the delivered pizzas </p>
  <p> ->Then the total no. of pizzas are calculated and ordered from max to min and then limited to 1 to show the most pizzas in a single order.</p>
  
 </ol>

#### Answer:

<img width="241" height="64" alt="Image" src="https://github.com/user-attachments/assets/8c6c44f2-fc43-4403-9a8b-e908dcd71e72" />

- The maximum number of pizzas delivered in a single order is 3.

***

#07.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

#### SQL Query

````sql
with delivered_pizzas as 
(select c.order_id,c.customer_id,c.extras,c.exclusions
from customer_orders c
join runner_orders r
on c.order_id = r.order_id
where r.cancellation is null
)
select customer_id, changes, count(changes) as count
from (
	select *, 
				case 
					when extras is null and exclusions is null then 'no change'
					when extras is not null or exclusions is not null then 'change'
					end as changes
	from delivered_pizzas
	 )as changes
group by customer_id,changes
order by customer_id;
````

<ol>
  <p> ->*This is my favorite question from the area as it really taught me alot and showed me how much I can achieve*</p>
  <p> ->*I approached the query by breaking it into 3 parts (delivered,change,no change) and combining them all together*</p>
  <p> ->This query uses a combination of CTE, subquery, case function and other group by and order by functions to find the desired result. </p>
  <p> ->The CTE is used to filter out only the delivered pizzas</p>
  <p> ->a CASE function is used within a subquery to flag orders that have a change (includes extras or exclusions) or no change (does not include extras and exclusions) </p>
  <p> ->Finally the results are grouped by the flags to show how many changes per customer
  
 </ol>

#### Answer:

<img width="238" height="134" alt="Image" src="https://github.com/user-attachments/assets/612c60ae-5e8c-4607-baaa-15df93dd4209" />

- Customer no.101 received 2 pizzas with no change.
- Customer no.102 received 3 pizzas with no change.
- Customer no.103 received 3 pizzas, all with some change.
- Customer no.104 received 2 pizzas with no change and 1 pizza with some change.
- Customer no.105 received 1 pizza with some change

***

#08.How many pizzas were delivered that had both exclusions and extras?

#### SQL Query

````sql
select c.customer_id,c.order_id, count(*) as delivered_orders_with_exclusions_and_extras
from 
runner_orders r
join 
customer_orders c
on r.order_id = c.order_id
where r.cancellation is null
and 
c.exclusions is not null
and c.extras is not null
group by c.order_id,c.customer_id;
````

<ol>
  <p> -> JOIN statement is used to combine runner_orders and customer_orders tables to include the "delivered" factor.</p>
  <p> -> WHERE clause is used with AND clause to filter the pizzas with both changes </p>
  
</ol>

#### Answer:

<img width="423" height="69" alt="Image" src="https://github.com/user-attachments/assets/2b3cb8fa-9514-4e0c-afaa-3197b01c6952" />

- Only 1 order has been delivered with both exclusions and extras.

***

#09.What was the total volume of pizzas ordered for each hour of the day?

#### SQL Query

````sql
select
extract(hour from order_time) as `hour`, count(order_id) as order_volume
from
customer_orders
group by `hour`
order by `hour`;
````

<ol>
  <p> -> EXTRACT statement has been used to obtain the hour from the order_time column with the time stamp.</p>
  <p> -> The orders are then counted and grouped by the hour to show the hourly volume of pizza orders</p>
  
</ol>

#### Answer:

<img width="206" height="132" alt="Image" src="https://github.com/user-attachments/assets/12fa45ce-0942-4efc-9001-969cb65ef101" />

- There is a spike in pizza orders in the afternoon at 1PM, and evening times at 6PM, 9PM and 11PM

***

#10.What was the volume of orders for each day of the week?

#### SQL Query

````sql

select
dayname(order_time) as `day`,
count(order_id) as order_volume
from customer_orders
group by `day`;

````

<ol>
  <p> -> DAYNAME query has been used to display the day of the week.</p>
  <p> -> Then the orders are counted and grouped by the day to show the volume of pizza orders by the day of the week </p>
  
</ol>

#### Answer:

<img width="191" height="95" alt="Image" src="https://github.com/user-attachments/assets/12de77cf-9764-4d16-8192-785139d3505b" />

- Wednesday and Saturday have had the most volume with 5 orders each on those days while there has been 3 orders placed on Thursday and 1 order on Friday.
- No orders found on Monday, Tuesday, Thursday and Sunday.

***

