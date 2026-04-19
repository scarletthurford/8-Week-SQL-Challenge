# A. Pizza Metrics

## 1. How many pizzas were ordered?

<br/>

```sql
SELECT
  COUNT(order_id) AS pizza_count
FROM customer_orders_temp;
```

### Results:

| pizza_count | 
| ------------- | 
| 14 |

## 2. How many unique customer orders were made?

<br/>

```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_pizza_count
FROM customer_orders_temp;
```

### Results:

| unique_pizza_count | 
| ------------- | 
| 10 |

## 3. How many successful orders were delivered by each runner?

<br/>

```sql
SELECT
  runner_id,
  COUNT(order_id) AS successful_orders
FROM runner_orders_temp
WHERE cancellation NOT LIKE '%Cancellation'
GROUP BY runner_id
ORDER BY runner_id ASC;
```

### Results:

| pizza_count | successful_orders | 
| ------------- | ------------- | 
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |

## 4. How many of each type of pizza was delivered?

<br/>

```sql
SELECT
  p.pizza_name,
  COUNT(c.pizza_id) as pizzas_delivered
FROM customer_orders_temp as c 
JOIN pizza_names as p
  ON c.pizza_id = p.pizza_id
JOIN runner_orders_temp as r
  ON c.order_id = r.order_id
WHERE r.cancellation NOT LIKE '%Cancellation'
GROUP BY c.pizza_id, p.pizza_name;
```

### Results:

| pizza_name | pizzas_delivered | 
| ------------- | ------------- | 
| Vegetarian | 3 |
| Meatlovers | 9 |

## 5. How many Vegetarian and Meatlovers were ordered by each customer?

<br/>

```sql
SELECT
  c.customer_id,
  p.pizza_name,
  COUNT(c.pizza_id) AS pizza_count
FROM customer_orders_temp as c
JOIN pizza_names as p
  ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name, p.pizza_id
ORDER BY c.customer_id ASC;
```

### Results:

| pizza_name | pizza_name | pizza_count
| ------------- | ------------- | ------------- | 
| 101 | Meatlovers | 2 |
| 101 | Vegetarian | 1 |
| 102 | Meatlovers | 2 |
| 102 | Vegetarian | 1 |
| 103 | Meatlovers | 3 |
| 103 | Vegetarian | 1 |
| 104 | Meatlovers | 3 |
| 105 | Vegetarian | 1 |

## 6. What was the maximum number of pizzas delivered in a single order?

<br/>

```sql
SELECT
  c.order_id,
  COUNT(c.order_id) AS maximum_pizzas
FROM customer_orders_temp AS c
JOIN runner_orders_temp AS r
  ON c.order_id = r.order_id
WHERE r.cancellation NOT LIKE '%Cancellation'
GROUP BY c.order_id
ORDER BY maximum_pizzas DESC
LIMIT 1;
```

### Results:

| order_id | maximum_pizzas | 
| ------------- | ------------- | 
| 4 | 3 | 

## 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

<br/>

```sql
SELECT
  c.customer_id,
  SUM(CASE
    WHEN c.exclusions = ' ' AND c.extras = ' ' THEN 1
    ELSE 0 
    END) AS no_changes,
  SUM(CASE 
    WHEN c.exclusions <> ' ' OR c.extras <> ' ' THEN 1
    ELSE 0
    END) AS changes
 FROM customer_orders_temp AS c
 JOIN runner_orders_temp AS r
  ON c.order_id = r.order_id
 WHERE r.cancellation NOT LIKE '%Cancellation'
 GROUP BY c.customer_id
 ORDER BY c.customer_id;
```

### Results:

| customer_id | no_changes | changes |
| ------------- | ------------- | ------------- | 
| 101 | 2 | 0 |
| 102 | 3 | 0 |
| 103 | 0 | 3 |
| 104 | 1 | 2 |
| 105 | 0 | 1 |


## 8. How many pizzas were delivered that had both exclusions and extras?

<br/>

```sql
SELECT
  COUNT(c.order_id) as pizzas_delivered_ee
 FROM customer_orders_temp AS c
 JOIN runner_orders_temp AS r
  ON c.order_id = r.order_id
 WHERE r.cancellation NOT LIKE '%Cancellation'
  AND c.exclusions <> ' ' 
  AND c.extras <> ' ';
```

### Results:

| pizzas_delivered_ee | 
| ------------- | 
| 1 |

## 9. What was the total volume of pizzas ordered for each hour of the day?

<br/>

```sql
SELECT
  DATE_PART('hour', order_time) AS hour,
  COUNT(order_id) as pizzas_ordered
 FROM customer_orders_temp 
 GROUP BY hour
 ORDER BY hour ASC;
```

### Results:

| hour | pizzas_ordered | 
| ------------- | ------------- | 
| 11 | 1 | 
| 13 | 3 | 
| 18 | 3 | 
| 19 | 1 | 
| 21 | 3 | 
| 23 | 3 |

## 10. What was the volume of orders for each day of the week?

<br/>

```sql
SELECT
  CASE
    WHEN EXTRACT(DOW FROM order_time) = 0 THEN 'Sunday'
    WHEN EXTRACT(DOW FROM order_time) = 1 THEN 'Monday'
    WHEN EXTRACT(DOW FROM order_time) = 2 THEN 'Tuesday'
    WHEN EXTRACT(DOW FROM order_time) = 3 THEN 'Wednesday'
    WHEN EXTRACT(DOW FROM order_time) = 4 THEN 'Thursday'
    WHEN EXTRACT(DOW FROM order_time) = 5 THEN 'Friday'
    WHEN EXTRACT(DOW FROM order_time) = 6 THEN 'Saturday'
    ELSE 'other'
  END AS day_of_week,
  COUNT(order_id) AS pizzas_ordered
FROM customer_orders_temp
GROUP BY day_of_week,
  EXTRACT(DOW FROM order_time)
ORDER BY EXTRACT(DOW FROM order_time);
```

### Results:

| day_of_week | pizzas_ordered | 
| ------------- | ------------- | 
| Wednesday | 5 | 
| Thursday | 3 | 
| Friday | 1 | 
| Saturday | 5 | 
 


