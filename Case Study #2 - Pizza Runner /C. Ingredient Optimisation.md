# C. Ingredient Optimisation

## 1. What are the standard ingredients for each pizza?

<br/>

```sql
WITH standard_ingredients AS (
  SELECT
    p_name.pizza_name,
    p_recipe.toppings
  FROM pizza_names AS p_name
  JOIN pizza_recipes AS p_recipe
    ON p_name.pizza_id = p_recipe.pizza_id
)

SELECT
  s.pizza_name,
  (SELECT
    STRING_AGG(t.topping_name, ', ')
   FROM (SELECT UNNEST(STRING_TO_ARRAY(s.toppings, ', '))::int AS top_id) AS joined_rows
   JOIN pizza_toppings AS t
     ON joined_rows.top_id = t.topping_id
   ) AS ingredients
FROM standard_ingredients AS s
ORDER BY s.pizza_name;
```
I needed to split the string of toppings using UNNEST, which separates the array into rows. I didn’t want many rows for the same pizza, so I used STRING_AGG to concatenate the list of toppings back into one row.

### Results:

| pizza_name | ingredients | 
| ------------- | ------------- | 
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami | 
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce | 

## 2. What was the most commonly added extra?

<br/>

```sql
WITH most_common_extra AS (
  SELECT
    UNNEST(STRING_TO_ARRAY(extras, ', '))::int AS extras_split
  FROM customer_orders_temp
  WHERE extras <> ' '
 )

SELECT
  p.topping_name,
  COUNT(extras_split) AS times_added
FROM most_common_extra as e
JOIN pizza_toppings AS p
  ON e.extras_split = p.topping_id 
GROUP BY e.extras_split, p.topping_name
ORDER BY times_added DESC
LIMIT 1;
```
Like previously, I needed to split up the extras using the UNNEST function as some orders had more than 1 extra. Then I got the count of these extras split up to find the most commonly added extra.
### Results:

| topping_name | times_added | 
| ------------- | ------------- | 
| Bacon | 4 | 

## 3. What was the most common exclusion?

<br/>

```sql
WITH most_common_exclusion AS (
  SELECT
    UNNEST(STRING_TO_ARRAY(exclusions, ', '))::int AS exclusions_split
  FROM customer_orders_temp
  WHERE exclusions <> ' '
)

SELECT
  p.topping_name,
  COUNT(exclusions_split) AS times_removed
FROM most_common_exclusion as e
JOIN pizza_toppings AS p
  ON e.exclusions_split = p.topping_id 
GROUP BY e.exclusions_split, p.topping_name
ORDER BY times_removed DESC
LIMIT 1;
```

This question had the same steps as the previous one but with exclusions.

### Results:

| topping_name | times_removed | 
| ------------- | ------------- | 
| Cheese | 4 | 

## 4. 
