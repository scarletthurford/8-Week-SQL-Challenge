# Case Study #1 - Danny's Diner

Danny's diner is the first case study in the 8-week SQL challenge. Please note that all information for this task is from https://8weeksqlchallenge.com/case-study-1/

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. He plans on using these insights to help him decide whether he should expand the existing customer loyalty program.

Danny has shared 3 key datasets for this case study: <br/>
• sales <br/>
• menu <br/>
• members <br/>
 
## Business Questions and Results
### 1. What is the total amount each customer spent at the restaurant?

<br/>

```sql
SELECT
  sales.customer_id,
  SUM(menu.price) AS total_amount
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```

#### Results:
| customer_id | total_amount |
| ------------- | ------------- |
| A  | 76  |
| B  | 74 |
| C  | 36 |

### 2. How many days has each customer visited the restaurant?

<br/>

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC;
```

#### Results:
| customer_id | days_visited |
| ------------- | ------------- |
| A  | 4  |
| B  | 6 |
| C  | 2 |

### 3. What was the first item from the menu purchased by each customer?

<br/>

```sql
WITH ordered_sales AS(
  SELECT 
	menu.product_name,
  	sales.customer_id,
  	sales.order_date,
  	DENSE_RANK() OVER (
  ORDER BY sales.order_date ASC) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name, sales.order_date
) 

SELECT
  customer_id,
  product_name
FROM ordered_sales
WHERE rank = 1
ORDER BY customer_id ASC;
```

#### Results:
| customer_id | product_name |
| ------------- | ------------- |
| A  | curry  |
| A  | sushi  |
| B  | curry |
| C  | ramen |

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers? 

<br/>

```sql
SELECT
  menu.product_name AS most_purchased_item,
  COUNT(sales.product_id) times_purchased
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```

#### Results:
| most_purchased_item | times_purchased |
| ------------- | ------------- |
| ramen  | 8 |

### 5. Which item was the most popular for each customer?

<br/>

```sql
WITH most_popular AS (
  SELECT
    sales.customer_id,
    menu.product_name,
  	COUNT(sales.product_id) AS product_count,
  	DENSE_RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.product_id) DESC) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT
  customer_id,
  product_name,
  product_count
FROM most_popular
WHERE rank = 1;
```

#### Results:
| customer_id | product_name | product_count |
| ------------- | ------------- | ------------- |
| A  | ramen  | 3  |
| B | ramen  | 2  |
| B | curry  | 2  |
| B  | sushi | 2  |
| C  | ramen | 3  |

### 6. Which item was purchased first by the customer after they became a member?

<br/>

```sql
WITH after_cust_joining AS (
  SELECT
    sales.customer_id,
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
    AND sales.order_date > members.join_date
)

SELECT
  customer_id,
  product_name AS product_after_joining
FROM after_cust_joining
INNER JOIN dannys_diner.menu
  ON after_cust_joining.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

#### Results:
| customer_id | product_after_joining | 
| ------------- | ------------- | 
| A  | ramen  | 
| B | sushi  | 


### 7. Which item was purchased just before the customer became a member?

<br/>

```sql
WITH before_cust_joining AS (
  SELECT
    sales.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS row_num
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
    AND sales.order_date < members.join_date
)

SELECT
  customer_id,
  product_name AS product_before_joining
FROM before_cust_joining
INNER JOIN dannys_diner.menu
  ON before_cust_joining.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

#### Results:
| customer_id | product_before_joining | 
| ------------- | ------------- | 
| A  | sushi  | 
| B | sushi  | 

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
  sales.customer_id,
  COUNT(sales.product_id) AS total_items,
  SUM(menu.price) AS total_amount
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id 
  AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```

| customer_id | total_items | total_amount |
| ------------- | ------------- | ------------- | 
| A  | 2 | 25 |
| B | 3 | 40 |


### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

<br/>

```sql
WITH customer_points AS (
  SELECT 
    sales.customer_id,
    CASE 
      WHEN sales.product_id = 1 THEN menu.price * 2 * 10
      ELSE menu.price * 10
    END AS points
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT
  customer_id,
  SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY customer_id ASC;
```

#### Results:
| customer_id | total_points | 
| ------------- | ------------- | 
| A  | 860  | 
| B | 940  | 
| C | 360  | 

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

<br/>

```sql
WITH jan_points AS(
  SELECT
    customer_id, 
    join_date, 
    join_date + 6 AS week_after_join, 
    DATE_TRUNC( 'month', DATE '2021-01-31')
      + interval '1 month'
      - interval '1 day' AS end_date
  FROM dannys_diner.members
) 

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN sales.order_date BETWEEN jan_points.join_date AND jan_points.week_after_join THEN 2 * 10 * menu.price
    WHEN sales.product_id = 1 THEN 2 * 10 * menu.price
    ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
INNER JOIN jan_points
  ON sales.customer_id = jan_points.customer_id
  AND sales.order_date >= jan_points.join_date
  AND sales.order_date <= jan_points.end_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id 
ORDER BY sales.customer_id ASC;
```

#### Results:
| customer_id | points | 
| ------------- | ------------- | 
| A  | 1020  | 
| B | 320  | 
