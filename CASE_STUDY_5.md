# Case Study #5 - Data Mart ###
Link to the case study : https://8weeksqlchallenge.com/case-study-5/

## 1. Data Cleansing Steps

The 'cws' temporary table will be used throughout the whole case. 

### In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

```
CREATE TEMP TEABLE cws  as (
	SELECT 
                region, 
                platform, 
                customer_type,
		week_date::date as week_date, 
		extract(week from week_date::date) as week, 
		extract(month from week_date::date) as month_number, 
		extract(year from week_date::date) as calendar_year, 
		segment, 
		case 
			when segment like '_1' then 'Young Adults' 
			when segment like '_2' then 'Middle Aged'
			when segment like '_3' or segment like '_4' then 'Retirees'
			when segment is null then 'unknown' 
			end as age_band, 
		case 
			when segment like 'F_' then 'Families' 
			when segment like 'C_' then 'Couples' 
			when segment is null then 'unknown' 
			end as demographic, 
		transactions, 
		sales,
		round(sales/transactions, 2) as avg_transaction
	from weekly_sales)

 
 ```
 
## 2. Data Exploration

### What day of the week is used for each week_date value?
 
``` 
 
 select distinct extract(dow from week_date) from cws
```

### What range of week numbers are missing from the dataset?

```
SELECT *
FROM generate_series(1, 52) AS missing_weeks
WHERE NOT EXISTS (
		SELECT 1
		FROM clean_weekly_sales
		WHERE missing_weeks = week_number
	);
 
```
### How many total transactions were there for each year in the dataset?
```
select calendar_year, sum(transactions) as total_sales from cws group by calendar_year
```

### What is the total sales for each region for each month?
```
select 
    region, calendar_year, month_number, sum(sales) as total_sales from cws 
group by 
    region, calendar_year, month_number
order by 
    calendar_year, month_number, region
```

### What is the total count of transactions for each platform

```
SELECT platform,
	sum(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform
```

### What is the percentage of sales for Retail vs Shopify for each month?
```
with percent as (	
		select 
		calendar_year as calendar_year,
		month_number,
		sales,
		case when platform = 'Retail' then sales else null end as retail,
		case when platform = 'Shopify' then sales else null end as Shopify
		from cws)

select calendar_year, month_number, round((sum(retail::numeric)/sum(sales))*100, 2) as retail, round((sum(Shopify::numeric)/sum(sales))*100, 2) as shopify
from percent
group by calendar_year, month_number

```
### What is the percentage of sales by demographic for each year in the dataset?

```
with percent as (	
		select 
		calendar_year as calendar_year,
		month_number,
		sales,
		case when demographic = 'Families' then sales else null end as Families,
		case when demographic = 'Couples' then sales else null end as Couples
		from cws)

select calendar_year, round((sum(Couples::numeric)/sum(sales))*100, 2) as Couples, round((sum(Families::numeric)/sum(sales))*100, 2) as Families
from percent
group by calendar_year
```

Which age_band and demographic values contribute the most to Retail sales?

```
WITH get_total_sales_from_all AS (
	SELECT
		demographic,
		age_band,
		sum(sales) AS total_sales,
		rank() OVER (ORDER BY sum(sales) desc) AS rank,
		round(100 * sum(sales) / sum(sum(sales)) over (), 2) AS percentage
	FROM 
		cws
	WHERE platform = 'Retail' AND age_band <> 'unknown'
	GROUP BY demographic, age_band
	)
SELECT
	demographic, age_band, total_sales, percentage
from
	get_total_sales_from_all
WHERE rank = 1
```
### Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

In order to have reliable data, we have to avoid averages of averages. We cannot use the avg_trancation column.

```
SELECT calendar_year,
	platform,
	(sum(sales) / sum(transactions)) AS avg_transaction_size
FROM clean_weekly_sales
GROUP BY calendar_year,
	platform
ORDER BY calendar_year,
	platform;
  
```

## 3. Before & After Analysis
 

### What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

```
WITH p as (
	  SELECT 
      calendar_year,
      sum(CASE WHEN week BETWEEN 21 AND 24 THEN sales else NULL end) as before, 
      sum(CASE WHEN week BETWEEN 25 AND 28 THEN sales else NULL end) as after
	  from cws
	  group by calendar_year)
	
SELECT before, after, (1 - before::numeric/after::numeric) * 100 from p
```


### What about the entire 12 weeks before and after?

```
WITH p as (
	  SELECT 
      calendar_year,
      sum(CASE WHEN week BETWEEN 12 AND 24 THEN sales else NULL end) as before, 
      sum(CASE WHEN week BETWEEN 25 AND 37 THEN sales else NULL end) as after
	  from cws
	  group by calendar_year)
	
SELECT before, after, (1 - before::numeric/after::numeric) * 100 from p
```
  

### How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

```
with p as (...)

SELECT calendar_yearn, before, after, (1 - before::numeric/after::numeric) * 100 from p
where  calendar_year between '2018' and '2020'
group by calendar_year

select calendar_year, before, round(((after::numeric - before::numeric)/before::numeric)*100, 2), after from g
```
