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