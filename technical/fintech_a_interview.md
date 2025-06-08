# 💼 FinTech A Interview Experience

## 🧪 Round: Technical  
**📅 Date**:  
**🏷️ Tags**: `SQL`, `Data Analysis`, `Window Functions`, `Aggregation`, `Retention`, `Intermediate SQL`, `Advanced SQL`

---

### ❓ Q1. Orders at least a week old but not delivered

In a dummy eCommerce company, each order has a max of two stages – *Placed* and *Delivered*.  
A row is inserted when the order is placed, and another row when it is delivered.

> Create a table of order IDs that are at least a week old but not yet delivered.

<details>
<summary>🧑‍💻 My Answer</summary>

```sql
WITH placed AS (
  SELECT order_id, datetime AS placed_time 
  FROM orders 
  WHERE status = 'Placed'
),
delivered AS (
  SELECT order_id, datetime AS delivered_time 
  FROM orders 
  WHERE status = 'Delivered'
)
SELECT 
  p.order_id, placed_time, delivered_time 
FROM 
  placed p 
  LEFT JOIN delivered d ON p.order_id = d.order_id
WHERE 
  delivered_time IS NULL 
  AND placed_time < CURRENT_TIME - INTERVAL '1 week';
```

</details>

<details>
<summary>🤖 GPT Answer</summary>

```sql
SELECT 
  order_id
FROM 
  orders o1
WHERE 
  status = 'Placed'
  AND NOT EXISTS (
    SELECT 1 
    FROM orders o2 
    WHERE o1.order_id = o2.order_id 
      AND o2.status = 'Delivered'
  )
  AND CURRENT_DATE - DATE(o1.datetime) >= 7;
```

</details>

**Tags**: `eCommerce`, `Status`, `Subquery`, `Filtering`, `Date Comparison`

---

### ❓ Q2. Second highest score per student

Given a list of student scores across multiple exams, find the second highest score for each student.

<details>
<summary>🧑‍💻 My Answer</summary>

```sql
WITH base AS (
  SELECT student, exam, scores, 
         DENSE_RANK() OVER (PARTITION BY student ORDER BY scores DESC) AS rk
)
SELECT * 
FROM base 
WHERE rk = 2;
```

</details>

<details>
<summary>🤖 GPT Answer</summary>

```sql
SELECT 
  student, 
  MAX(score) AS second_highest
FROM (
  SELECT 
    student, 
    score, 
    DENSE_RANK() OVER (PARTITION BY student ORDER BY score DESC) AS rnk
  FROM scores
) ranked
WHERE rnk = 2
GROUP BY student;
```

</details>

**Tags**: `Window Function`, `DENSE_RANK`, `Ranking`, `Subquery`

---

### ❓ Q3. Transactions Analysis

**Part 1**: Find the sum of all transaction amounts from the last 7 days
**Part 2**: Get the last 7 transactions for each user

<details>
<summary>🧑‍💻 My Answer</summary>

```sql
SELECT 
  SUM(amount)
FROM 
  orders
WHERE 
  datetime BETWEEN CURRENT_TIME - INTERVAL '7 days' AND CURRENT_TIME();
```

</details>

<details>
<summary>🤖 GPT Answer</summary>

```sql
-- Part 1: Total Amount (Last 7 Days)
SELECT 
  SUM(amount) AS total_last_7_days
FROM 
  transactions
WHERE 
  date >= CURRENT_DATE - INTERVAL 7 DAY;

-- Part 2: Last 7 Transactions per User
SELECT *
FROM (
  SELECT *, 
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY datetime DESC) AS rnk
  FROM transactions
) ranked
WHERE rnk <= 7;
```

</details>

**Tags**: `Aggregation`, `Window Function`, `ROW_NUMBER`, `Filtering`, `Recent Events`

---

### ❓ Q4. 28-Day Activity-Based Retention

> What % of users who performed an activity on a given date repeated it at least once in the next 28 days?
> What % did it at least *N* times?

<details>
<summary>🧑‍💻 My Answer</summary>

* Formula:
  `users who did same activity again in next 28 days / total users who did that activity on day 0`
* Mentioned using `RANGE BETWEEN` but did not include query.

</details>

<details>
<summary>🤖 GPT Answer</summary>

```sql
WITH base AS (
  SELECT user_id, DATE(datetime) AS activity_date, activity_type
  FROM user_activity
),
future_activity AS (
  SELECT 
    a.user_id,
    a.activity_date,
    a.activity_type,
    COUNT(*) AS repeat_count
  FROM base a
  JOIN base b 
    ON a.user_id = b.user_id 
    AND a.activity_type = b.activity_type
    AND b.activity_date > a.activity_date 
    AND b.activity_date <= DATE_ADD(a.activity_date, INTERVAL 28 DAY)
  GROUP BY a.user_id, a.activity_date, a.activity_type
),
retention AS (
  SELECT 
    activity_date, 
    activity_type,
    COUNT(DISTINCT user_id) AS retained_users
  FROM future_activity
  GROUP BY activity_date, activity_type
)
SELECT 
  b.activity_date,
  b.activity_type,
  ROUND(100.0 * COALESCE(r.retained_users, 0) / COUNT(DISTINCT b.user_id), 2) AS retention_rate
FROM base b
LEFT JOIN retention r 
  ON b.activity_date = r.activity_date 
  AND b.activity_type = r.activity_type
GROUP BY b.activity_date, b.activity_type, r.retained_users;
```

</details>

**Tags**: `Retention`, `JOIN`, `Group By`, `Time Series`, `Engagement Analysis` 