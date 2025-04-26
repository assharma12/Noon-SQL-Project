# Noon-SQL-Project

## Noon launched Food in Dubai on 1st of May and your line manager has asked you to share key performance metrics to gauge the performance of the verticals. All the data of orders is being stored in the below table. Kindly use this table to write queries to find the insights.


## Here are the queries for this projects 

```sql
select * from orders ```

# Top 3 outlets by cuisine without using top and  limit

```sql
 with order_count as(
select  cuisine,Restaurant_id, count(*) as no_of_orders
from orders
group by cuisine,Restaurant_id
order by no_of_orders desc)
select * from(
select *,row_number() over(partition by cuisine  order by no_of_orders desc) as rnk
from order_count
) a
where rnk <=3 ```

# Find the daily new customer count from the launch date (Every day how many new customers are we Acquiring)
```sql
  with cte as (
select customer_code,cast(min(placed_at) as date) as first_order_date
from orders
group by customer_code
)
select first_order_date,count(*) as new_cust_count
from cte 
group by first_order_date
order by first_order_date ```


# 2nd solution

```sql
  WITH customer_first_orders AS (
  SELECT
    customer_code,
    DATE(placed_at) AS order_date,
    ROW_NUMBER() OVER (PARTITION BY customer_code ORDER BY placed_at) AS rn
  FROM orders
)
SELECT
  order_date,
  COUNT(*) AS new_customers
FROM customer_first_orders
WHERE rn = 1
GROUP BY order_date
ORDER BY order_date ```

# Count of all the users who were acquired in jan 2025 and only placed one order in jan
# and did not placed any other order

```sql
 select customer_code,count(*) as no_of_orders
from orders
where month(Placed_at) = 1 and year(Placed_at) = 2025 
and Customer_code not in (select distinct customer_code 
from orders
where  month(placed_at) !=1 and year(placed_at) = 2025) 
group by customer_code
having count(*)=1 ```

 # list all the customer with no order in last 7 day and were acquired one month ago with their first order on promo
select* from orders

```sql
 with cte as( 
select customer_code,min(placed_at) as first_order_date,max(placed_at) as Latest_order_date
from orders 
group by customer_code)
select cte.*,orders.promo_code_name
from cte join orders on cte.customer_code = orders.customer_code and cte.first_order_date = orders.placed_at
where cte.Latest_order_date < date_sub(curdate(),interval 7 day)
and cte.first_order_date < date_sub(curdate(),interval 1 month)
and orders.promo_code_name is not null ```


# Growth team is planning to create a trigger that will target customers after thier every third order with a personalized
# commnucation and they have asked you to create a query for this

```sql
 with cte as (
select *,
   row_number() over(partition by customer_code order by placed_at) as order_count
 from orders)
select * 
  from cte 
   where order_count%3 = 0 and cast(placed_at as date) = curdate() ```

# List all customer who   placed more than 1 order  and all thier orders on a promo only

 

```sql
 select customer_code,count(*) as no_of_orders,count(promo_code_name) as promo_count
from orders
where promo_code_name is not null
group by customer_code
having count(*) >1 and count(*) = promo_count ```

# What percantage of customers were organically  aquired in Jan 2025 (Placed thier first order without promo code)

```sql
 with cte as(
select *,
row_number() over(partition by customer_code order by placed_at) as rn 
from orders
where month(placed_at) = 1)
 select  count(case when rn = 1 and promo_code_name is null then customer_code end)*100.0/count(distinct customer_code)
 as organic_count
from cte ```
  
  

