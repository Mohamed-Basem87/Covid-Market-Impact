# COVID Market Impact Dashboard — Project Description

**Tool:** Microsoft Power BI | **Type:** Financial Analytics Portfolio Project

## Overview
The COVID Market Impact Dashboard is a six-page Power BI report analyzing how COVID-19 affected the stock performance, risk profile, and financial fundamentals of 20 Fortune-500-class companies from 2018–2022. It compares Technology vs. Non-Technology companies across return, volatility, drawdown, beta and profitability metrics.

## What It Does
- Tracks daily stock performance (24,500+ rows) and annual financials for 20 companies across five years.
- Calculates total return, volatility, drawdown, beta and Sharpe from split/dividend-adjusted close-to-close returns, with beta aligned to common stock and benchmark dates.
- Isolates the COVID-19 shock and recovery period specifically, measuring drawdown depth and recovery speed by company and by sector.
- Segments companies into Tech vs. Non-Tech cohorts to quantify the performance and resilience gap between them.
- Delivers a company drill-down view and a final recommendations page mapping stocks to investor risk profiles (Growth / Balanced / Conservative).

## Data Model
A star-schema model with two dimensions (Company, Calendar) and three fact tables (daily stock returns, market beta returns, annual company financials), driven by 81 DAX measures.

## Key Findings
- **Tesla** delivered the highest total return (+476.5%) and the strongest COVID recovery (+877%).
- **American Airlines** had the deepest **2020 COVID** drawdown (-84.1%); **Zoom** had the largest full-period drawdown (-88.5%).
- **Costco** led the **Highest Sharpe Ratio** card (0.0573). The separate Return/Volatility chart uses total return (Tesla 115.4, Costco 104.3, Apple 103.0).
- **Facebook** ranked first on **Business Strength Score** (`0.5 × ROA + 0.5 × Profit Margin`). Apple led ROA at 21.12%; JPMorgan Chase led average profit margin at 31.0%.
- **Tech companies** averaged +108.6% return vs +54.1% for Non-Tech, a +54.5-point gap. In the 2020 COVID recovery window, Tech averaged +244.5% vs Non-Tech +80.1% (+164.4 points).
- Annual revenue totals place **Non-Tech above Tech in every year**: 2018 $1.384T vs $0.850T; 2019 $1.391T vs $0.959T; 2020 $1.289T vs $1.142T; 2021 $1.551T vs $1.484T; 2022 $1.836T vs $1.655T.

## Skills Demonstrated
Power BI data modeling, DAX measure design, financial/risk analytics, dashboard UX design, and data storytelling.
