# 🛒 Superstore End-to-End Sales Analytics

> A complete data analytics project — raw data → Python EDA → SQL window functions → A/B testing → K-Means segmentation → Power BI + Looker Studio dashboards.

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Looker Studio](https://img.shields.io/badge/Looker%20Studio-4285F4?style=flat&logo=google&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=flat&logo=scipy&logoColor=white)

---

## 📊 Live Interactive Dashboard

**[→ Open Live Looker Studio Dashboard](https://lookerstudio.google.com/reporting/73ef6ad7-9c26-4732-bbe0-34e03d08ca5c)**

> No login required · Filter by Year, Region, Category, and Customer Segment

---

## 🎯 Project Overview

Analysed **9,994 orders** from the Superstore retail dataset (2014–2017) across 793 unique customers, 3 product categories, and 49 US states. The entire analysis pipeline — data cleaning, EDA, SQL analytics, statistical testing, and machine learning — runs in a single Jupyter notebook.

### Key Findings

| Finding | Detail |
|---------|--------|
| 📉 Discounting destroys margin | Discounted orders have a significantly lower profit margin (p < 0.001, Cohen's d = 1.03 — large effect) |
| 👑 Revenue concentration | Champions segment = 25% of customers, drives the largest share of revenue |
| 🗺️ Tables = structural loss | Tables sub-category loses money in every region, every single year |
| 📅 Seasonal demand | Q4 accounts for ~35% of annual sales — consistent pattern across all 4 years |
| 🔄 Rolling trend | 3-month rolling average confirms underlying growth masked by monthly volatility |

---

## 📁 Files in This Repository

```
superstore-sales-analytics/
│
├── Superstore_sale_analytics.ipynb    ← main notebook — all analysis (112 cells)
├── Sample_-_Superstore.csv            ← original raw dataset (9,994 rows × 21 cols)
├── customers_segmented_final.csv      ← 793 customers with RFM + K-Means segments
├── superstore_with_segments.csv       ← full orders enriched with K-Means segments
├── Superstore_sales_analytics.pbix    ← Power BI Desktop dashboard file
└── README.md
```

---

## 📓 Notebook — Full Walkthrough

**[Superstore_sale_analytics.ipynb](Superstore_sale_analytics.ipynb)** · 112 cells · 7 phases

---

### Phase 1 — Data Loading & Cleaning `Cells 1–15`

- Loaded `Sample_-_Superstore.csv` with `latin-1` encoding — 9,994 rows, 21 columns, zero nulls
- Parsed `Order Date` and `Ship Date` as datetime objects
- Derived columns: `Ship Days` (delivery time), `Profit Margin` (Profit / Sales), `Year`, `Month`
- Checked for duplicate orders — none found
- Quick breakdowns: top 5 categories, sub-categories, and regions by sales

---

### Phase 2 — Exploratory Data Analysis `Cells 16–49`

8 charts built with Seaborn and Matplotlib.

| Chart | Key Insight |
|-------|-------------|
| Sales distribution (histogram + log) | Heavily right-skewed — most orders are small, a few are very large |
| Profit distribution | Significant left tail — many loss-making transactions |
| Discount vs Profit scatter + regression | Clear inverse relationship — higher discount → lower or negative profit |
| Monthly sales trend 2014–2017 | Consistent Q4 peaks every year confirm seasonal demand |
| Correlation heatmap | Discount negatively correlated with Profit; Sales positively correlated |
| Box plots — Profit by Region & Category | Furniture has the most loss outliers; Technology has the highest upside |
| Top 10 by Revenue vs Top 10 by Profit | The two lists barely overlap — revenue leaders ≠ profit leaders |

---

### Phase 3 — SQL Window Functions `sql/window_functions.sql`

10 queries on MySQL 8.0 demonstrating advanced window function patterns.

| # | Function Used | Business Question Answered |
|---|--------------|---------------------------|
| Q1 | `RANK()` OVER (PARTITION BY category) | Which products are the revenue leaders within each category? |
| Q2 | `DENSE_RANK()` OVER (PARTITION BY region) | Who are the highest-value customers in each region? |
| Q3 | `SUM()` OVER running total | Cumulative revenue growth — when did the business cross $1M? |
| Q4 | `LAG()` | Month-over-month revenue growth % — best and worst months |
| Q5 | `LEAD()` | Days until each customer's next order — identify long gaps |
| Q6 | `NTILE(4)` | Bucket all customers into profit quartiles — who is costing money? |
| Q7 | `AVG()` OVER (ROWS BETWEEN 2 PRECEDING) | 3-month rolling average — smoothed underlying trend |
| Q8 | `FIRST_VALUE / LAST_VALUE` | Customer first + last purchase dates and total lifespan |
| Q9 | `AVG()` OVER (PARTITION BY category) | Orders where discount exceeds the category average — loss analysis |
| Q10 | `RANK()` + CTE | Top 3 and bottom 3 sub-categories per region by profit |

---

### Phase 4 — A/B Test: Promotion Effectiveness `Cells 55–71`

Tested whether applying a discount to an order significantly reduces its profit margin.

```
H₀: Profit margin is equal for discounted vs non-discounted orders
H₁: Profit margin is lower for discounted orders
α = 0.05 · One-tailed test
```

**Sample sizes:**
- Control (no discount): **4,798 orders**
- Treatment (discounted): **5,196 orders**

**Results:**

| Step | Result |
|------|--------|
| Shapiro-Wilk normality test | p < 0.001 for both groups — data is non-normal |
| Test selected | Mann-Whitney U (non-parametric — correct choice for non-normal data) |
| p-value | **< 0.001** |
| Cohen's d | **1.03 — large effect size** |
| 95% Bootstrap CI | **[−43.7%, −40.6%]** |

**Conclusion:** We reject H₀. Discounted orders are associated with a statistically significant and large reduction in profit margin. The bootstrap CI confirms the true effect is between −40.6 and −43.7 percentage points — this is not chance variation.

---

### Phase 5 — K-Means Customer Segmentation `Cells 72–112`

Built RFM (Recency, Frequency, Monetary) features for 793 customers and applied K-Means clustering.

**RFM Dataset Summary:**

| Metric | Mean | Min | Max |
|--------|------|-----|-----|
| Recency (days since last order) | 147.8 | 1 | 1,457 |
| Frequency (unique orders) | 6.3 | 1 | 37 |
| Monetary (total spend $) | $2,896.85 | $4.83 | $25,043 |

**K Selection process:**
- Log-transformed all 3 features (fix right-skew) → StandardScaled (equalise feature ranges)
- Elbow method: inertia drops steeply K=2→4 (1481 → 987), flattens after K=4
- Silhouette scores: peaks at K=2 (0.35), flat K=3–K=8 (0.24–0.26) — domain knowledge breaks tie
- **Decision: K=4** — maps to 4 actionable business segments

**Cluster Profiles:**

| Segment | Count | Recency | Avg Orders | Avg Spend | Characteristic |
|---------|-------|---------|------------|-----------|----------------|
| 🔵 Champions | 199 (25.1%) | 19.7 days | 6.8 | $2,712.90 | Recent, frequent, solid spend |
| 🟢 Loyal | 258 (32.5%) | 103.3 days | 8.6 | $4,773.00 | Highest orders + spend — slightly dormant |
| 🟠 At-Risk | 253 (31.9%) | 235.2 days | 4.9 | $1,936.80 | Gap widening — need win-back now |
| 🔴 Hibernating | 83 (10.5%) | Longest gap | Lowest | Lowest | Very long since last order |

**Visualisations produced:**
- 2D scatter — Recency vs Monetary coloured by segment, with centroids
- Interactive 3D Plotly scatter — R, F, M axes (saved as HTML)
- 4-panel summary: segment sizes, avg spend, avg recency, revenue share donut
- Snake plot — normalised R, F, M profile per segment across all dimensions

**Business recommendations:**

| Segment | Action |
|---------|--------|
| Champions | Loyalty rewards, early access, premium upsell — do NOT discount |
| Loyal | Personalised bundles, milestone offers — nudge toward Champions |
| At-Risk | Immediate win-back campaign with time-limited offer |
| Hibernating | Single low-cost email — archive if no response within 30 days |

---

## Phase 6 — Power BI Dashboard

**File:** [Superstore_sales_analytics.pbix](Superstore_sales_analytics.pbix) *(open in Power BI Desktop — free)*

4-page executive dashboard with star schema data model and custom DAX measures.

**Data model:** `DateTable (1) → superstore_with_segments (*) ← customers_segmented_final (1)`

**DAX Measures created:** Total Sales, Total Profit, Profit Margin %, Order Count, YoY Sales Growth, Champions Revenue %, States With Net Loss, Segment Color (conditional formatting)

| Page | Key Visuals |
|------|-------------|
| 1 — Executive Overview | KPI cards, monthly trend line, annual sales by category, year/region/category slicers |
| 2 — Product Analysis | Treemap by sub-category, top 10 products by profit (colour-coded), discount scatter, sub-category profit matrix |
| 3 — Geography | US filled map by state, region bar chart, state performance table, 3 KPI callout cards |
| 4 — Customer Segmentation | Segment donut, revenue by segment bar, RFM scatter, top 20 customer table with conditional formatting |

---

## Phase 7 — Looker Studio Dashboard

**[→ Open Live Dashboard]((https://lookerstudio.google.com/reporting/73ef6ad7-9c26-4732-bbe0-34e03d08ca5c))** · No login required

**Data sources:** `superstore_with_segments.csv` + `customers_segmented_final.csv` (via Google Sheets)

**Calculated fields:** Profit Margin %, Ship Days, Discounted Order (Yes/No), Order Year, Year-Month

| Visual | Description |
|--------|-------------|
| 4 KPI Scorecards | Total Revenue $2.3M · Total Profit $286K · Avg Margin 12% · Orders 5K |
| Time Series | Monthly Sales & Profit — dual axis (bars + line) |
| Bar Chart 1 | Revenue by Sub-Category — horizontal, sorted descending |
| Bar Chart 2 | Profit by Region — colour-coded positive/negative |
| Geo Map | Sales by US State — Google Maps choropleth |
| Donut Chart | Revenue by K-Means Customer Segment |
| Customer Table | Top 20 customers — spend, orders, recency heatmap |
| Filter Controls | Date range, Region, Category, Customer Segment |

---

## ⚙️ Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.10+ |
| Data wrangling | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn, Plotly |
| Statistics | SciPy (Mann-Whitney U, Shapiro-Wilk, Bootstrap CI) |
| Machine Learning | Scikit-learn (KMeans, StandardScaler, silhouette_score) |
| SQL | MySQL 8.0, SQL Workbench, mysql-connector-python |
| BI Dashboards | Power BI Desktop, Looker Studio |
| Dataset | [Superstore — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) |

---

## 🚀 How to Run

```bash
# 1. Clone the repo
git clone https://github.com/sandeepsingh9871/superstore-sales-analytics.git
cd superstore-sales-analytics

# 2. Install Python dependencies
pip install pandas numpy matplotlib seaborn scipy scikit-learn plotly mysql-connector-python

# 3. Open and run the notebook
jupyter notebook Superstore_sale_analytics.ipynb
# Run all cells top to bottom (Kernel → Restart & Run All)

# Note: SQL cells (Phase 3) require MySQL 8.0 running locally
# Create a database called 'superstore_analytics' first:
# CREATE DATABASE superstore_analytics;
```

**To view dashboards without running any code:**
- Power BI: download `Superstore_sales_analytics.pbix` → open in Power BI Desktop (free)
- Looker Studio: click the live link above — browser only, no software needed

---

## 📦 Python Requirements

```
pandas
numpy
matplotlib
seaborn
scipy
scikit-learn
plotly
mysql-connector-python
```

---

## 📄 Dataset

**[Superstore Dataset on Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)**

- 9,994 rows × 21 columns
- US retail orders: January 2014 – December 2017
- 3 categories · 17 sub-categories · 49 states · 793 unique customers · 1,862 products

---

## 👤 Author

**Sandeep Singh**
---

*End-to-end analytics portfolio project.*
