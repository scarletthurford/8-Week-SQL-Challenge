# Case Study #4 - Data Bank

Data Bank is the fourth case study in the 8-week SQL challenge. Please note that all information for this task is from:
https://8weeksqlchallenge.com/case-study-4/

## Problem Statement

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

Danny has shared 3 key datasets for this case study: <br/>
• regions  <br/>
• customer nodes <br/>
• customer transactions <br/>

## A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

<br/>

```sql
SELECT
  COUNT(DISTINCT node_id) AS unique_nodes
FROM data_bank.customer_nodes;
```

#### Results:

| unique_nodes |
| ------------ |
| 5 |

There are 5 unique nodes on the Data Bank system.

### 2. What is the number of nodes per region?

<br/>

```sql
SELECT
  r.region_name,
  COUNT(c.node_id) AS node_count
FROM data_bank.customer_nodes AS c
JOIN data_bank.regions AS r
  ON c.region_id = r.region_id
GROUP BY c.region_id, r.region_name
ORDER BY c.region_id;
```

#### Results: 

| region_name	| node_count |
| ------------ | ------------ |
| Australia	| 770 |
| America | 735 |
| Africa | 714 |
| Asia | 665 |
| Europe | 616 | 

### 3. How many customers are allocated to each region?

<br/>

```sql
SELECT
  r.region_name,
  COUNT(c.customer_id) AS customer_count
FROM data_bank.customer_nodes AS c
JOIN data_bank.regions AS r
  ON c.region_id = r.region_id
GROUP BY c.region_id, r.region_name
ORDER BY c.region_id;
```

#### Results:

| region_name	| customer_count |
| ------------ | ------------ |
| Australia | 770 |
| America |	735 |
| Africa | 714 |
| Asia | 665 |
| Europe | 616 |

### 4. How many days on average are customers reallocated to a different node?

<br/>

```sql
WITH node_days AS (
  SELECT 
    customer_id, 
    node_id,
    end_date - start_date AS days_in_node
  FROM data_bank.customer_nodes
  WHERE end_date != '9999-12-31'
  GROUP BY customer_id, node_id, start_date, end_date
),
sum_node_days AS (
  SELECT 
    customer_id,
    node_id,
    SUM(days_in_node) AS total_days_in_node
  FROM node_days
  GROUP BY customer_id, node_id
)

SELECT 
  ROUND(AVG(total_days_in_node)) AS average_node_reallocation
FROM sum_node_days;
```

#### Results:

| average_node_reallocation	|
| ------------ |
| 24 | 

It takes an average of 24 days for customers to be reallocated to a different node.

## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?

<br/>

```sql
SELECT
  txn_type,
  COUNT(customer_id) AS transaction_count,
  SUM(txn_amount) AS transaction_total
FROM data_bank.customer_transactions
GROUP BY txn_type;
```
#### Results:

| txn_type | transaction_count | transaction_total |
| ------------ | ------------ | ------------ |
| purchase | 1617	| 806537 |
| deposit | 2671 | 1359168 |
| withdrawal | 1580 | 793003 |


### 2. What is the average total historical deposit counts and amounts for all customers?

<br/>

```sql
WITH total_deposits AS (
 SELECT 
    COUNT(customer_id) AS customer_count,
    AVG(txn_amount) AS average_txn
  FROM data_bank.customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
)
SELECT
  ROUND(AVG(customer_count)) AS deposit_count,
  ROUND(AVG(average_txn)) AS deposit_amount
FROM total_deposits;
```

#### Results:

| deposit_count | deposit_amount | 
| ------------ | ------------ | 
| 5 | 509	| 




