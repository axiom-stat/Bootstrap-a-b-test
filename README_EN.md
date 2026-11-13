<div align="right">
  <a href="README.md">🇷🇺 Русский</a> | <b>🇬🇧 English</b>
</div>

# Dynamic Marketplace Fee A/B Test: Short-Term Revenue vs Hidden Churn

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-Bootstrap%20BCa-8CAAE6?logo=scipy&logoColor=white)
![Pingouin](https://img.shields.io/badge/Pingouin-Statistics-orange)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)

End-to-end product analytics audit of a dynamic ML-driven service fee A/B experiment on a quick-delivery e-commerce marketplace (~14,000 orders). This case study exposes a classic local optimization trap: top-line buyer fee revenue increased by +12%, yet **Bootstrap (BCa)**, **Chi-square testing**, and **LTV unit-economics modeling** reveal that the algorithm erodes long-term Gross Merchandise Value (GMV) and drives net negative profitability

---

### Business Problem & Key Findings

The monetization team tested replacing a flat service fee (`5.5% + $1.50`) with an ML-powered dynamic pricing algorithm. Initial reporting showed a strong lift in average buyer fee per order, prompting a push for a 100% rollout

A deep-dive investigation uncovered a hidden retention crisis:

| Metric (`Standard` Tier) | Control (Flat Fee) | Experimental (Dynamic ML) | Impact & Significance |
| :--- | :---: | :---: | :--- |
| Median Buyer Fee (all orders, `service_fee`) | ~\$3.82 | ~\$4.90 | **+\$1.08** (95% BCa CI `[1.03, 1.15]`) |
| 7-Day Repeat Order Rate (`Retention`) | 34.0% | 25.1% | **-8.9 pp** (-26.1% relative drop, $p < 0.001$) |
| Average Order Rating (`CSAT`, all orders) | 3.90 | 3.65 | **-0.25 stars** (1-2 star share jumped 16.6% to 29.0% in `Standard`) |
| Expected Orders per User (`Standard` < \$500) | 1.51 | 1.34 | **-11.6%** (order frequency collapse) |
| Expected GMV per User (`Standard` < \$500) | \$63.08 | \$54.76 | **-\$8.33** (-13.2% turnover drop) |

**Why the feature loses money over time:**
On small baskets (\$15-\$20) in the core `Standard` tier, the ML algorithm pushed the effective fee up to 25-40% of cart value, triggering severe price shock and customer churn. Per 1,000 `Standard` users (delivered orders under \$500), the business gains +\$964 in buyer service fees but loses -\$8,327 in GMV. Assuming a conservative 15% merchant commission on GMV, lost merchant margin equals -\$1,249 - resulting in a **net loss of -\$285 per 1,000 users**, before accounting for Customer Acquisition Cost (CAC) required to replace churned buyers

---

### Analytical Pipeline

1. **Data Hygiene & Audit:** cleaned raw transaction logs (~1.58% corrupted records: 28 duplicate rows, negative fees, zero-duration GPS glitches) and verified strict user-level randomization across `user_id`
2. **EDA & Take-Rate Dynamics:** analyzed heavy-tailed order distributions, whale orders (>\$800), and hyperbolic fee spikes on low-value carts
3. **Bootstrap (BCa) vs t-test:** addressed severe right-skewness by running vectorized `scipy.stats.bootstrap` (10,000 resamples, BCa intervals) for both mean and median fee shifts
4. **Segmentation & Churn Validation:** isolated heterogeneous treatment effects across loyalty tiers (`Standard`, `Plus`, `Premium`) and confirmed statistically significant retention drop via `pingouin.chi2_independence` (Cramer's V = 0.097, Power = 1.0)
5. **Unit-Economics Modeling:** projected customer lifetime orders via $1 / (1 - p)$ to quantify GMV loss and formulate an actionable rollback decision

---

### Executive Decision & Recommendations

- **Decision:** **Rollback** the current model and halt the 100% rollout
- **Cap Small-Basket Take-Rate:** enforce a hard ceiling of 12-15% on orders under \$40 to eliminate checkout price shock
- **Align ML Objective Function:** optimize the pricing model for long-term LTV (balancing repeat purchase probability and GMV retention) rather than single-order `service_fee`

---

### Repository Structure & Setup

- [`marketplace_ab_test_project.ipynb`](marketplace_ab_test_project.ipynb) - complete Jupyter Notebook with clean code, visualizations, and insights
- [`marketplace_orders_experiment.csv`](marketplace_orders_experiment.csv) - experiment dataset (~14,000 transactions)
- [`requirements.txt`](requirements.txt) - Python dependencies

Run locally:
```bash
git clone https://github.com/axiom-stat/Bootstrap-a-b-test.git
cd Bootstrap-a-b-test
pip install -r requirements.txt
jupyter notebook marketplace_ab_test_project.ipynb
```