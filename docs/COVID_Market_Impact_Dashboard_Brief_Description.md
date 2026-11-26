# COVID Market Impact Dashboard — Project Description

**Tool:** Microsoft Power BI | **Type:** Financial Analytics Portfolio Project

## Overview
The COVID Market Impact Dashboard is a six-page Power BI report analyzing how COVID-19 affected the stock performance, risk profile, and financial fundamentals of 20 Fortune-500-class companies from 2018–2022. The analysis compares Technology vs. Non-Technology companies across return, volatility, drawdown, beta, and profitability metrics, then translates the findings into investor-facing recommendations. The PBIX and source CSVs are not committed here, but the measures and values below were verified against an author-supplied local copy.

## What It Does
- Tracks daily stock performance (24,500+ rows) and annual financials for 20 companies across five years.
- Calculates industry-standard metrics: total return, volatility, max drawdown, market beta, Sharpe ratio (average daily return / sample volatility, without a risk-free rate), ROA, and moving-average trend signals.
- Isolates the COVID-19 shock and recovery period specifically, measuring drawdown depth and recovery speed by company and by sector.
- Segments companies into Tech vs. Non-Tech cohorts to quantify the performance and resilience gap between them.
- Delivers a company drill-down view and a final recommendations page mapping stocks to investor risk profiles (Growth / Balanced / Conservative).

## Data Model
A star-schema model with two dimensions (Company, Calendar) and three fact tables (daily stock returns, market beta returns, annual company financials), driven by 81 DAX measures.

## Key Findings
- **Tesla** delivered the highest total return (+476.5%) and the strongest COVID recovery (+877%).
- **American Airlines** suffered the deepest **2020 COVID** drawdown (-84.5%); **Zoom** had the largest full-period drawdown (-88.5%).
- The **Highest Sharpe Ratio** card identifies **Apple** (0.0562); the formula is average daily return divided by sample standard deviation, with no risk-free-rate subtraction. A separate chart uses **Total Return / Volatility** (Tesla 147, Apple 128, Microsoft 119) and is not a Sharpe ratio.
- **Facebook** ranked first on **Business Strength Score** (`0.5 × ROA + 0.5 × Profit Margin`). Apple led ROA at 21.12%; JPMorgan Chase led average profit margin at 31.0%.
- **Tech companies** outperformed Non-Tech by **+67 percentage points** on average return (105% vs 38%); in the 2020 COVID recovery window, Tech averaged +244.5% vs Non-Tech +80.1% (+164.4 points). No recovery-speed multiple was calculated.
- Verified source sums show **Non-Tech revenue above Tech in every year** (2018: $1.384T vs $0.850T; 2020: $1.289T vs $1.142T; 2022: $1.836T vs $1.655T). The committed screenshot instead labels Tech above Non-Tech from 2020 and shows $1.66T vs $1.44T in 2022; that visual needs manual correction.

## Skills Demonstrated
Power BI data modeling, DAX measure design, financial/risk analytics, dashboard UX design, and data storytelling.
