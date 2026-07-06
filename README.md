Starbucks — Financial Performance Dashboard
A Power BI report analyzing Starbucks' financial performance using data pulled directly from SEC EDGAR's XBRL API. Built alongside a parallel Costco analysis to compare two very different retail economics: high-margin/lower-volume versus low-margin/high-volume.
Live report
View the interactive dashboard
Problem statement
Starbucks' financials show strong seasonality (holiday-driven fiscal Q1 strength) and, in several recent years, negative stockholders' equity due to sustained share buybacks. This project builds an automated pipeline to extract, clean, and model Starbucks' reported financials, while explicitly handling data patterns — like negative equity — that can otherwise produce misleading ratios if not accounted for.
Data source
SEC EDGAR XBRL Company Facts API: `https://data.sec.gov/api/xbrl/companyfacts/CIK0000829224.json`
Filing types used: 10-K (annual), 10-Q (quarterly)
CIK: 0000829224
Data pipeline
Power Query function (`fnGetSECConcept`) pulls individual XBRL concepts (Revenues, GrossProfit, CostOfGoodsAndServicesSold, OperatingIncomeLoss, NetIncomeLoss, Assets, Liabilities, StockholdersEquity) from the EDGAR API
Results are combined into a single long-format fact table
Duplicate reporting periods are deduplicated by keeping the most recently filed record per period
A calculated `Dim\_Date` table generates a full calendar aligned to Starbucks' fiscal year (year-end in late September/early October)
Key data quality issue and fix
Unlike Costco, Starbucks does report a `GrossProfit` tag in some years but not others. A coalesce pattern uses the reported figure where available and falls back to a calculated value otherwise:
```dax
Gross Profit = 
VAR DirectTag = CALCULATE(SUM(Fact\_Financials\[val]), Fact\_Financials\[Concept] = "GrossProfit")
VAR Calculated = \[Revenue] - CALCULATE(SUM(Fact\_Financials\[val]), Fact\_Financials\[Concept] = "CostOfGoodsAndServicesSold")
RETURN COALESCE(DirectTag, Calculated)
```
Starbucks has carried negative stockholders' equity in recent fiscal years. Return on Equity is guarded against producing a misleading result in this case:
```dax
Return on Equity = 
VAR Equity = \[Stockholders Equity]
RETURN IF(Equity > 0, DIVIDE(\[Net Income], Equity), BLANK())
```
Model structure
`Fact\_Financials` (long format: Concept, val, start, end, fy, fp, form, filed)
`Dim\_Date` (calculated date table, marked as the model's official date table)
Key measures
```dax
Revenue = CALCULATE(SUM(Fact\_Financials\[val]), Fact\_Financials\[Concept] = "Revenues")

Gross Margin % = DIVIDE(\[Gross Profit], \[Revenue])

Operating Margin % = DIVIDE(\[Operating Income], \[Revenue])

Net Margin % = DIVIDE(\[Net Income], \[Revenue])

Revenue YoY % = 
VAR CurrentRev = \[Revenue]
VAR PriorRev = CALCULATE(\[Revenue], SAMEPERIODLASTYEAR(Dim\_Date\[Date]))
RETURN DIVIDE(CurrentRev - PriorRev, PriorRev)
```
Report pages
Executive Summary — headline KPIs, revenue trend, revenue vs net income
Profitability Deep Dive — revenue-to-net-income waterfall, margin trend lines
Balance Sheet & Financial Health — assets/liabilities/equity, debt-to-equity, ROE (with negative-equity handling noted)
Quarterly Trend — holiday-driven seasonality, quarter-over-quarter growth
Methodology — data sources, extraction date, XBRL concepts used
Theme
Custom Power BI theme (`Starbucks\_Theme.json`) built on Starbucks' brand palette — signature green (#00704A) on a warm cream background (#F2F0EB).
Tools used
Power BI Desktop, Power Query (M), DAX, SEC EDGAR XBRL API
Disclaimer
All financial figures are sourced directly from Starbucks' public SEC filings. This project is for portfolio/demonstration purposes and is not affiliated with or endorsed by Starbucks Corporation.
