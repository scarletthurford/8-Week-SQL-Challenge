# Case Study #2 - Pizza Runner

Pizza Runner is the second case study in the 8-week SQL challenge. Please note that all information for this task is from https://8weeksqlchallenge.com/case-study-2/

## Business Task

Danny is expanding his Pizza Empire and launches Pizza Runner, a retro‑themed pizza delivery startup. Because Danny has a few years of experience as a data scientist, he is very aware that data collection is going to be critical in order to scale the business.

He requires further assistance to clean the operational datasets and explore customer and runner behaviour to help optimise delivery performance and improve business operations.

Danny has shared 6 key datasets within the `pizza_runner` schema for this task: </br>
• `runners` </br>
• `runner_orders` </br>
• `customer_orders` </br>
• `pizza_names` </br>
• `pizza_recipes` </br>
• `pizza_toppings` </br>

### Task Contents
- [Data Cleaning](#data-cleaning)
- **Solutions**
  - [A. Pizza Metrics](https://github.com/scarletthurford/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner%20/A.%20Pizza%20Metrics.md)
  - [B. Runner and Customer Experience](https://github.com/scarletthurford/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner%20/B.%20Runner%20and%20Customer%20Experience.md)
  - [C. Ingredient Optimisation](https://github.com/scarletthurford/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner%20/C.%20Ingredient%20Optimisation.md)
  - [D. Pricing and Ratings](https://github.com/scarletthurford/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner%20/D.%20Pricing%20and%20Ratings.md)


## Data Cleaning

### Customer Orders Table

Looking at the `customer_orders` table below, you can see that there are missing and null values in both the `exclusions` and `extras` columns.

<img width="1800" height="700" alt="customer_orders" src="https://github.com/user-attachments/assets/a019f52e-8dab-4113-89e8-ba623d606c1f" /> </br>

To clean the table, the steps I took were as follows: 

1. Create a temporary table with all of the columns 
2. Remove null values in `exclusions` and `extras` columns and replace blank space with ' '.

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

Following cleaning, the `customers_orders_temp` table was used to run all queries.

<img width="1800" height="700" alt="customer_orders_cleaned" src="https://github.com/user-attachments/assets/6e31bb4f-9783-4bb8-b3bc-c4762843dde4" />

### Runner Orders Table

Looking at the `runner_orders` table below, there are missing and null values in the columns: `pickup_time`, `distance`, `duration`, and `cancellation`.

<img width="1800" height="507" alt="runner_orders" src="https://github.com/user-attachments/assets/31af5a62-751a-4533-ab20-09e34be1c393" /> </br>

To clean the table, the steps I took were as follows: 
1. Replace nulls and missing values with blank space ' ' in `pickup_time`.
2. Remove "km" and replace nulls and missing values with blank space ' ' in `distance`.
3. Remove "minutes", "mins" and "minute" and replace nulls and missing values with blank space ' ' in `duration`.
4. Replace missing and null values with blank space ' ' in `cancellation`.

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

Following cleaning, the `runner_orders_temp` table was used to run all queries.

<img width="1800" height="507" alt="runner_orders_cleaned" src="https://github.com/user-attachments/assets/ddb38aa8-001a-4bda-a3da-dc6e35d77d43" />






