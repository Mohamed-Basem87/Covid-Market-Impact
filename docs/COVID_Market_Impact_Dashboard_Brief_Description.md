# COVID Market Impact Dashboard — Project Description

**Tool:** Microsoft Power BI | **Type:** Financial Analytics Portfolio Project

## Overview
The COVID Market Impact Dashboard is a six-page Power BI report analyzing how COVID-19 affected the stock performance, risk profile, and financial fundamentals of 20 Fortune-500-class companies from 2018–2022. The analysis compares Technology vs. Non-Technology companies across return, volatility, drawdown, beta, and profitability metrics, then translates the findings into investor-facing recommendations.

## What It Does
- Tracks daily stock performance (24,500+ rows) and annual financials for 20 companies across five years.
- Calculates industry-standard metrics: total return, volatility, max drawdown, market beta, Sharpe (risk-adjusted) ratio, ROA, and moving-average trend signals.
- Isolates the COVID-19 shock and recovery period specifically, measuring drawdown depth and recovery speed by company and by sector.
- Segments companies into Tech vs. Non-Tech cohorts to quantify the performance and resilience gap between them.
- Delivers a company drill-down view and a final recommendations page mapping stocks to investor risk profiles (Growth / Balanced / Conservative).

## Data Model
A star-schema model with two dimensions (Company, Calendar) and three fact tables (daily stock returns, market beta returns, annual company financials), driven by ~90 DAX measures.

## Key Findings
- **Tesla** delivered the highest total return (+476.5%) and the strongest COVID recovery (+877%).
- **American Airlines** suffered the deepest drawdown (-84.5%) and weakest risk-adjusted return.
- **Apple** offered the best risk-adjusted return, making it the strongest balanced pick.
- **Tech companies** outperformed Non-Tech by +67 points on average return and recovered from the pandemic roughly twice as fast.

## Skills Demonstrated
Power BI data modeling, DAX measure design, financial/risk analytics, dashboard UX design, and data storytelling.
