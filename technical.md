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


## 🧠 Learnings

* Prepare window functions thoroughly: `DENSE_RANK`, `ROW_NUMBER`, etc.
* Always clarify assumptions in ambiguous cohort-style questions.
* Brush up retention logic, especially for product metrics interviews.

---

# 💼 EdTech B Interview Experience

## 🧪 Round: Product Strategy
**👤 Interviewer**: Michaela  
**📅 Date**:  
**🏷️ Tags**: `Product Strategy`, `A/B Testing`, `Retention`, `Experiment Design`, `Subscription Models`, `Churn Analysis`, `EdTech`, `Behavioral Analytics`

---

### ❓ Q1. How would you design an experiment to improve learner retention on Preply's subscription model?

**📄 Background**:
- Preply operates on a subscription model where users subscribe to a tutor for a set number of hours per month (e.g. 4 hours within a 28-day cycle).
- Tutor-based pricing model — users select a specific tutor.
- Tutors do **not** get paid for trial lessons (impacts supply-side behavior).
- Preply wants to test if offering **trial lessons with new tutors** can improve learner retention.

---

<details>
<summary>🧑‍💻 My Answer</summary>

- Define revenue as:  
  `Revenue = Retention × Price`
- Current Model (Control):
  - Subscription with fixed hours and one selected tutor
- Treatment Group:
  - Subscription (4 hours) + 2 **free trials** with other tutors
- Risk:
  - Tutors not paid for trials → supply-side dissatisfaction
- User Segmentation:
  - Regions: Spain, India
  - Segments: Age, Gender, Learning Goal, New User
- Metrics:
  - Churn rate (aim for 25pp reduction)
  - # of users who took trials and still churned
  - Net revenue comparison:  
    `$Revenue_Treatment - Cost_of_Trials`
- Success:
  - Reduce churn from 27% to ~2%
- Track clickstream events to see who actually *used* the trial slots
- Analyze **when** in the lifecycle the trial lessons are consumed (early, mid, late)
</details>

<details>
<summary>🤖 GPT Answer</summary>

**Experiment Framework:**
- **Hypothesis**: Offering 2 free trial lessons with new tutors increases learner engagement and retention.
  
**Groups**:
- **Control**: Subscription with 4 hours, single tutor (status quo)
- **Treatment**: Subscription with 4 hours + 2 free trials with different tutors

**Key Variables to Track**:
- **Primary Metric**:  
  - Retention after 28 days (binary or rolling retention)
  - Churn rate comparison (absolute and relative)
- **Secondary Metrics**:
  - NPS / CSAT for trial users
  - Tutor churn rate (to monitor dissatisfaction)
  - Trial usage rate (conversion of trial → paying relationship)

**Segmentation**:
- New vs existing users  
- Geography: Spain, India  
- Intent: casual learners vs test-prep users  
- Past activity: Active vs dormant learners

**Design Notes**:
- Ensure equal split across subgroups (stratified sampling)
- Define minimum detectable effect and estimate sample size
- Define success threshold (e.g. 10% relative improvement in retention)
- Monitor for novelty effects and teacher-side fatigue

**Next Steps**:
- Build a dashboard to track usage of trials and retention
- Tag user cohorts via event tracking (clicks, trials started, sessions post-trial)
</details>

**Tags**: `Retention`, `A/B Test`, `Learner Behavior`, `Trial Design`, `Subscription Risk`  
**Note**: Could benefit from a 3rd group (trial-only without subscription) to isolate trial effectiveness independently.

---

### ❓ Q2. When do users take their 2 trial lessons? How can we use that information?

**📄 Background**:
Understanding *when* users take their 2 free trial lessons could help personalize nudges or optimize the UX. The hypothesis is that trial timing affects retention or satisfaction.

---

<details>
<summary>🧑‍💻 My Answer</summary>

- Define a metric: **Active Learning Hours**
- Segment trial users into early, mid, and late trial engagement
- Use clickstream data to detect:
  - When the user saw the "free trial" banner or button
  - When they actually scheduled and completed the trials
- Hypothesize: early trials increase engagement + conversion
</details>

<details>
<summary>🤖 GPT Answer</summary>

**Step-by-step Analysis**:

1. **Feature Engineering**:
   - `trial_day_offset = DATEDIFF(trial_lesson_date, subscription_start_date)`
   - Buckets: 
     - Early (0–7 days)
     - Mid (8–21 days)
     - Late (22–28 days)

2. **Exploratory Analysis**:
   - Compare churn rates across buckets
   - Analyze how many trials convert into paid recurring sessions

3. **Clickstream Funnel**:
   - `Trial_Clicked → Trial_Booked → Trial_Completed`
   - Conversion drop-offs at each stage

4. **Retention Curve**:
   - Overlay trial timing onto 28-day retention curve

5. **Output Metrics**:
   - `% of users using trials in first 7 days`
   - `Churn_early_trial - Churn_late_trial`
   - NPS by trial timing

**Bonus**:
- Run logistic regression to quantify impact of trial timing on churn
</details>

**Tags**: `Clickstream`, `Trial Timing`, `Behavior Analysis`, `Lifecycle Events`, `Retention Modeling`  
**Note**: Consider A/B testing nudges to encourage earlier usage of trial lessons.

---




# 💼 Payment C Interview Experience

## 🧪 Round: Product Analytics
**📅 Date**:  
**🏷️ Tags**: `Checkout Funnel`, `A/B Testing`, `Buy Now Pay Later`, `Conversion Uplift`, `Experimentation`, `Causal Inference`, `Diff-in-Diff`

---

### ❓ Q1. How would you analyze the impact of introducing a "Buy Now Pay Later" (BNPL) feature in the checkout flow?

---

**🧭 Background**

Checkout Flow:
```

Login → Review Page → Final Payment

```

Review Page includes:
- Payment option selection
- Address
- Option for "Buy Now, Pay Later" (BNPL)

The experiment involved adding a BNPL option in the Review Page to test for uplift in conversions.

---

<details>
<summary>🧑‍💻 My Answer</summary>

- Setup:
  - **P_T** (Treatment): Saw BNPL option
    - T1: Opted for BNPL → 20 users
    - T2: Didn't opt → 180 users
    - Total: 200
    - Completion Rate (T1 + T2) = 89%
    - T1 (opted): 85%
    - T2 (didn't opt): 80%
  - **Control Group** (No BNPL shown): 200 users, 88% completion

- Final Adoption Rate: 20% of treatment users opted BNPL
- Observations:
  - T1: Completion = 85% → lower than total treatment
  - T2: Completion = 80% → GMV is high, so skewed results
  - Chi-square test for significance
  - Confounding: High GMV users may behave differently
  - Suggested Diff-in-Diff approach to account for pre-post variation
- Metrics:
  - **P0**: Conversion/transaction rate
  - **P1**: Time spent per option in Review page
- Behavioral Events to track:
  - Fill event
  - Option select (Adoption)
  - Hover time (Interest)
- Suggest penalizing/dropping users with excessive delay or bounce
</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 📐 Experiment Design & Analysis

**Hypothesis**: Introducing BNPL will increase overall conversion rate without decreasing GMV per transaction.

---

#### 📊 Experimental Setup

| Group       | Saw BNPL | Opted BNPL | Completed Checkout | Conversion Rate |
|-------------|----------|------------|---------------------|-----------------|
| Treatment   | ✅        | 20         | 170 (out of 200)    | 85%             |
| Control     | ❌        | 0          | 176 (out of 200)    | 88%             |

- BNPL **adoption rate** = 20/200 = 10%
- Within treatment:
  - T1 (BNPL opted): 85% completion
  - T2 (Did not opt): 80% completion

---

### ⚙️ Recommendations for Analysis

1. **Intention-to-Treat (ITT)**:
   - Compare overall conversion of Control vs Treatment (regardless of adoption)
   - Treatment = 85%, Control = 88% → Slight drop, not enough evidence BNPL improves completion.

2. **ATT (Average Treatment effect on the Treated)**:
   - Focus on T1 vs matched users in control (propensity score matching if needed)
   - Look for lift in GMV or retention among BNPL adopters

3. **Time-Based Metrics**:
   - Avg time spent per option
   - Hover events: Implicit interest
   - Use event logs to capture friction in decision-making

4. **Causal Framework**:
   - Pre/post period comparison → **Diff-in-Diff** analysis
   - Control for GMV bands (low, medium, high)

5. **Confounding Controls**:
   - Users with higher GMV may naturally have more friction → Use matching or regression adjustments
   - Add covariates like region, device, session length

6. **Metrics to Watch**:
   - Primary (P0): Conversion rate
   - Secondary (P1): Time on Review page per section
   - Tertiary: GMV per session, drop-offs post-BNPL

---

### ✅ Recommendation

- BNPL adoption is still low (10%) — consider surfacing more upfront
- Slight reduction in conversion for those shown BNPL could suggest added complexity
- Suggest A/B test refinement: break BNPL into clear, educational UI vs default toggle

</details>

---

**Tags**: `A/B Test`, `BNPL`, `Conversion`, `Hover Events`, `Causal Impact`, `Checkout UX`, `Chi-Square`, `ATT`, `Diff-in-Diff`  
**Note**: The effect of the feature may be more pronounced in certain user segments (e.g., mobile vs desktop, low-income regions). Consider further segmentation analysis.


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
```


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

---


# 💼 E-commerce G Interview Experience

## 🧪 Round: NLP & Data Processing
**📅 Date**:  
**🏷️ Tags**: `Customer Support`, `PII Data`, `LLM`, `NLP`, `Data Preprocessing`, `Regex`

---

## ❓ Q1. How to preprocess user queries and remove Personally Identifiable Information (PII)?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **PII Removal:**
   - **Account Number:** Match patterns like account number or bank details and replace them.
   - **IFSC Code & Branch:** Regex to filter out IFSC and branch code details.
   - **Dollar Amount:** Regex pattern for detecting currency symbols or amounts.

2. **Message Preprocessing:**
   - **Remove Repetition:** Detect repeated words (e.g., "where where is my deposit?!") and correct.
   - **Remove Extra Spaces/Newlines:** Clean up unneeded spaces and line breaks.
   - **Emoji Removal:** Detect and remove emojis using regex.
   - **Spacy for NLP:** Use SpaCy to process the message and extract entities (e.g., account details, amounts).

### 📝 Code Example

```python
import re
import spacy

# Load the SpaCy model
nlp = spacy.load('en_core_web_sm')

class PreprocessMessage:
    def __init__(self, msg):
        self.msg = msg

    def remove_pii(self):
        # Example regex for account numbers
        account_pattern = r"(\d{2}-\d{4}-\d{4})"  # Example pattern for account number
        self.msg = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.msg)

        # Remove IFSC Code
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.msg = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.msg)

        # Remove Amounts
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.msg = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.msg)

        return self.msg

    def preprocess(self):
        self.msg = self.remove_pii()
        # Remove extra spaces or newlines
        self.msg = re.sub(r'\s+', ' ', self.msg)
        # Process with SpaCy
        doc = nlp(self.msg)
        return doc

# Example usage
message = "Where is my deposit of $500 made yesterday (Account Number: 1234-5678-1234)?"
preprocessor = PreprocessMessage(message)
processed_msg = preprocessor.preprocess()
print(processed_msg)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

1. **PII Removal:**

   * **Account Information:** Apply regex patterns to detect account numbers, IFSC codes, or bank-related data and mask them.
   * **Amount Detection:** Detect any monetary values or amounts in the message and replace them.

2. **Message Preprocessing:**

   * Clean up any noisy elements like emojis, extra spaces, and redundant phrases.
   * Use SpaCy's NLP capabilities to tokenize, recognize named entities, and ensure proper cleanup of the message.

```python
import re
import spacy

# Load SpaCy model
nlp = spacy.load("en_core_web_sm")

class TextPreprocessing:
    def __init__(self, message):
        self.message = message

    def remove_pii(self):
        # Example pattern for account number
        account_pattern = r"\b\d{4}-\d{4}-\d{4}-\d{4}\b"  # Simplified regex
        self.message = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.message)

        # IFSC Code pattern
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.message = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.message)

        # Dollar Amount
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.message = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.message)

        return self.message

    def clean_message(self):
        # Remove unwanted spaces and newlines
        self.message = re.sub(r'\s+', ' ', self.message).strip()
        doc = nlp(self.message)
        return doc

# Usage Example
query = "Where is my deposit of $1000 made yesterday (Account Number: 1234-5678-9876)?"
processor = TextPreprocessing(query)
cleaned_message = processor.clean_message()
print(cleaned_message)
```

</details>

---

## ❓ Q2. How to process user data (transaction data) and identify product categories?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **Transaction Aggregation:**

   * **Group by product and transaction ID:** Aggregate transactions for each user.
   * **Concatenate product names:** Create a list of products per user per transaction.

2. **Example User Data**:

   * Aggregated product list by transaction.
   * Example: User A bought "Perfume" and "Chocolate", which are grouped into one transaction.

### 📝 Code Example

```python
import pandas as pd

# Sample Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group products by transaction
grouped = df.groupby(['txn_id'])['product'].apply(lambda x: ', '.join(x)).reset_index()

# Display grouped transaction data
print(grouped)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

* **Group Data by Transaction ID:** Create a list of products per transaction.
* **Concatenate Products:** Combine product names using a delimiter (e.g., commas).
* **Transaction-level Aggregation:** This allows understanding of which products are purchased together.

```python
import pandas as pd

# Sample Transaction Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group and Concatenate Products per Transaction
grouped_products = df.groupby(['txn_id'])['product'].apply(', '.join).reset_index()
print(grouped_products)
```

</details>

---

## ❓ Q3. How to track transaction pairs and calculate total transaction count?

---

<details>
<summary>🙋 My Answer</summary>

1. **Set Hashmap:**

   * Create a dictionary of product pairs and track their occurrence.
   * Update counts for each pair as they appear in the data.

### 📝 Code Example

```python
# Create a dictionary to store product pair frequencies
product_pairs = {}

def update_count(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                if pair in product_pairs:
                    product_pairs[pair] += 1
                else:
                    product_pairs[pair] = 1

# Apply the function
update_count(df)

# Display result
print(product_pairs)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

1. **Track Pair Frequencies:**

   * Use a dictionary to count product pairs across transactions.
   * Store counts for each pair and update on encountering new transaction pairs.

```python
# Sample transaction data
product_pairs = {}

def track_pairs(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                product_pairs[pair] = product_pairs.get(pair, 0) + 1

# Run the function to count pairs
track_pairs(df)
print(product_pairs)
```

</details>

---
# 💼 EdTech H Interview Experience

## 🧪 Round: NLP & Data Processing
**📅 Date**:  
**🏷️ Tags**: `Customer Support`, `PII Data`, `LLM`, `NLP`, `Data Preprocessing`, `Regex`

---

## ❓ Q1. How to preprocess user queries and remove Personally Identifiable Information (PII)?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **PII Removal:**
   - **Account Number:** Match patterns like account number or bank details and replace them.
   - **IFSC Code & Branch:** Regex to filter out IFSC and branch code details.
   - **Dollar Amount:** Regex pattern for detecting currency symbols or amounts.

2. **Message Preprocessing:**
   - **Remove Repetition:** Detect repeated words (e.g., "where where is my deposit?!") and correct.
   - **Remove Extra Spaces/Newlines:** Clean up unneeded spaces and line breaks.
   - **Emoji Removal:** Detect and remove emojis using regex.
   - **Spacy for NLP:** Use SpaCy to process the message and extract entities (e.g., account details, amounts).

### 📝 Code Example

```python
import re
import spacy

# Load the SpaCy model
nlp = spacy.load('en_core_web_sm')

class PreprocessMessage:
    def __init__(self, msg):
        self.msg = msg

    def remove_pii(self):
        # Example regex for account numbers
        account_pattern = r"(\d{2}-\d{4}-\d{4})"  # Example pattern for account number
        self.msg = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.msg)

        # Remove IFSC Code
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.msg = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.msg)

        # Remove Amounts
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.msg = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.msg)

        return self.msg

    def preprocess(self):
        self.msg = self.remove_pii()
        # Remove extra spaces or newlines
        self.msg = re.sub(r'\s+', ' ', self.msg)
        # Process with SpaCy
        doc = nlp(self.msg)
        return doc

# Example usage
message = "Where is my deposit of $500 made yesterday (Account Number: 1234-5678-1234)?"
preprocessor = PreprocessMessage(message)
processed_msg = preprocessor.preprocess()
print(processed_msg)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

1. **PII Removal:**

   * **Account Information:** Apply regex patterns to detect account numbers, IFSC codes, or bank-related data and mask them.
   * **Amount Detection:** Detect any monetary values or amounts in the message and replace them.

2. **Message Preprocessing:**

   * Clean up any noisy elements like emojis, extra spaces, and redundant phrases.
   * Use SpaCy's NLP capabilities to tokenize, recognize named entities, and ensure proper cleanup of the message.

```python
import re
import spacy

# Load SpaCy model
nlp = spacy.load("en_core_web_sm")

class TextPreprocessing:
    def __init__(self, message):
        self.message = message

    def remove_pii(self):
        # Example pattern for account number
        account_pattern = r"\b\d{4}-\d{4}-\d{4}-\d{4}\b"  # Simplified regex
        self.message = re.sub(account_pattern, "[REDACTED_ACCOUNT]", self.message)

        # IFSC Code pattern
        ifsc_pattern = r"\b[A-Z]{4}\d{7}\b"
        self.message = re.sub(ifsc_pattern, "[REDACTED_IFSC]", self.message)

        # Dollar Amount
        amount_pattern = r"\$\d+(\.\d{2})?"
        self.message = re.sub(amount_pattern, "[REDACTED_AMOUNT]", self.message)

        return self.message

    def clean_message(self):
        # Remove unwanted spaces and newlines
        self.message = re.sub(r'\s+', ' ', self.message).strip()
        doc = nlp(self.message)
        return doc

# Usage Example
query = "Where is my deposit of $1000 made yesterday (Account Number: 1234-5678-9876)?"
processor = TextPreprocessing(query)
cleaned_message = processor.clean_message()
print(cleaned_message)
```

</details>

---

## ❓ Q2. How to process user data (transaction data) and identify product categories?

---

<details>
<summary>🙋 My Answer</summary>

### 🛠️ Approach

1. **Transaction Aggregation:**

   * **Group by product and transaction ID:** Aggregate transactions for each user.
   * **Concatenate product names:** Create a list of products per user per transaction.

2. **Example User Data**:

   * Aggregated product list by transaction.
   * Example: User A bought "Perfume" and "Chocolate", which are grouped into one transaction.

### 📝 Code Example

```python
import pandas as pd

# Sample Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group products by transaction
grouped = df.groupby(['txn_id'])['product'].apply(lambda x: ', '.join(x)).reset_index()

# Display grouped transaction data
print(grouped)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

### 🧑‍💻 Approach

* **Group Data by Transaction ID:** Create a list of products per transaction.
* **Concatenate Products:** Combine product names using a delimiter (e.g., commas).
* **Transaction-level Aggregation:** This allows understanding of which products are purchased together.

```python
import pandas as pd

# Sample Transaction Data
data = {
    'user': ['a', 'a', 'b', 'c', 'c', 'd', 'd', 'd', 'a'],
    'product': ['perfume', 'chocolate', 'coke', 'coke', 'chocolate', 'perfume', 'coke', 'chocolate', 'perfume'],
    'txn_id': [1234, 1234, 1235, 1236, 1236, 1237, 1237, 1237, 1238]
}

df = pd.DataFrame(data)

# Group and Concatenate Products per Transaction
grouped_products = df.groupby(['txn_id'])['product'].apply(', '.join).reset_index()
print(grouped_products)
```

</details>

---

## ❓ Q3. How to track transaction pairs and calculate total transaction count?

---

<details>
<summary>🙋 My Answer</summary>

1. **Set Hashmap:**

   * Create a dictionary of product pairs and track their occurrence.
   * Update counts for each pair as they appear in the data.

### 📝 Code Example

```python
# Create a dictionary to store product pair frequencies
product_pairs = {}

def update_count(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                if pair in product_pairs:
                    product_pairs[pair] += 1
                else:
                    product_pairs[pair] = 1

# Apply the function
update_count(df)

# Display result
print(product_pairs)
```

</details>

---

<details>
<summary>🤖 GPT Answer</summary>

1. **Track Pair Frequencies:**

   * Use a dictionary to count product pairs across transactions.
   * Store counts for each pair and update on encountering new transaction pairs.

```python
# Sample transaction data
product_pairs = {}

def track_pairs(df):
    for txn_id, group in df.groupby('txn_id'):
        products = group['product'].tolist()
        for i in range(len(products)):
            for j in range(i + 1, len(products)):
                pair = tuple(sorted([products[i], products[j]]))
                product_pairs[pair] = product_pairs.get(pair, 0) + 1

# Run the function to count pairs
track_pairs(df)
print(product_pairs)
```

</details>

---
# 💼 SaaS I Interview Experience

## 🧪 Round: Product Analytics
**📅 Date**:  
**🏷️ Tags**: `User Analytics`, `Retention`, `Experiment Design`, `Product Metrics`, `User Behavior`

---

### ❓ Q1. Drop in Monthly Active Users: Confluence - Deep Dive

<details>
<summary>🧑‍💻 My Answer</summary>

### Data Analysis and Steps to Investigate the Drop:

1. **Trend Analysis**:
   * **MoM (Month over Month)**, **WoW (Week over Week)**, and **Daily**:
     * Look for any spikes or drops in user activity, especially at the beginning or end of the month/week.
   * **Diff-in-Diff Analysis**:
     * Compare the behavior of users before and after a certain event or change.

2. **Localization of Drop**:
   * Identify the **most common user attributes** where the drop is significant (e.g., **role type**, **geography**).
   * Examine the differences between **active** and **inactive** users.

3. **Key Segments**:
   * **Users dropping**: Focus on high GMV users versus low GMV users.
   * **Geography**: Identify any regional drops that could be contributing to the overall decline.

4. **Check for Recent Changes**:
   * **Pricing changes**: Investigate if any pricing adjustments were made recently.
   * **Sign-up/Onboarding Changes**: Review changes in user engagement patterns.

5. **Event Type Analysis**:
   * Investigate a **drop in event types** such as:
     * Fewer **pages created**.
     * Fewer **members added** to spaces or teams.

6. **New vs Existing User Analysis**:
   * Compare drop rates
   * Analyze retention patterns
</details>

<details>
<summary>🤖 GPT Answer</summary>

### Comprehensive Analysis Framework

1. **Time Series Analysis**:
   * Daily, weekly, monthly trends
   * Seasonality detection
   * Anomaly identification

2. **User Segmentation**:
   * By user type
   * By geography
   * By usage patterns

3. **Feature Impact**:
   * Recent feature rollouts
   * UI/UX changes
   * Performance issues

4. **External Factors**:
   * Market conditions
   * Competitor actions
   * Industry trends

5. **Recommendations**:
   * Short-term fixes
   * Long-term solutions
   * Monitoring improvements
</details>

---

### ❓ Q2. New Onboarding Video for New Users

**📄 Background**: Design an experiment to measure the impact of a new onboarding video on user retention.

<details>
<summary>🧑‍💻 My Answer</summary>

### Experiment Design:

1. **Hypothesis**:
   * New video improves retention by 2pp
   * Better engagement with key features

2. **Metrics**:
   * P0: Retention rate
   * P1: Video completion rate
   * P2: Average watch time
   * Guardrail: Feature usage

3. **Setup**:
   * Control: No video
   * Treatment: New video
   * Randomization: User-level

4. **Analysis**:
   * Statistical significance
   * Segment analysis
   * Long-term impact
</details>

<details>
<summary>🤖 GPT Answer</summary>

### Detailed Experiment Framework

1. **Sample Size Calculation**:
   * MDE: 2pp
   * Power: 0.8
   * Alpha: 0.05

2. **Implementation**:
   * A/B test setup
   * Tracking implementation
   * Quality checks

3. **Analysis Plan**:
   * Statistical testing
   * Segment analysis
   * Long-term tracking

4. **Success Criteria**:
   * Retention improvement
   * Feature adoption
   * User satisfaction
</details>

---

### ❓ Q3. Users with High Likelihood of Going Dormant

**📄 Background**: Build a model to predict users likely to become dormant (less than 10 events in past month).

<details>
<summary>🧑‍💻 My Answer</summary>

### Modeling Approach:

1. **Features**:
   * User demographics
   * Usage patterns
   * Engagement metrics
   * Historical behavior

2. **Model Selection**:
   * LightGBM classifier
   * Feature importance analysis
   * Regular model updates

3. **Evaluation**:
   * Precision/Recall
   * F1 Score
   * Business impact
</details>

<details>
<summary>🤖 GPT Answer</summary>

### Comprehensive Modeling Strategy

1. **Feature Engineering**:
   * Time-based features
   * Engagement metrics
   * User attributes
   * Historical patterns

2. **Model Development**:
   * LightGBM implementation
   * Hyperparameter tuning
   * Cross-validation

3. **Deployment**:
   * Real-time scoring
   * Monitoring
   * Regular updates

4. **Success Metrics**:
   * Model performance
   * Business impact
   * User retention
</details>

## 🧠 Learnings

* Focus on both statistical significance and business impact
* Consider multiple user segments in analysis
* Balance short-term fixes with long-term solutions
* Regular monitoring and iteration is key