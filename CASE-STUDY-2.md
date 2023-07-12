# CASE 2 - PIZZA RUNNER

##Link to the case study: https://8weeksqlchallenge.com/case-study-2/

### A. Pizza Metrics

How many pizzas were ordered?
```
select count(*) from customer_orders;
```

How many unique customer orders were made?
```
select count(distinct order_id) from customer_orders;
```

How many successful orders were delivered by each runner?
```
select runner_id, count(order_id) from runner_orders
where cancellation is null 
group by runner_id;
```

How many of each type of pizza was delivered?
```
select pn.pizza_name, count(co.pizza_id) from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
join pizza_names pn on co.pizza_id = pn.pizza_id
where ro.cancellation is null
group by pn.pizza_name;
```

How many Vegetarian and Meatlovers were ordered by each customer?
```
select 
	customer_id, 
	sum(case when co.pizza_id = 1 then 1 else 0 end) as meatlovers,
	sum(case when co.pizza_id = 2 then 1 else 0 end) as vegetarian
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
join pizza_names pn on co.pizza_id = pn.pizza_id
group by customer_id
order by customer_id;
```

What was the maximum number of pizzas delivered in a single order?
```
select 
	co.order_id, 
	count(pizza_id) 
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null
group by co.order_id
order by count(pizza_id) desc
limit 1;
```

For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```
select 
	customer_id,
	sum(case when exclusions is null and extras is null then 1 else 0 end) as no_change,
	sum(case when exclusions is not null or extras is not null then 1 else 0 end) as changed
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null
group by customer_id
order by customer_id;
```

How many pizzas were delivered that had both exclusions and extras?
```
select customer_id, count(pizza_id) as pizza_double_changed
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where exclusions is not null and extras is not null and cancellation is null
group by customer_id;
```

What was the total volume of pizzas ordered for each hour of the day?
```
select 
	extract(hour from order_time) as hour, 
	count(pizza_id),
	round(100*count(order_id)/sum(count(order_id)) over(), 2) as "Volume"
from customer_orders
group by extract(hour from order_time)
order by hour;
```

What was the volume of orders for each day of the week?
```
select 
	extract(dow from order_time) as day, 
	round(100*count(order_id)/sum(count(order_id)) over(), 2) as "Volume"
from customer_orders
group by extract(dow from order_time)
order by day;
```

### B. Runner and Customer Experience

How many runners signed up for each 1 week period? 
```
select count(runner_id), extract(week from registration_date)
from runners
group by extract(week from registration_date);
```

What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```
with x as (
	select runner_id, extract(minute from (CAST(pickup_time as TIMESTAMP) - order_time)) as x
	from runner_orders ro
	join customer_orders co on 
		co.order_id = ro.order_id) 
select runner_id, round(avg(x)) from x
group by runner_id;

```

Is there any relationship between the number of pizzas and how long the order takes to prepare?
```
with x as (
select count(pizza_id) as num_pizza, extract(minute from (CAST(pickup_time as TIMESTAMP) - order_time)) as time
from runner_orders ro
join customer_orders co on 
co.order_id = ro.order_id
where ro.distance != 0
group by runner_id, extract(minute from (CAST(pickup_time as TIMESTAMP) - order_time)))

select num_pizza, round(avg(time))
from x
group by num_pizza;
```

What was the average distance travelled for each customer?
```
select customer_id, avg(distance) as avg_distance
from runner_orders ro
join customer_orders co on co.order_id = ro.order_id
group by customer_id;
```

What was the difference between the longest and shortest delivery times for all orders?
```
select max(duration) - min(duration)
from runner_orders;
```

What was the average speed for each runner for each delivery and do you notice any trend for these values?
```
select runner_id, order_id, distance, round(avg(distance/(duration/ 60)))
from runner_orders
where cancellation is null
group by runner_id, order_id, distance;
```

What is the successful delivery percentage for each runner?
```
select runner_id,
	count(pickup_time) as delivered_orders,
	count(*) as total_orders,
	round(100 * count(pickup_time) / count(*)) as pct
from runner_orders_temp
group by runner_id
```

### C. Ingredient Optimisation

What are the standard ingredients for each pizza?
```
with x as (
SELECT pizza_id, CAST(TRIM(unnest(regexp_split_to_array(toppings, ','))) as INT) as topping
FROM pizza_recipes),

y as (select pizza_name, x.pizza_id, topping, topping_name
from x 
join pizza_toppings pt on x.topping = pt.topping_id
join pizza_names pn on x.pizza_id = pn.pizza_id)

select pizza_name, STRING_AGG(topping_name, ',')
from y
group by pizza_name;
```






