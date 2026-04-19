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

<br/>

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

## 4. What was the average distance travelled for each customer?

<br/>

```sql
SELECT
  c.customer_id,
  ROUND(AVG(r.distance::numeric), 1) AS avg_distance_travelled
 FROM customer_orders_temp as c
 JOIN runner_orders_temp as r
  ON c.order_id = r.order_id
WHERE r.cancellation NOT LIKE '%Cancellation'
GROUP BY c.customer_id
ORDER BY c.customer_id ASC;
```

### Results:

| customer_id | avg_distance_travelled | 
| ------------- | ------------- | 
| 101 | 20.0 | 
| 102 | 16.7 | 
| 103 | 23.4 |
| 104 | 10.0 | 
| 105 | 25.0 | 

## 5. What was the difference between the longest and shortest delivery times for all orders?

<br/>

```sql
SELECT
  (MAX(duration::numeric) - MIN(duration::numeric)) AS delivery_time_diff
FROM runner_orders_temp
WHERE duration <> ' ';
```

### Results:

| delivery_time_diff | 
| ------------- | 
| 30 |

## 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

<br/>

```sql
SELECT
  order_id,
  runner_id,
  distance,
  duration,
  ROUND((distance::numeric / duration::numeric) * 60, 1) AS avg_speed
FROM runner_orders_temp
WHERE distance <> ' ' AND duration <> ' ';
```

### Results

| order_id | runner_id | distance | duration | avg_speed |
| ------------- | ------------- | ------------- | ------------- | ------------- | 
| 1 | 1 | 20 | 32 | 37.5 | 
| 2 | 1 | 20 | 27 | 44.4 | 
| 3 | 1 | 13.4 | 20 | 40.2 | 
| 4 | 2 | 23.4 | 40 | 35.1 | 
| 5 | 3 | 10 | 15 | 40.0 | 
| 7 | 2 | 25 | 25 | 60.0 | 
| 8 | 2 | 23.4 | 15 | 93.6 |
| 10 | 1 | 10 | 10 | 60.0 | 

## 7. What is the successful delivery percentage for each runner?

<br/>

```sql  
SELECT
  runner_id,
  ROUND(100 * SUM(
    CASE WHEN cancellation NOT LIKE '%Cancellation' THEN 1
      ELSE 0
      END) / COUNT(*), 0
  ) AS success_delivery_percent
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id;

```

### Results:

| runner_id | success_delivery_percent |
| ------------- | ------------- | 
| 1 | 100 |
| 2 | 75 |
| 3 | 50 |

