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