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


