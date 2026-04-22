# D. Pricing and Ratings

## 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

<br/>

```sql
SELECT
  p.pizza_name,
  CASE
    WHEN c.pizza_id = 1 THEN COUNT(c.pizza_id) * 12
    WHEN c.pizza_id = 2 THEN COUNT(c.pizza_id) * 10
  END AS money_made_USD
FROM customer_orders_temp as c
JOIN pizza_names as p
  ON c.pizza_id = p.pizza_id
GROUP BY p.pizza_name, c.pizza_id;
```

### Results:

| pizza_name | money_made_USD | 
| ------------- | ------------- | 
| Meatlovers | 120 |
| Vegetarian | 40 |


## 2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra

<br/>

```sql
WITH split_extras AS (
  SELECT
    UNNEST(STRING_TO_ARRAY(extras, ', '))::int AS extras_split
  FROM customer_orders_temp
  WHERE extras <> ' '
), 
extras_charges AS (
  SELECT
    SUM(CASE
      WHEN extras_split = 4 THEN 2
      ELSE 1
    END) AS extras_charge
  FROM split_extras
),
pizza_charges AS (
  SELECT
    SUM(CASE
      WHEN pizza_id = 1 THEN 12
      WHEN pizza_id = 2 THEN 10
    END) AS pizza_charge
  FROM customer_orders_temp 
)

SELECT
  extras_charge + pizza_charge AS total_charge_usd
FROM extras_charges, pizza_charges;
```

### Results:

| total_charge_usd | 
| ------------- | 
| 167 | 
