# 💼 RideShare D Interview Experience

## 🧪 Round: Data Analytics
**📅 Date**:  
**🏷️ Tags**: `SQL`, `User Behavior`, `Experiment Design`, `Driver Experience`, `A/B Testing`, `Churn`, `Trend Analysis`, `Cancellation Policy`, `Hypothesis Testing`

---

## ❓ Q1. Find the second order_id for users who have made at least 2 purchases.

---

<details>
<summary>🙋 My Answer</summary>

```sql
WITH base AS (
  SELECT user_id, order_id,
         RANK() OVER (PARTITION BY user_id ORDER BY timestamp) AS rk
  FROM orders
)
SELECT user_id, order_id
FROM base
WHERE rk = 2
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧮 Explanation:

* Use `RANK()` or `ROW_NUMBER()` to rank orders by timestamp per user.
* Filter users whose rank = 2 to get the second order.

### ✅ Assumptions:

* A user can have multiple orders.
* We only need those users who have more than one order.

</details>

---

## ❓ Q2. For each order, find how many orders were placed by the same user in the last 4 days.

---

<details>
<summary>🙋 My Answer</summary>

```sql
SELECT user_id, order_id,
       COUNT(*) OVER (
         PARTITION BY user_id
         ORDER BY timestamp
         RANGE BETWEEN INTERVAL '4 days' PRECEDING AND CURRENT ROW
       ) AS rolling_4d_count
FROM orders
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧮 Explanation:

* This uses a rolling count of user's orders in the last 4 days using a window function.
* You could also do this with a **self join** if `RANGE` window isn't supported:

```sql
SELECT a.user_id, a.order_id, COUNT(b.order_id) AS count_last_4d
FROM orders a
JOIN orders b ON a.user_id = b.user_id
            AND b.timestamp BETWEEN a.timestamp - INTERVAL '4 days' AND a.timestamp
GROUP BY a.user_id, a.order_id
```

</details>

---

## ❓ Q3. How to measure and track **Driver Experience**?

---

<details>
<summary>🙋 My Answer</summary>

### Possible Metrics:

* Job Completion Rate
* Driver Churn
* Ride Ratings (but sparse)
* Engagement with other platforms
* Loans taken from Uber (as engagement proxy)
* Sentiment via surveys
* Presence of competing apps (e.g., Ola)

### Trends:

* Track rolling average (7, 30, 60-day windows)
* Calculate STD, flag if beyond ±2σ
* Localize by geography
* If drop, check:

  * Ongoing experiments
  * Seasonality
  * Pricing

### Natural vs Non-natural shift:

* Forecast via time series model
* Compare predicted vs actual (residuals)
* Compute p-value to test significance

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🔍 Driver Experience Measurement

**Quantitative Metrics**

* ✅ Job Completion Rate (primary)
* 🚫 Cancellation Rate
* 🔁 Acceptance Rate
* 💸 Payout Stability
* 📉 Churn (monthly active vs retained drivers)

**Qualitative Indicators**

* 📋 Surveys (NPS, sentiment analysis)
* 📱 Competitor app presence
* 📉 Decreased engagement (fewer logins, job accepts)

### 🧪 Trend Stability

* Use `rolling_avg`, `rolling_stddev`
* Threshold-based alerting (e.g., drops > 3σ)
* Segment by city/region for root cause isolation

**Test for Statistical Drift**

* TS Forecast vs Actual
* Hypothesis Test: `H0: No difference`, `H1: Significant drop`

</details>

---

## ❓ Q4. Design an Experiment to evaluate **Removing Cancellation Fees**.

---

<details>
<summary>🙋 My Answer</summary>

### Objective:

Measure impact of waiving cancellation charges on new user signups and other metrics.

### Hypotheses:

* **H0**: No impact on P0
* **H1**: Improvement in user experience → signups, completion rate

### Metrics:

* **P0**: New User Signups
* **P1**: Ride Completion Rate = `1 - Cancellation Rate`
* **P2**: GMV per User

### Design:

* Sample users based on segments (GMV, Premium, Gender)
* Stratified Sampling + A/A test validation
* Control: Users charged cancellation fee
* Treatment: Waiver applied

### Analysis:

* MDE estimation
* Daily new user baseline
* Duration based on power analysis

### Outcome Scenarios:

* If uplift → Assess cost tradeoff
* No uplift → Deep dive by cohort

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧪 Experimental Design

**Groups**:

* Control: Existing users with cancellation fee
* Treatment: Cancellation fee waived

**Segmentation**:

* GMV Tier
* Subscription (Uber Premium)
* Region
* Demographics

### Key Metrics

| Metric | Description           |
| ------ | --------------------- |
| P0     | New User Signups      |
| P1     | 1 - Cancellation Rate |
| P2     | GMV per User          |

### Statistical Setup:

* Power: 80%, Alpha: 0.05
* Run A/A test to validate randomization
* MDE Estimation + Sample Size Calculation

### Risk Mitigation:

* Monitor outliers in daily signups
* Pause if there's a major drop or confounding bug
* Use CUPED if pre-treatment covariates are available

</details> 