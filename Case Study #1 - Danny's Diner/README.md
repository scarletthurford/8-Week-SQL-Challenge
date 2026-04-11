
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

