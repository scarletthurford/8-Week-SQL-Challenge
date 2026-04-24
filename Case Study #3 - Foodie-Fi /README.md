# Case Study #3 - Foodie-Fi

Foodie-Fi is the first case study in the 8-week SQL challenge. Please note that all information for this task is from:
https://8weeksqlchallenge.com/case-study-3/

## Problem Statement

Danny wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows! He finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

Danny has shared 2 key datasets for this case study: <br/>
• plans  <br/>
• subscriptions <br/>

## A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

I needed to first join the sample table with the plans table to retrieve the plan name that each customer was on.

<br/>

```sql
SELECT
  s.customer_id,
  p.plan_id, 
  p.plan_name,  
  s.start_date
FROM foodie_fi.plans as p
JOIN foodie_fi.subscriptions AS s
  ON p.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id ASC;
```
### Table:

<img src="Images/customer_analysis.png" alt="Customer_Analysis" width="50%">

Based on the results above, I have selected three customers to focus on and will describe their onboarding journey.

#### Customer 1:

| customer_id | plan_id | plan_name | start_date |
| ------------- | ------------- | ------------- | ------------- | 
| 1 | 0 | trial | 2020-08-01 |
| 1 | 1 | basic monthly | 2020-08-08 |

This customer started their free trial on 2020-08-01. After their trial ended, they joined the basic monthly plan. 

#### Customer 11:

| customer_id | plan_id | plan_name | start_date |
| ------------- | ------------- | ------------- | ------------- | 
| 11 | 0 | trial | 2020-11-19 |
| 11 | 4 | churn | 2020-11-26 |

This customer started their free trial on 2020-11-19. However, after their free trial ended on 2020-11-26, they decided to cancel the plan and not continue with the service, suggesting that they could either not afford the program or they did not enjoy it. 

#### Customer 13:

| customer_id | plan_id | plan_name | start_date |
| ------------- | ------------- | ------------- | ------------- | 
| 13 | 0 | trial | 2020-12-15 |
| 13 | 1 | basic monthly | 2020-12-22 |
| 13 | 2 | pro monthly | 2021-03-29 |

This customer started their free trial on 2020‑12‑15 and moved to the basic monthly plan once the trial ended. On 2021‑03‑29, they upgraded to the pro monthly plan, suggesting that after three months the basic plan no longer met their needs.

## B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

<br/>

```sql
SELECT
  COUNT(DISTINCT customer_id) as total_customers
FROM foodie_fi.subscriptions;
```

#### Results:

| total_customers |
| ------------- |
| 1000 |

Foodie_Fi has had 1000 different customers.

### 2. What is the monthly distribution of trial plan start_date values for our dataset?

<br/>

```sql
SELECT
  EXTRACT(MONTH FROM start_date) as month_number,
  TO_CHAR(start_date, 'Month') AS month_name,
  COUNT(s.customer_id) AS trial_subscriptions
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id = 0
GROUP BY month_number, month_name
ORDER BY month_number;
```
#### Results:

| month_number | month_name | trial_subscriptions |
| ------------- | ------------- | ------------- | 
| 1 | January | 88 |
| 2 | February | 68 |
| 3 | March | 94 |
| 4 | April | 81 |
| 5 | May | 88 |
| 6 | June | 79 |
| 7 | July | 89 |
| 8 | August | 88 |
| 9 | September | 87 |
| 10 | October | 79 |
| 11 | November | 75 |
| 12 | December | 84 |

Out of all the months, March has the highest number of people joining a free trial and February as the lowest.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

<br/>

```sql
SELECT 
  p.plan_id,
  p.plan_name,
  COUNT(s.customer_id) as count_of_events
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE s.start_date > '2020-12-31'
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;
```

#### Results:

| plan_id | plan_name | count_of_events |
| ------------- | ------------- | ------------- | 
| 1 | basic monthly | 8 |
| 2 | pro monthly | 60 |
| 3 | pro annual | 63 |
| 4 | churn | 71 |

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

<br/>

```sql
SELECT
  COUNT(DISTINCT s.customer_id) AS churn_customers,
  ROUND(100.0 * COUNT(DISTINCT s.customer_id) /
    (SELECT COUNT(DISTINCT customer_id)
    FROM foodie_fi.subscriptions), 1) AS churn_percentage
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE p.plan_id = 4;
```

#### Results:

| churn_customers | churn_percentage |
| ------------- | ------------- | 
| 307 | 30.7 |

Out of all 1000 customers who have ever joined Foodie-Fi, 307 have churned. This represents 30.7% of all customers.

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

<br/>

```sql
WITH lead_trial_churn AS (
  SELECT 
    s.customer_id,
    p.plan_name, 
	  LEAD(p.plan_name, 1) OVER (
      PARTITION BY s.customer_id
      ORDER BY s.start_date) AS next_plan
  FROM foodie_fi.subscriptions AS s
  JOIN foodie_fi.plans AS p
    ON s.plan_id = p.plan_id
)
  
SELECT
  COUNT(customer_id) AS churned_after_trial,
  ROUND(100 * COUNT(customer_id) /
    (SELECT COUNT(DISTINCT customer_id) 
    FROM foodie_fi.subscriptions)
  ) AS churn_percent_after_trial
FROM lead_trial_churn
WHERE plan_name = 'trial'
  AND next_plan = 'churn';
```

#### Results:

| churned_after_trial | churn_percent_after_trial |
| ------------- | ------------- | 
| 92 | 9 |

92 customers churned straight after their free trial. This represents 9% of all of Foodie-Fi's customers.

### 6. What is the number and percentage of customer plans after their initial free trial?

<br/>

```sql
WITH lead_trials_all AS (
  SELECT
    customer_id,
    plan_id,
    LEAD(plan_id, 1) OVER (
      PARTITION BY customer_id
      ORDER BY start_date) AS next_plan
  FROM foodie_fi.subscriptions 
)

SELECT
  next_plan AS plan_id,
  COUNT(customer_id) AS customers_after_trial,
  ROUND(100.0 * COUNT(DISTINCT customer_id) / 
    (SELECT COUNT(DISTINCT customer_id) 
    FROM foodie_fi.subscriptions), 1) AS customer_join_percentage
FROM lead_trials_all
WHERE next_plan IS NOT NULL 
  AND plan_id = 0
GROUP BY next_plan
ORDER BY next_plan;
```

#### Results:

| plan_id	| customers_after_trial	| customer_join_percentage | 
| ------------- | ------------- | ------------- |
| 1	| 546 |	54.6 |
| 2	| 325 |	32.5 |
| 3	| 37 | 3.7 |
| 4	| 92 | 9.2 |

The majority of customers joined either the basic monthly or pro monthly plan after their free trial.

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

<br/>

```sql
WITH customer_latest_plan AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY start_date DESC
    ) AS current_plan
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
  p.plan_name,
  COUNT(*) AS customers,
  ROUND(100.0 * COUNT(*) /
    SUM(COUNT(*)) OVER (), 1) AS percentage
FROM customer_latest_plan AS c
JOIN foodie_fi.plans AS p
  ON c.plan_id = p.plan_id
WHERE current_plan = 1
GROUP BY p.plan_name, c.plan_id
ORDER BY c.plan_id;
```

#### Results:

| plan_id	| customers	| percentage | 
| ------------- | ------------- | ------------- |
| trial	| 19 |	1.9 |
| basic monthly	| 224 |	22.4 |
| pro monthly	| 326 | 32.6 |
| pro annual	| 195 | 19.5 |
| churn	| 236 | 23.6 |

The majority of customers at 2020-12-31 were on a pro monthly plan.

### 8. How many customers have upgraded to an annual plan in 2020?

<br/>

```sql
SELECT
  COUNT(DISTINCT customer_id) as customer_count
FROM foodie_fi.subscriptions
WHERE plan_id = 3 AND start_date <= '2020-12-31'
```

#### Results:

| customer_count	|
| ------------- | 
| 195	|

195 different customers upgraded to an annual plan in 2020.

### 9.  How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

<br/>

```sql
WITH trial_join AS (
  SELECT 
    customer_id,
    start_date AS trial_date
   FROM foodie_fi.subscriptions
   WHERE plan_id = 0
),
annual_join AS (
  SELECT
    customer_id,
    start_date AS annual_date
  FROM foodie_fi.subscriptions
 WHERE plan_id = 3
)

SELECT 
  ROUND(AVG(a.annual_date - t.trial_date)) AS average_days
FROM trial_join AS t
JOIN annual_join AS a
  ON t.customer_id = a.customer_id;
```

#### Results:

| average_days	|
| ------------- | 
| 105	|

It takes 105 days on average for customers to upgrade to an annual plan from the day they join Foodie-Fi.

### 10.

<br/>

This question is similar to the previous one but with buckets, splitting up a year.

```sql
WITH trial_join AS (
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
), 
annual_join AS (
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
), 
buckets AS (
  SELECT 
    WIDTH_BUCKET(a.annual_date - t.trial_date, 0, 365, 12) AS days_to_upgrade
  FROM trial_join AS t
  JOIN annual_join AS a
    ON t.customer_id = a.customer_id
)
  
SELECT
  ((days_to_upgrade - 1) * 30 || ' - ' || days_to_upgrade * 30 || ' days') AS bucket,
  COUNT(*) AS customer_count
FROM buckets
GROUP BY days_to_upgrade
ORDER BY days_to_upgrade;
```

#### Results:

| bucket | customer_count |
| ------------- | ------------- | 
| 0 - 30 days |	49 |
| 30 - 60 days | 24 |
| 60 - 90 days | 35 |
| 90 - 120 days |	35 |
| 120 - 150 days | 43 |
| 150 - 180 days | 37 |
| 180 - 210 days | 24 |
| 210 - 240 days | 4 |
| 240 - 270 days | 4 |
| 270 - 300 days | 1 |
| 300 - 330 days | 1 |
| 330 - 360 days | 1 |

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

<br/>

```sql
WITH lead_plans AS (
  SELECT 
    s.customer_id,
    p.plan_id,
    LEAD(p.plan_id, 1) OVER (
      PARTITION BY s.customer_id
      ORDER BY s.start_date) AS next_plan
  FROM foodie_fi.subscriptions AS s 
  JOIN foodie_fi.plans AS p
    ON s.plan_id = p.plan_id
  WHERE DATE_PART('year', start_date) = 2020
)

SELECT 
  COUNT(customer_id) AS pro_to_basic
FROM lead_plans
WHERE plan_id = 2 AND next_plan = 1;
```

#### Results:

| pro_to_basic |
| ------------- | 
| 0 |

There were 0 customers who downgraded from a pro monthly to a basic monthly plan in 2020.
