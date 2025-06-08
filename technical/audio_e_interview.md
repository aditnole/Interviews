# 💼 Audio E Interview Experience

## 🧪 Round: Experiment Design
**📅 Date**:  
**🏷️ Tags**: `Experiment Design`, `Recommendation System`, `A/B Testing`, `Engagement Metrics`, `SQL Analytics`

---

## ❓ Q1. Evaluate change in recommendation algorithm's impact on engagement.

---

<details>
<summary>🙋 My Answer</summary>

### 🎯 Hypothesis
- **Null (H₀):** No significant change in engagement.
- **Alternate (H₁):** Engagement improves with new recommendation system.

### 📏 Metrics
- **P0:** Completion rate of content (Primary)
- **P1:** Avg listening hours per user
- **P2:** Churn/drop-off

### 📐 Experiment Setup
- Define MDE with Product (e.g., 10pp increase in listening hours)
- α = 0.05, Power = 0.8
- Compute sample size accordingly
- Stratified sampling by:
  - Geography, Language, Subscribed status
  - Age, Gender, GMV
- Run A/B test:
  - **Control:** Old Recommendation
  - **Treatment:** New Recommendation

### 🧪 Result Scenarios:
- **Significant uplift:** Adopt new algorithm
- **No uplift:** Deep-dive into user cohorts for insights

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧮 Experiment Design

**Goal:** Determine whether the new recommendation engine improves user engagement.

#### Metrics:
- ✅ **Primary (P0):** Completion rate per content
- 📈 **Secondary (P1):** Avg listening time per user
- 🔁 **Drop-off rate**
- 🧠 Additional: # of series finished, avg session length, DAU

#### Statistical Setup:
- 🎯 **MDE:** Choose detectable change (e.g., +2pp in completion rate)
- ⚖️ **Significance Level:** α = 0.05
- 🔋 **Power:** 0.8
- 🧪 **Sample Size Calculation:** Based on MDE & baseline metrics

#### Sampling Strategy:
- Stratified sampling over:
  - 🎭 Demographics (Age, Gender)
  - 🌎 Geography
  - 🌐 Language
  - 🧑‍💻 Subscribed vs unsubscribed
  - 💰 GMV segments

</details>

---

## ❓ Q2. For each series, compute average number of days it takes users to complete 1 hour of listening.

---

<details>
<summary>🙋 My Answer</summary>

```sql
WITH base AS (
  SELECT date, user_id, series_id, listentime
  FROM events
),
cumm AS (
  SELECT e.date, e.user_id, e.series_id, e.listentime, b.listentime
  FROM events e
  LEFT JOIN base b
    ON e.user_id = b.user_id
    AND e.series_id = b.series_id
    AND b.date <= e.date
),
total_hours AS (
  SELECT user_id, series_id,
         SUM(b.listentime) AS cumulative_listening,
         MIN(date) AS start_date,
         MAX(date) AS end_date
  FROM cumm b
  GROUP BY user_id, series_id
  HAVING SUM(b.listentime) BETWEEN 60 AND 120
),
final AS (
  SELECT user_id, series_id,
         DATE_DIFF('day', start_date, end_date) AS days_to_complete
  FROM total_hours
)
SELECT series_id, AVG(days_to_complete) AS avg_days_to_1hr
FROM final
GROUP BY series_id;
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧮 Objective:

Compute how long it takes (in days) for users to accumulate at least 1 hour (60 mins) of listening per series.

### 🧠 Approach:

1. **Cumulative Listening:** Track user listening time per series per day.
2. **Threshold Condition:** Filter users who hit 60–120 mins in total.
3. **Date Delta:** Calculate the days between first and last listening date.
4. **Aggregate:** Take average across users per series.

```sql
WITH cumulative AS (
  SELECT user_id, series_id, date,
         SUM(listentime) OVER (
           PARTITION BY user_id, series_id
           ORDER BY date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
         ) AS cum_listening
  FROM events
),
qualified AS (
  SELECT user_id, series_id, MIN(date) AS start_date,
         MAX(date) AS end_date
  FROM cumulative
  WHERE cum_listening >= 60 AND cum_listening <= 120
  GROUP BY user_id, series_id
)
SELECT series_id,
       AVG(DATE_DIFF('day', start_date, end_date)) AS avg_days
FROM qualified
GROUP BY series_id;
```

</details> 