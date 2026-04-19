# B. Runner and Customer Experience

## 1. How many runners signed up for each 1 week period? (i.e. week starts 2020-01-01)

<br/>

```sql
SELECT
     ((registration_date - DATE '2021-01-01')::int / 7) + 1 AS week,
    COUNT(runner_id) AS runner_count
FROM runners
GROUP BY week
ORDER BY week;
```

As 2020‑01‑01 was declared as the start of Week 1, I converted the date difference into an integer so it could be divided into 7‑day weekly intervals.

### Results:

| week | runner_count | 
| ------------- | ------------- | 
| 1 | 2 | 
| 2 | 1 | 
| 3 | 1 | 

## 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

<br/>

```sql
WITH average_runner_pickup AS (
SELECT
  r.runner_id,
  EXTRACT(EPOCH FROM (r.pickup_time::TIMESTAMP - c.order_time::TIMESTAMP)) / 60 AS min_diff
FROM customer_orders_temp as c
JOIN runner_orders_temp as r
  ON c.order_id = r.order_id
WHERE r.pickup_time <> ' '
)

SELECT
  runner_id,
  ROUND(AVG(min_diff)) AS avg_pickup_time
FROM average_runner_pickup
GROUP BY runner_id
ORDER BY runner_id ASC;
```

### Results:

| runner_id | avg_pickup_time | 
| ------------- | ------------- | 
| 1 | 16 | 
| 2 | 24 | 
| 3 | 10 |

## 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH prep_time AS (
SELECT
  c.order_id,
  COUNT(c.order_id) AS number_of_pizzas,
  EXTRACT(EPOCH FROM (r.pickup_time::TIMESTAMP - c.order_time::TIMESTAMP)) / 60 AS min_diff
FROM customer_orders_temp as c
JOIN runner_orders_temp as r
  ON c.order_id= r.order_id
WHERE r.pickup_time <> ' '
GROUP BY c.order_id, r.pickup_time, c.order_time
)

SELECT
  number_of_pizzas,
  ROUND(AVG(min_diff)) AS avg_pickup_time
FROM average_runner_pickup
GROUP BY number_of_pizzas
ORDER BY number_of_pizzas ASC;
```

### Results:

| number_of_pizzas | avg_pickup_time | 
| ------------- | ------------- | 
| 1 | 12 | 
| 2 | 18 | 
| 3 | 29 |

## 4. 

