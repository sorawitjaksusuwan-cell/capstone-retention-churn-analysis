# Retention Curve & Early Churn Warning Signals — Capstone Project

End-to-end analysis for a streaming subscription service ("PlayNow"), identifying when and why subscribers churn, to support a targeted retention campaign — Data Analytics Bootcamp Capstone (Sprint 4, group project).

## Business Problem

PlayNow has been growing steadily in sign-ups, but the Head of Retention has a more pressing question: **how long do subscribers actually stay, and where does churn concentrate?** The team wants to see retention broken down by cohort, and to identify early warning signals before a subscriber cancels — so retention efforts can be targeted rather than blanket.

**Stakeholders:** Marketing, Product, and Management teams
**Decision this analysis supports:** which campaigns or plan benefits to adjust first to reduce cancellations in the coming months

## Business & Analytical Questions

| Business Question | Analytical Questions |
|---|---|
| **BQ1:** How does the retention curve trend differ across cohorts by month_index? | Retention rate per cohort per month_index; where the curve drops most sharply |
| **BQ2:** How does usage behavior before churn differ between churned and retained users? | Gap in avg. watch minutes and avg. sessions between churned vs. retained; thresholds that flag at-risk behavior |
| **BQ3:** Which cohort/plan tier segments carry the highest churn risk, and what are the warning signals? | Churn rate by plan tier (bar chart); relationship between month_index and churn (t-test); correlation between usage metrics |

## Metrics

| Metric | Definition |
|---|---|
| Retention Rate | % of an initial cohort still `active_flag = 1` at each month_index |
| Avg. Watch Minutes (gap) | % difference in average watch minutes, churned vs. retained |
| Avg. Sessions (gap) | % difference in average session count, churned vs. retained |
| Sessions–Watch Time Correlation | Relationship between sessions and watch minutes, by churn status |
| Churn Rate by Segment | % of users churned per month_index, by plan tier / region / cohort |

## Data & Tools

- **Tools:** Python (pandas, scipy — Welch's t-test, correlation), cohort analysis
- **Data:** Subscriber-level streaming activity (`stream_retention_churn.csv`) — cohort month, monthly activity, region, plan tier, watch minutes, sessions, churn flag
- **Data quality issues found and resolved:**
  - 20 duplicate rows
  - Non-standardized `region` and `plan_tier` values (mixed Thai/English, inconsistent casing)
  - Inconsistent date formats across `cohort_month`, `activity_month`, `cancel_date`
  - Negative values in `sessions` / `watch_minutes`
  - 86 rows with conflicting `active_flag` vs. `churned_flag`
  - Deduplicated to one row per subscriber (anchored to each subscriber's first cohort) before behavior analysis

## Approach

1. Framed the business problem into 3 BQs and broke each into specific, testable AQs
2. Audited and cleaned the raw data (see data quality issues above), logging every transformation
3. Built a cohort retention curve (6 cohorts, May–Oct 2025) to answer BQ1
4. Compared usage behavior (watch minutes, sessions) between churned and retained users — overall, and split by region and plan tier — to answer BQ2
5. Defined risk thresholds from the behavior gap, then quantified churn rate by plan tier, region, and cohort to answer BQ3
6. Validated the key finding with a Welch's t-test, and backed every chart with a traceable summary table (documented in the proposal's appendix) so results could be audited back to raw counts
7. Worked in 3 team checkpoints: (1) data audit + cleaning + BQ1, (2) BQ2 + BQ3 analysis, (3) synthesis into guardrails, recommendations, and final presentation

## Key Insight

**Retention drops hardest immediately after sign-up:**
- Across all 6 cohorts, retention starts at 15–25% in month 1 and falls to just 9–13% by month 2 — the steepest drop in the entire curve
- Retention then stabilizes through months 2–4 (roughly 6–13%) before an unusual spike to 42–54% at month 5 across all cohorts (worth further investigation — possibly a billing-cycle or re-engagement effect rather than organic retention)

**Usage behavior clearly separates churned from retained users:**
- Retained users have visibly higher average watch minutes and session counts than churned users — low activity is a leading indicator of churn
- Risk thresholds identified: **watch minutes < 122 min** and **sessions < 3** per period
- Sessions and watch minutes are strongly correlated (r = 0.91), so both should be monitored together as a combined engagement signal

**Risk concentrates in specific segments:**
- Highest-risk region: **West**
- Highest-risk plan tier: **Basic**
- Highest-risk cohort: **2025-09**
- Premium-tier users show meaningfully higher engagement than Basic/Standard, even within the churned group
- **Welch's t-test** on average watch minutes, Basic vs. Premium tier: p < 0.001, reject H0 — the difference is statistically significant, confirming plan tier is a real factor in usage behavior, not noise
  - ⚠️ *Note: the test's stated direction (H1: Basic > Premium) runs opposite to the descriptive charts, which show Premium engagement higher. Worth double-checking the test setup (e.g., which group was coded as which) before presenting this stat — the "significant difference" finding is solid, but the direction should be confirmed.*

## Recommendations

1. **Upgrade campaigns for Basic-tier subscribers** showing risk signals — highlight Premium benefits (quality, exclusive content, no ads); consider a rewards/usage-milestone program to build engagement habits
2. **Region-specific campaigns for West** (and secondarily Central) — investigate local factors (content gaps, competitor promotions, economic factors) before designing localized offers
3. **Early warning system** — automated triggers when a subscriber's usage drops below the risk thresholds (122 min / 3 sessions), paired with personalized re-engagement messages
4. **A/B test retention campaigns** (messaging, offer format, channel) before full rollout, rather than assuming any single intervention will work

## Risks & Limitations

- Region and plan tier values required heavy standardization (mixed Thai/English, inconsistent casing) — mitigated with a mapping dictionary and pre/post validation, but new unseen variants could still slip through in future data pulls
- Newer cohorts (e.g., 2025-10) haven't been observed long enough to populate later month_index values — later-month comparisons are only shown for cohorts with enough elapsed time
- The month-5 retention spike is not yet explained and should be investigated before being used to inform strategy
- Findings are based on the observed data window only; no causal claims are made about *why* Basic-tier or West-region users churn more, only that they do

## Links

- [Capstone proposal (full BQ/AQ, metrics, cleaning plan, checkpoints, appendix)](https://docs.google.com/document/d/17uHUMtKpER54ztvYs_KwKdtZ8DrKjB9hELHHIlCNtuc/edit)
- [Final presentation](https://docs.google.com/presentation/d/1oteib-A27BTfFOZFels2lqaXeHBVFBX5yFHZHsk-jrA/edit)
