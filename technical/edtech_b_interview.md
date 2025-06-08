# 💼 EdTech B Interview Experience

## 🧪 Round: Product Strategy
**👤 Interviewer**: mmmmm  
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