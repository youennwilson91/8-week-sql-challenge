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
