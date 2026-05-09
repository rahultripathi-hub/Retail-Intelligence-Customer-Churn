# Retail Intelligence: Customer Churn

Customer churn is one of the most critical challenges in retail and e-commerce. Even a small improvement in retention can significantly impact revenue and long-term growth.

This notebook provides a full exploratory data analysis on **100,000 retail customers**, structured around five business questions:

- Which customers are most likely to churn?
- What behavioral patterns lead to churn?
- Does engagement reduce churn risk?
- Which segments are high-value but at risk?
- How can businesses improve retention strategies?

---

## Project Files

```
retail-churn-analysis/
├── retail_customer_churn.csv          # Raw dataset (100,000 rows)
├── retail_customer_churn.ipynb        # Main analysis notebook
└── README.md                          # This file
```

> **Important:** Place `retail_customer_churn.csv` in the **same folder** as the notebook before running. The notebook loads it with `pd.read_csv('retail_customer_churn.csv')` using a relative path.

---

## Requirements

```bash
pip install pandas numpy matplotlib
```

Python 3.8+ recommended. No additional libraries required.

---

## How to Run

```bash
jupyter notebook retail_customer_churn.ipynb
# or
jupyter lab retail_customer_churn.ipynb
```

Then: **Kernel → Restart & Run All**

---

## Dataset Columns

| Column | Type | Description |
|---|---|---|
| `customer_id` | string | Unique customer identifier |
| `age_group` | categorical | 18-24, 25-34, 35-44, 45-54, 55+ |
| `gender` | categorical | Male, Female, Other |
| `region` | categorical | North, South, East, West, Central |
| `customer_segment` | categorical | New, Returning, Loyal, VIP |
| `preferred_channel` | categorical | Online, Mobile App, In-Store |
| `purchase_frequency` | integer | Number of purchases made |
| `avg_order_value` | float | Average value per order ($) |
| `total_spent` | float | Cumulative lifetime spend ($) |
| `recency_days` | integer | Days since last purchase |
| `website_visits` | integer | Total website visits |
| `discount_usage_rate` | float | Proportion of orders using a discount (0–1) |
| `email_open_rate` | float | Proportion of emails opened (0–1) |
| `cart_abandonment_rate` | float | Proportion of carts abandoned (0–1) |
| `loyalty_score` | integer | Internal loyalty metric (0–100) |
| `engagement_score` | float | Composite engagement metric (0–100) |
| `churn_risk` | categorical | Low, Medium, High |
| `churn_flag` | integer | Target variable — 1 = churned, 0 = retained |

---

## Notebook Structure

### Section 1 — Setup & Data Loading

Imports `pandas`, `numpy`, `matplotlib.pyplot`, `matplotlib.ticker`, and `matplotlib.patches`. Configures global plot styling via `plt.rcParams` (white background, light grey grid, no top/right spines). Defines a shared `COLORS` dictionary (red, amber, green, dark_red, blue, teal, purple, gray). Loads the CSV with `pd.read_csv`.

---

### Section 2 — Dataset Overview

Runs `df.info()` for a full summary of shape, dtypes, and non-null counts. Then separately prints:

- `df.dtypes` and `df.isnull().sum()` with null percentages
- Unique values for all categorical/boolean columns using `df.select_dtypes(include=['object', 'category', 'bool']).columns`
- Numerical summary via `df.describe().round(2)`

---

### Section 3 — Overall Churn Rate & Risk Distribution

Computes overall churn rate, churned/retained counts, risk level distribution, and churn rate per risk tier using `.groupby('churn_risk')['churn_flag'].agg(['mean', 'count'])`.

**Visualizations (3 separate charts):**

| Chart | Type | What it shows |
|---|---|---|
| Overall churn split | Pie chart | 53.8% retained vs 46.2% churned with `autopct` labels |
| Customers by risk level | Bar chart | Count of Low / Medium / High risk customers |
| Churn rate by risk level | Bar chart | 0% (Low), ~50% (Medium), 100% (High) |

---

### Section 4 — Five Business Questions

#### Question 1 — Which Customers Are Most Likely to Churn?

Loops through five categorical dimensions — `customer_segment`, `age_group`, `region`, `preferred_channel`, `gender` — and prints churn rate and count for each group.

**Visualizations (4 separate horizontal bar charts):**

- Churn rate by Customer Segment
- Churn rate by Age Group
- Churn rate by Region
- Churn rate by Channel

Each chart includes a red dashed vertical line at the overall average churn rate for instant above/below-average comparison.

> **Key insight noted in the notebook:** Churn is remarkably uniform across all dimensions — the problem is behavioural, not demographic.

---

#### Question 2 — What Behavioral Patterns Lead to Churn?

Creates three types of bins to analyse churn against purchasing behaviour:

- **Recency buckets** via `pd.cut` with fixed thresholds: `[0, 30, 90, 180, 365]` days
- **Cart abandonment quartiles** via `pd.qcut` into Q1–Q4
- **Loyalty score quartiles** via `pd.qcut` into Q1–Q4
- **Email open rate quartiles** via `pd.qcut` into Q1–Q4

**Visualizations (3 separate bar charts):**

| Chart | Colour logic | Key finding |
|---|---|---|
| Recency | Green → green → amber → red (by risk) | 82.3% churn after 180–365 days inactive |
| Cart abandonment | Green → amber → red → dark red (Q1→Q4) | 61% churn in top quartile vs 31% in bottom |
| Loyalty score | Dark red → amber → teal → green (Q1→Q4) | 57% churn in lowest quartile vs 35% in highest |

---

#### Question 3 — Does Engagement Reduce Churn Risk?

Builds a full numerical comparison table across 10 metrics for churned vs retained customers, adding `Difference` and `% Change` columns.

Then creates engagement and visits quartiles via `pd.qcut` and prints churn rates per tier.

**Visualizations (2 separate charts):**

| Chart | Type | What it shows |
|---|---|---|
| Retained vs churned mean metrics | Grouped bar | Side-by-side bars for loyalty score, engagement score, website visits |
| Engagement quartile churn rate | Bar chart | Churn rate drops from Q1 (low engagement) to Q4 (high engagement) |

Also explicitly prints the **non-signals** — metrics with negligible difference between churned and retained:

- `email_open_rate`
- `discount_usage_rate`
- `avg_order_value`

---

#### Question 4 — High-Value Customers at Risk

Cross-tabulates `customer_segment` × `churn_risk` using `.groupby().size().unstack()`, adds percentage columns for High/Medium/Low per segment.

Prints detailed profiles for **VIP** and **Loyal** high-risk customers: count, avg spend, avg recency, avg loyalty score, avg cart abandonment rate.

**Visualizations (3 separate charts):**

| Chart | Type | What it shows |
|---|---|---|
| Risk distribution by segment | Stacked bar | High/Medium/Low risk counts for VIP, Loyal, Returning, New |
| VIP total spend — high vs low risk | Overlapping histogram | Spend distribution of high-risk vs low-risk VIPs with mean lines |
| Avg spend by segment (high-risk only) | Horizontal bar | Which segments have highest spend at stake (teal bars) |

---

#### Question 5 — Retention Strategy Insights

**Visualizations (2 separate charts, each in its own `plt.subplots(1, figsize=(6.9, 5))`):**

| Chart | Type | What it shows |
|---|---|---|
| Recency distribution — retained vs churned | Overlapping histogram | Clear separation between groups; orange dashed line at 90 days, dark red at 180 days |
| Churn rate — loyalty × cart abandonment | Heatmap (`imshow`) | Interaction of two signals; `RdYlGn_r` colourmap; white/black text per cell |

The heatmap bins both `cart_abandonment_rate` and `loyalty_score` into 5 quintiles via `pd.qcut`, pivots with `.unstack()`, and annotates every cell with its churn % (white text when rate > 60% or < 20%, black otherwise).

---

### Summary & Actionable Recommendations

A formatted print block that dynamically computes all numbers from the dataframe inside the f-string, covering:

- Dataset totals
- Top signals (ranked strongest → weakest)
- Non-signals (negligible difference)
- High-value at risk (VIP and Loyal profiles)
- Six retention actions

---

## Key Findings

### Churn Overview
- **46.2% overall churn rate** — 46,215 of 100,000 customers
- **32,410 high-risk customers** — 100% churn rate
- **~27,000 medium-risk customers** — ~50% churn rate
- **~40,000 low-risk customers** — 0% churn rate

### Top Predictors of Churn

| Signal | Finding |
|---|---|
| **Recency** | 180–365 day inactive = **82.3% churn** vs 0% for < 30 days |
| **Cart abandonment** | Q4 (high) = **61% churn** vs Q1 (low) = **31.4% churn** |
| **Loyalty score** | Q1 (low) = **57.4% churn** vs Q4 (high) = **34.8% churn** |
| **Engagement score** | Churned avg **48.4** vs retained **53.6** |
| **Website visits** | Churned avg **54.8** vs retained **61.7** |

### Signals That Do NOT Predict Churn
- Email open rate — nearly identical, churned vs retained
- Discount usage rate — nearly identical
- Average order value — $209 (churned) vs $210 (retained)
- Channel — all ~46% churn across Online, Mobile App, In-Store
- Age group — all ~46% churn across every bracket
- Customer segment — all ~46% churn across VIP, Loyal, New, Returning

### High-Value Customers at Risk
- **8,144 VIP** customers at high risk — avg **$2,261** lifetime spend, avg **292 days** inactive
- **7,982 Loyal** customers at high risk — avg **$2,358** lifetime spend

### Retention Actions
1. **Win-back at 90 days** — churn risk jumps from 1% → 18% at this threshold
2. **Cart recovery flows** — target top abandonment quartile (61% at risk)
3. **Loyalty programme push** — low-loyalty customers churn at 57%
4. **VIP personal outreach** — highest revenue per customer saved
5. **Engagement monitoring** — a 2-week visit drop is an early warning sign
6. **Do not over-invest in email** — open rate does not correlate with retention

---

## Libraries Used

| Library | Role in this notebook |
|---|---|
| `pandas` | Data loading, groupby, `pd.cut`, `pd.qcut`, `.unstack()`, cross-tabs |
| `numpy` | `np.arange` for grouped bar x-positioning, `np.zeros` for stacked bar accumulation |
| `matplotlib.pyplot` | All charts — pie, bar, horizontal bar, histogram, heatmap |
| `matplotlib.ticker` | `FuncFormatter` for `$` prefix and `,` thousand-separator axis labels |
| `matplotlib.patches` | Imported for legend customisation |
| `warnings` | Suppresses pandas deprecation notices during `pd.cut`/`pd.qcut` operations |
