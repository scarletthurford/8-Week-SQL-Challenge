# Case Study #2 - Pizza Runner


## Data Cleaning

The customer_orders and runners_orders both contain null values

<br/>

```sql
CREATE TEMP TABLE customer_orders_temp AS
  SELECT
    order_id,
    customer_id, 
    pizza_id, 
    CASE
	    WHEN exclusions LIKE 'null' OR exclusions IS NULL THEN ' '
	    ELSE exclusions
	    END AS exclusions,
    CASE
	    WHEN extras LIKE 'null' OR extras IS NULL THEN ' '
	    ELSE extras
	    END AS extras,
	  order_time
FROM pizza_runner.customer_orders;
```

<br/>


```sql
CREATE TEMP TABLE runner_orders_temp AS
  SELECT 
    order_id,
    runner_id,
    CASE
      WHEN pickup_time = 'null' OR pickup_time IS NULL THEN ' '
      ELSE pickup_time
    END AS pickup_time,
    CASE
      WHEN distance = 'null' OR distance IS NULL THEN ' '
      WHEN distance LIKE '%km' THEN TRIM(BOTH 'km' FROM distance)
      ELSE distance
    END AS distance,
    CASE 
      WHEN duration = 'null' OR duration IS NULL THEN ' '
      WHEN duration LIKE '%minutes' THEN TRIM(BOTH 'minutes' FROM duration)
      WHEN duration LIKE '%mins' THEN TRIM(BOTH 'mins' FROM duration)
      WHEN duration LIKE '%minute' THEN TRIM(BOTH 'minute' FROM duration)
      ELSE duration
    END AS duration,
    CASE 
      WHEN cancellation = 'null' OR cancellation IS NULL THEN ' '
      ELSE cancellation
    END AS cancellation
  FROM pizza_runner.runner_orders;
```






