# B. Runner and Customer Experience

## 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT
     ((registration_date - DATE '2021-01-01')::int / 7) + 1 AS week,
    COUNT(runner_id) AS runner_count
FROM runners
GROUP BY week
ORDER BY week;
```

Had to convert the start of the week as 2021-01-01 then cast it as an integer.

### Results:

