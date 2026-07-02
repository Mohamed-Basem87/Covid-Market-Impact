# 📊 COVID Market Impact Dashboard

**A Power BI analytics project examining how COVID-19 reshaped stock performance, risk, and profitability across 20 Fortune-500-class companies (2018–2022), comparing Technology vs. Non-Technology sectors.**

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Language-DAX-7B7FD6)
![Status](https://img.shields.io/badge/Status-Complete-3FBF8F)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Project Overview

This project analyzes the financial market impact of COVID-19 on 20 major companies, split into **Technology** and **Non-Technology** cohorts, using five years of daily stock data (2018–2022) and annual company financials. It combines classic financial risk/return analytics (total return, volatility, drawdown, beta, Sharpe ratio) with a dedicated pandemic-shock analysis (drawdown depth, recovery speed) and closes with an investor-facing recommendation framework.

The data was modeled into a **Star Schema** (2 dimension tables + 3 fact tables, ~50,850 total rows) and visualized through an interactive Power BI dashboard containing **six analytical pages**, driven by roughly **90 DAX measures**.

📄 Full write-up: [`docs/COVID_Market_Impact_Dashboard_Documentation.docx`](docs/COVID_Market_Impact_Dashboard_Documentation.docx)
📝 Short summary: [`docs/COVID_Market_Impact_Dashboard_Brief_Description.md`](docs/COVID_Market_Impact_Dashboard_Brief_Description.md)

---

## Project Structure

```
covid-market-impact-dashboard/
│
├── PowerBI/
│   └── COVID_Market_Impact_Dashboard.pbix     # Power BI dashboard file (add before committing)
│
├── data/
│   ├── daily_stock_prices.csv                 # Daily OHLCV price history per company (2018–2022)
│   ├── company_financials.csv                 # Annual revenue, net income, assets, employees
│   └── market_index.csv                       # Daily market benchmark used for beta calculation
│
├── docs/
│   ├── COVID_Market_Impact_Dashboard_Documentation.docx   # Full project documentation
│   └── COVID_Market_Impact_Dashboard_Brief_Description.md # Short project summary
│
├── screenshots/
│   ├── 01-overview.png
│   ├── 02-ranking.png
│   ├── 03-covid-deep-dive.png
│   ├── 04-tech-vs-nontech.png
│   ├── 05-company-drilldown.png
│   ├── 06-recommendations.png
│   └── 07-data-model.png
│
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🖼️ Dashboard Preview

### Overview
![Overview](screenshots/01-overview.png)

### Ranking
![Ranking](screenshots/02-ranking.png)

### COVID Deep Dive
![COVID Deep Dive](screenshots/03-covid-deep-dive.png)

### Tech vs Non-Tech
![Tech vs Non-Tech](screenshots/04-tech-vs-nontech.png)

### Company Drill Down
![Company Drill Down](screenshots/05-company-drilldown.png)

### Recommendations
![Recommendations](screenshots/06-recommendations.png)

---

## 📊 Data Architecture

The project follows a simplified two-stage flow: raw daily price and annual financial data are cleaned and shaped in **Power Query**, then loaded into a **Star Schema** in the Power BI data model for high-performance analytical queries.

```
Raw Sources (daily prices, annual financials, market index)
        │
        ▼
  Power Query (cleaning, type casting, column derivation)
        │
        ▼
  Star Schema Data Model  →  DAX Measures  →  6-Page Power BI Report
```

![Data Model](screenshots/07-data-model.png)

---

## 🛠️ Data Catalog

The model uses a **Star Schema** with one date dimension, one company dimension, and three fact tables (as seen in the Power BI Model View).

### 1. `dimCompany` (Dimension Table)

**Purpose:** Stores static company classification attributes used to segment every other table.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `Company` | Text | Company name (Primary Key) |
| `Sector` | Text | Broad sector classification (e.g., Technology, Retailing, Transportation) |
| `Industry` | Text | Detailed industry classification (e.g., Airlines, Computer Software) |
| `Is Tech` | Text | Binary flag: `Tech` or `Non-Tech`, drives the sector-comparison pages |

### 2. `dcalendar` (Date Dimension Table)

**Purpose:** Custom calendar table generated in DAX, marked as the official Date Table to enable native time-intelligence functions.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `Date` | Date | Continuous daily calendar generated with `CALENDARAUTO()` |
| `Year` | Whole Number | Calendar year |
| `Quarter` | Text | Quarter label (e.g., "Q3") |
| `Month` | Text | Three-letter month abbreviation |
| `Month Number` | Whole Number | Numeric month (1–12), used for correct chronological sorting |

```dax
dcalendar =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month", FORMAT([Date], "mmm"),
    "Month Number", MONTH([Date])
)
```

### 3. `20 Companies` (Fact Table — Annual)

**Purpose:** Stores annual financial statement data used for profitability and efficiency metrics.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `Year` | Date | Fiscal year of the record |
| `Company` | Text | Foreign key to `dimCompany` |
| `Number of Employees` | Whole Number | Headcount for the fiscal year |
| `Annual Rev` | Decimal Number | Total annual revenue |
| `Net Income` | Decimal Number | Net income for the fiscal year |
| `Total Assets` | Decimal Number | Total assets on the balance sheet |
| `Profit` | Decimal Number | Net profit margin (Net Income / Revenue) |
| `Profit of Assets` | Decimal Number | Return on assets, ROA (Net Income / Total Assets) |

### 4. `Daily Stock Return` (Fact Table — Daily)

**Purpose:** Core price-history fact table; the source for return, volatility, and drawdown calculations.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `Date` | Date | Trading date, foreign key to `dcalendar` |
| `company` | Text | Foreign key to `dimCompany` |
| `Open`, `High`, `Low`, `Close` | Decimal Number | Daily OHLC price |
| `Adj Close` | Decimal Number | Dividend/split-adjusted closing price, used for return calculations |
| `Volume` | Whole Number | Daily trading volume |
| `Daily Return` | Decimal Number | Day-over-day % change in `Adj Close` |
| `Running Max Close` | Decimal Number | Rolling maximum of `Adj Close` to date, used to compute drawdown |
| `Drawdown` | Decimal Number | % decline from `Running Max Close` to current `Adj Close` |
| `Day Index` | Whole Number | Sequential trading-day counter per company, used to align companies on a common time axis |

### 5. `BetaReturns` (Fact Table — Daily)

**Purpose:** Pairs each company's daily return against a market benchmark return specifically to support beta calculation.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `company` | Text | Foreign key to `dimCompany` |
| `Date` | Date | Trading date, foreign key to `dcalendar` |
| `Stock Return` | Decimal Number | Company's daily return |
| `Market Adj Close` | Decimal Number | Benchmark index adjusted close |
| `Market Return` | Decimal Number | Benchmark index daily return |

---

### Star Schema Relationships

| From | To | Cardinality | Key |
|---|---|---|---|
| `dimCompany` | `20 Companies` | 1 → * | `Company` |
| `dimCompany` | `Daily Stock Return` | 1 → * | `Company` |
| `dimCompany` | `BetaReturns` | 1 → * | `Company` |
| `dcalendar` | `Daily Stock Return` | 1 → * | `Date` |
| `dcalendar` | `BetaReturns` | 1 → * | `Date` |

This structure lets every visual filter simultaneously by company attributes (Sector, Is Tech) and date attributes (Year, Quarter) without duplicating logic anywhere in the model.

---

## 🔬 Data Dictionary: Key Field Analysis

This section details the analytical role of the most important fields and the general DAX pattern used to turn them into report-ready measures.

| Field Name | Source Table | Analytical Role & DAX Pattern |
|---|---|---|
| **Adj Close** | `Daily Stock Return` | Basis for **Total Return**: `DIVIDE([Ending Adj Close] - [Starting Adj Close], [Starting Adj Close])` evaluated at first/last `Day Index` per company. |
| **Daily Return** | `Daily Stock Return` | Basis for **Volatility**: `STDEV.P` over the `Daily Return` column within company/date filter context, and for **Sharpe / Risk-Adjusted Return**: `DIVIDE([Avg Daily Return], [Volatility])`. |
| **Running Max Close / Drawdown** | `Daily Stock Return` | Basis for **Max Drawdown**: `MIN([Drawdown])` per company, computed as `DIVIDE([Adj Close] - [Running Max Close], [Running Max Close])`. |
| **Stock Return / Market Return** | `BetaReturns` | Basis for **Beta**: `DIVIDE(COVARIANCE.P('BetaReturns'[Stock Return], 'BetaReturns'[Market Return]), VAR.P('BetaReturns'[Market Return]))`. |
| **Net Income / Total Assets** | `20 Companies` | Basis for **ROA**: `DIVIDE(SUM([Net Income]), SUM([Total Assets]))`. |
| **Net Income / Annual Rev** | `20 Companies` | Basis for **Profit Margin**: `DIVIDE(SUM([Net Income]), SUM([Annual Rev]))`. |
| **Is Tech** | `dimCompany` | Drives every Tech-vs-Non-Tech comparison measure via `CALCULATE([Measure], dimCompany[Is Tech] = "Tech")` and the `"Non-Tech"` equivalent. |
| **Day Index** | `Daily Stock Return` | Aligns companies on a common trading-day axis so COVID-drawdown and recovery measures compare like-for-like periods regardless of calendar-date gaps (holidays, listing dates). |

---

## 📊 Dashboard Pages: Technical Deep Dive

The Power BI report relies on the star schema above and layers roughly 90 DAX measures on top of it to deliver each page's insights.

### 📋 Overview Page

| Focus Area | Technical Implementation |
|---|---|
| **KPI Calculations (DAX)** | `Best Total Return Company` / `Worst Total Return Company` use `TOPN(1, ...)` over a virtual table of per-company `[Total Return]`, paired with footer measures that concatenate the value for card display. |
| **Tech vs Non-Tech Trend** | `Tech Avg Close` / `Non-Tech Avg Close` use `CALCULATE(AVERAGE('Daily Stock Return'[Adj Close]), dimCompany[Is Tech] = "Tech")`, evaluated across `dcalendar[Year]`/`dcalendar[Quarter]` for the area chart. |
| **Dynamic Bar Coloring** | `Total Return Bar Color` returns a hex string conditionally (`IF([Total Return] < 0, "#E06666", "#3FBF8F")`) and is bound to the bar chart's Data Colors via a field-value color measure. |

### 📈 Ranking Page

| Focus Area | Technical Implementation |
|---|---|
| **Risk-Adjusted Ranking** | `Risk-Adjusted Return` / `Sharpe Ratio` divide average daily return by volatility per company; `Highest/Lowest Sharpe Ratio Company` wrap this in `TOPN`/`BOTTOMN` over a company-level summary table. |
| **Scatter Plot (Return vs Risk)** | Plots `[Volatility]` (X) against `[Total Return]` (Y) at `Company` grain, colored by `dimCompany[Is Tech]` — no additional measure required beyond the two base metrics. |
| **Beta Ranking** | `Average Beta`, `Beta Rank`, and `Beta Podium Color` compute company beta via `BetaReturns`, rank it with `RANKX`, and conditionally color the top 3 podium bars (gold/silver/bronze-style palette). |

### 🦠 COVID Deep Dive Page

| Focus Area | Technical Implementation |
|---|---|
| **COVID Drawdown** | `COVID Drawdown` restricts `[Max Drawdown]` to the 2020 window using `CALCULATE([Max Drawdown], dcalendar[Year] = 2020)`, isolating the pandemic-specific decline from the full 2018–2022 drawdown. |
| **COVID Recovery** | `COVID Recovery` measures the % gain from each company's 2020 trough (`Day Index` at `MIN([Drawdown])`) to a subsequent peak, using `CALCULATE` with a `FILTER` over `Daily Stock Return` restricted to post-trough dates. |
| **Recovery Index** | `Normalized Index` rebases each cohort's average close to a common starting value (Q1 2020 = 100) so Tech and Non-Tech recovery trajectories are visually comparable regardless of absolute price level. |
| **COVID Outperformance / Spread** | `COVID Outperformance` = `[Tech Avg COVID Return] - [Non-Tech Avg COVID Return]`; `COVID Recovery Spread` = `[Best COVID Recovery Value] - [Weakest COVID Recovery Value]`. |

### ⚖️ Tech VS Non-Tech Page

| Focus Area | Technical Implementation |
|---|---|
| **Return Gap** | `Return Gap` / `Return Gap Value` = `[Tech Avg Return] - [Non-Tech Avg Return]`, displayed as both a KPI card and a labeled bar. |
| **Revenue Totals by Year** | Grouped bar aggregates `SUM('20 Companies'[Annual Rev])` split by `dimCompany[Is Tech]` and `dcalendar[Year]`, requiring the annual fact table to relate cleanly to both dimensions. |
| **Return Stability** | `Return Stability` computes the inverse of return dispersion (e.g., `1 / STDEV.P` of period returns) per cohort, so a higher bar means more consistent (less volatile) returns. |

### 🔎 Company Drill Down Page

| Focus Area | Technical Implementation |
|---|---|
| **Dynamic Titles** | `Price Trend Title` concatenates the slicer-selected company into a string (`SELECTEDVALUE(dimCompany[Company]) & " — Price Trend (2018–2022)"`) so the chart title updates live with the selector. |
| **Trend Signal (MA Crossover)** | `MA30` / `MA90` compute rolling averages over `Daily Stock Return[Adj Close]` using `AVERAGEX` with a `DATESINPERIOD` window; `Trend Signal` compares the two with an `IF(MA30 > MA90, "Bullish (Golden Cross)", "Bearish (Death Cross)")` pattern. |
| **Revenue Efficiency Trend** | `Revenue per Employee` = `DIVIDE(SUM('20 Companies'[Annual Rev]), SUM('20 Companies'[Number of Employees]))`, filtered to the selected company via `SELECTEDVALUE`. |

### 🏆 Recommendations Page

| Focus Area | Technical Implementation |
|---|---|
| **Investment Simulation** | The "Final Investment Value" table projects a hypothetical fixed investment forward using each company's `[Total Return]` measure: `Final Value = Initial Investment × (1 + [Total Return])`. |
| **Narrative Cards** | Each Q&A card binds a `TOPN(1, ...)` "winner" measure (e.g., `Best Investment`) to a text card, with a paired footer measure supplying the supporting stat sentence. |
| **Investor Profile Table** | A static mapping table (Growth / Balanced / Conservative → Company) cross-referenced against the underlying ranking measures rather than hard-coded, so it stays consistent if the underlying data changes. |

---

## 🧮 Analytical Measures Summary

This project's ~90 DAX measures fall into six functional groups:

- **Return & Ranking:** `Total Return`, `Rank`, `Best/Worst Total Return Company`, `Total Return Bar Color`
- **Risk & Volatility:** `Volatility`, `Max Drawdown`, `Highest/Lowest Volatility Company`, `Volume Spike Count`
- **Risk-Adjusted & Market:** `Risk-Adjusted Return`, `Sharpe Ratio`, `Average Beta`, `Beta Rank`
- **COVID-Specific:** `COVID Drawdown`, `COVID Recovery`, `COVID Outperformance`, `COVID Recovery Spread`
- **Profitability & Fundamentals:** `ROA`, `Profit Margin`, `Revenue per Employee`, `Total Revenue`
- **Tech vs Non-Tech Comparison:** `Tech/Non-Tech Avg Return`, `Return Gap`, `Return Stability`, `Tech Color`
- **Trend & Technical Indicators:** `MA30`, `MA90`, `Trend Signal`, `Normalized Index`

---

## 🏆 Key Insights

- **Tesla** delivered the highest total return (**+476.5%**) and the strongest COVID-era recovery (**+877%**) off its 2020 low.
- **American Airlines** posted the worst total return (**-76.0%**) and the deepest COVID drawdown (**-84.5%**), reflecting the severity of pandemic-era travel restrictions.
- **Apple** had the best risk-adjusted return, making it the strongest "Balanced" investor pick.
- **Johnson & Johnson** was the most stable holding in the dataset (lowest daily volatility, σ = 1.08%), consistent with its defensive healthcare classification.
- **Technology companies** outperformed Non-Technology by **+67 points** on average return (105% vs 38%) and recovered from the pandemic roughly **2.4× faster**, while carrying only modestly higher volatility.
- **Non-Tech companies** held a larger revenue base going into the pandemic, but Tech's faster growth overtook it by 2022.

---

## ⚙️ Tools & Technologies Used

| Category | Tool | Purpose |
|---|---|---|
| **Data Modeling** | Power BI Desktop | Star schema construction, relationship management |
| **Calculation Engine** | DAX | ~90 measures covering return, risk, profitability, and sector-comparison logic |
| **Data Preparation** | Power Query (M) | Cleaning, type casting, and derived-column creation (Running Max Close, Drawdown, Day Index) |
| **Version Control** | Git & GitHub | Project hosting, versioning |
| **Documentation** | Microsoft Word, Markdown | Full documentation and short-form project summary |

---

## 🚀 Getting Started

1. Clone this repo.
2. Open the `.pbix` file (once added to `PowerBI/`) in **Power BI Desktop**.
3. Refresh the data source connections if prompted.
4. Explore the report pages via the left-hand navigation rail.

> **Note:** The `.pbix` file and raw source data are not included in this repository scaffold. Add your own `.pbix` to `PowerBI/` and source CSVs to `data/` before pushing — see `.gitignore` for what's excluded by default.

---

## 📄 License

This project is licensed under the MIT License — see [`LICENSE`](LICENSE) for details.

---

## 👤 Author

**Mohamed Bassem**
