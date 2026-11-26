# 📊 COVID Market Impact Dashboard

**A Power BI analytics project examining how COVID-19 reshaped stock performance, risk, and profitability across 20 Fortune-500-class companies (2018–2022), comparing Technology vs. Non-Technology sectors.**

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Language-DAX-7B7FD6)
![Status](https://img.shields.io/badge/Status-Complete-3FBF8F)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Project Overview

This project analyzes the financial market impact of COVID-19 on 20 major companies, split into **Technology** and **Non-Technology** cohorts, using five years of daily stock data (2018–2022) and annual company financials. It combines classic financial risk/return analytics (total return, volatility, drawdown, beta, Sharpe ratio) with a dedicated pandemic-shock analysis (drawdown depth and recovery magnitude) and closes with an investor-facing recommendation framework.

The data is modeled in a **Star Schema** (2 dimension tables + 3 fact tables, ~50,850 total rows) and presented in **six analytical pages** with **81 DAX measures**. Return and risk measures use split/dividend-adjusted close-to-close prices; beta aligns stock and benchmark dates.

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
├── data/                                      # Scaffold only: source CSVs are not committed
│   ├── daily_stock_prices.csv                 # Planned: daily OHLCV price history (2018–2022)
│   ├── company_financials.csv                 # Planned: annual revenue, net income, assets, employees
│   └── market_index.csv                       # Planned: daily market benchmark for beta calculation
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

Raw daily price and annual financial data are cleaned in **Power Query**, then loaded into a **Star Schema** in the Power BI data model.

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
| `Adj Close` | Decimal Number | Split/dividend-adjusted close used by return, volatility, drawdown and recovery measures |
| `Volume` | Whole Number | Daily trading volume |
| `Daily Return` | Decimal Number | Adjusted close-to-close return: `(Adj Close today / Adj Close prior day) - 1` |
| `Running Max Adj Close` | Decimal Number | Rolling maximum of `Adj Close` to date, used to compute drawdown |
| `Drawdown` | Decimal Number | % decline from `Running Max Adj Close` to current `Adj Close` |
| `Day Index` | Whole Number | Sequential trading-day counter per company; COVID measures use calendar-date windows |

### 5. `BetaReturns` (Fact Table — Daily)

**Purpose:** Pairs each company's daily return against a market benchmark return specifically to support beta calculation.

| Column Name | Data Type (Power BI) | Description |
|---|---|---|
| `company` | Text | Foreign key to `dimCompany` |
| `Date` | Date | Trading date, foreign key to `dcalendar` |
| `Stock Return` | Decimal Number | Company's adjusted close-to-close return |
| `Market Adj Close` | Decimal Number | Benchmark index adjusted close |
| `Market Return` | Decimal Number | Benchmark return recalculated on dates aligned with the stock series |

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

Key measures use adjusted closes, sample standard deviation and date-aligned benchmark returns.

| Field Name | Source Table | Analytical Role & DAX Pattern |
|---|---|---|
| **Adj Close** | `Daily Stock Return` | Used by **Total Return**, **COVID Recovery**, **Normalized Index** and drawdown measures. |
| **Daily Return** | `Daily Stock Return` | Adjusted close-to-close return. **Volatility** is `STDEV.S([Adj Daily Return])`; **Sharpe Ratio** is `DIVIDE(AVERAGE([Adj Daily Return]), [Volatility])` with no risk-free-rate subtraction. |
| **Running Max Adj Close / Drawdown** | `Daily Stock Return` | Rolling maximum of `Adj Close` and `DIVIDE([Adj Close] - [Running Max Adj Close], [Running Max Adj Close])`; **Max Drawdown** is `MIN([Drawdown])` per company. |
| **Stock Return / Market Return** | `BetaReturns` | **Beta** divides covariance of stock and market returns by market-return variance, using only dates common to both series. |
| **Profit of Assets** | `20 Companies` | **ROA** is `AVERAGE('20 Companies'[Profit of Assets])` (Apple leads at 21.12%). |
| **Profit** | `20 Companies` | **Profit Margin** is `AVERAGE('20 Companies'[Profit])` (JPMorgan Chase leads at 31.0%). **Business Strength Score** is `0.5 * [ROA] + 0.5 * [Profit Margin]`, which ranks Facebook first. |
| **Is Tech** | `dimCompany` | Drives every Tech-vs-Non-Tech comparison measure via `CALCULATE([Measure], dimCompany[Is Tech] = "Tech")` and the `"Non-Tech"` equivalent. |
| **Day Index** | `Daily Stock Return` | Sequential trading-day counter per company; COVID measures use explicit calendar-date windows. |

---

## 📊 Dashboard Pages: Technical Deep Dive

The Power BI report relies on the star schema above and layers 81 DAX measures on top of it to deliver each page's insights.

### 📋 Overview Page

| Focus Area | Technical Implementation |
|---|---|
| **KPI Calculations (DAX)** | `Best Total Return Company` / `Worst Total Return Company` use `TOPN(1, ...)` over a virtual table of per-company `[Total Return]`, paired with footer measures that concatenate the value for card display. |
| **Tech vs Non-Tech Trend** | `Tech Avg Adj Close` / `Non-Tech Avg Adj Close` average `Adj Close` by cohort across `dcalendar[Year]`/`dcalendar[Quarter]`. |
| **Dynamic Bar Coloring** | `Total Return Bar Color` returns a hex string conditionally (`IF([Total Return] < 0, "#E06666", "#3FBF8F")`) and is bound to the bar chart's Data Colors via a field-value color measure. |

### 📈 Ranking Page

| Focus Area | Technical Implementation |
|---|---|
| **Sharpe Ratio** | `Sharpe Ratio` = `AVERAGE([Adj Daily Return]) / STDEV.S([Adj Daily Return])`; `Highest/Lowest Sharpe Ratio Company` wrap this in `TOPN`. |
| **Return / Volatility Chart** | The Top 10 chart uses `Total Return / Volatility` (Tesla 115.4, Costco 104.3, Apple 103.0), a different measure from Sharpe Ratio. |
| **Scatter Plot (Return vs Risk)** | Plots `[Volatility]` (X) against `[Total Return]` (Y) at `Company` grain, colored by `dimCompany[Is Tech]`. |
| **Beta** | `Beta` divides covariance by market-return variance using dates common to the stock and benchmark series. Top 3: Delta Air Lines 0.179, American Airlines 0.168, Tesla 0.131. |

### 🦠 COVID Deep Dive Page

| Focus Area | Technical Implementation |
|---|---|
| **COVID Drawdown** | `COVID Drawdown` = `ABS(MIN([Drawdown]))` for `2020-02-20` to `2020-12-31`; American Airlines is deepest at -84.1% on 2020-05-15. |
| **COVID Recovery** | `COVID Recovery` = `(MAX([Adj Close], 2020-03-24 to 2020-12-31) - MIN([Adj Close], 2020-02-20 to 2020-03-23)) / MIN([Adj Close], 2020-02-20 to 2020-03-23)`. It uses calendar windows, not `Day Index`. |
| **Recovery Index** | `Normalized Index` rebases average `Adj Close` to `2020-02-20 = 100` so Tech and Non-Tech recovery trajectories are visually comparable regardless of absolute price level. |
| **COVID Outperformance / Spread** | `COVID Outperformance Value` = average `[COVID Recovery]` for Tech minus Non-Tech; `COVID Recovery Spread` = maximum minus minimum `[COVID Recovery]` across companies. |

### ⚖️ Tech VS Non-Tech Page

| Focus Area | Technical Implementation |
|---|---|
| **Return Gap** | `Return Gap` / `Return Gap Value` = `[Tech Avg Return] - [Non-Tech Avg Return]`; Tech 108.6% vs Non-Tech 54.1% (+54.5 points). |
| **Revenue Totals by Year** | Grouped bar of `SUM('20 Companies'[Annual Rev])` by `dimCompany[Is Tech]` and `dcalendar[Year]`. Non-Tech exceeds Tech in every year from 2018–2022. |
| **Return Stability** | `Return Stability` = `1 / [Volatility]`, so a higher bar means lower adjusted daily return dispersion. |

### 🔎 Company Drill Down Page

| Focus Area | Technical Implementation |
|---|---|
| **Dynamic Titles** | `Price Trend Title` concatenates the slicer-selected company into a string (`SELECTEDVALUE(dimCompany[Company]) & " — Price Trend (2018–2022)"`) so the chart title updates live with the selector. |
| **Trend Signal (MA Crossover)** | `MA30` / `MA90` compute rolling averages over `Daily Stock Return[Adj Close]` using `CALCULATE(AVERAGE(...), DATESINPERIOD(...))`; `Trend Signal` compares the two with an `IF(MA30 > MA90, "Bullish (Golden Cross)", "Bearish (Death Cross)")` pattern. |
| **Revenue Efficiency Trend** | `Revenue per Employee` = `DIVIDE(SUM('20 Companies'[Annual Rev]), SUM('20 Companies'[Number of Employees]))`, filtered to the selected company via `SELECTEDVALUE`. |

### 🏆 Recommendations Page

| Focus Area | Technical Implementation |
|---|---|
| **Investment Simulation** | The "Final Investment Value" table projects a hypothetical fixed investment forward using each company's `[Total Return]` measure: `Final Value = Initial Investment × (1 + [Total Return])`. |
| **Narrative Cards** | Each Q&A card binds a `TOPN(1, ...)` "winner" measure (e.g., `Best Investment`) to a text card, with a paired footer measure supplying the supporting stat sentence. |
| **Investor Profile Table** | Growth maps to highest total return, Balanced to highest Sharpe Ratio, and Conservative to lowest volatility. |

---

## 🧮 Analytical Measures Summary

This project's 81 DAX measures fall into the following functional groups:

- **Return & Ranking:** `Total Return`, `Rank`, `Best/Worst Total Return Company`, `Total Return Bar Color`
- **Risk & Volatility:** `Volatility`, `Max Drawdown`, `Highest/Lowest Volatility Company`, `Volume Spike Count`
- **Sharpe & Market:** `Sharpe Ratio`, `Highest/Lowest Sharpe Ratio Company & Footer`, `Beta`, `Average Beta`
- **COVID-Specific:** `COVID Drawdown`, `COVID Recovery`, `COVID Outperformance Value`, `COVID Recovery Spread`, `Worst COVID Drawdown 2020 Value & Company`
- **Profitability & Fundamentals:** `ROA`, `Profit Margin`, `Profit Margin Avg`, `Avg Profit of Assets`, `Business Strength Score`, `Strongest Business Company`, `Revenue per Employee`, `Total Revenue`
- **Tech vs Non-Tech Comparison:** `Tech/Non-Tech Avg Return`, `Return Gap`, `Return Stability`, `Tech Color`
- **Trend & Technical Indicators:** `MA30`, `MA90`, `Trend Signal`, `Normalized Index`

---

## 🏆 Key Insights

- **Tesla** delivered the highest total return (**+476.5%**) and the strongest COVID-era recovery (**+877%**) off its 2020 low.
- **American Airlines** posted the deepest **2020 COVID** drawdown (**-84.1%**); **Zoom** recorded the largest full-period drawdown (**-88.5%**).
- **Costco** led the **Highest Sharpe Ratio** card (**0.0573**), calculated from adjusted daily returns and sample volatility. The separate Return/Volatility chart (Tesla 115.4, Costco 104.3, Apple 103.0) uses total return instead.
- **Johnson & Johnson** was the most stable holding (lowest adjusted daily volatility, σ = 1.31%).
- **Facebook** ranked first on **Business Strength Score** (`0.5 × ROA + 0.5 × Profit Margin`). Apple led ROA at **21.12%**, while JPMorgan Chase led average profit margin at **31.0%**.
- **Technology companies** averaged **+108.6%** return vs **+54.1%** for Non-Technology, a **+54.5-point** gap. In the 2020 COVID recovery window, Tech averaged **+244.5%** vs Non-Tech **+80.1%**, a **+164.4-point** gap.
- Annual revenue totals place **Non-Tech above Tech in every year**: 2018 $1.384T vs $0.850T; 2019 $1.391T vs $0.959T; 2020 $1.289T vs $1.142T; 2021 $1.551T vs $1.484T; 2022 $1.836T vs $1.655T.

---

## ⚙️ Tools & Technologies Used

| Category | Tool | Purpose |
|---|---|---|
| **Data Modeling** | Power BI Desktop | Star schema construction, relationship management |
| **Calculation Engine** | DAX | 81 measures covering return, risk, profitability, and sector-comparison logic |
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

**Mohamed Basem**
