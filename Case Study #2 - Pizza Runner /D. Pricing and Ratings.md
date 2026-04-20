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
