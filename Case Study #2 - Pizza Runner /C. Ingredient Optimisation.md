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

