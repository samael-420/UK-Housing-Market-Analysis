# UK-Housing-Market-Analysis

Analysis of 22 million UK property transactions using Python and Power BI, exploring price trends, regional inequality, property type across England and Wales.
 
---
 
## Dashboard Preview
 
**Page 1 — Market Overview**
![Market Overview](ukhousepre1.png)
 
**Page 2 — Regional Analysis**
![Regional Analysis](ukhousepre2.png)
 
---
 
## Project Overview
 
Using the UK Land Registry Price Paid dataset (22,489,348 transactions across 11 columns), this project analyses house price trends from 1995 to 2017. Due to the scale of the data, I used python to clean and pre-aggregate the dataset into summary tables before loading into Power BI to optimize performance
 
**Business Questions Answered:**
1. How have UK house prices trended over time?
2. Which regions are most expensive and which have grown fastest?
3. Which property types hold the most value?
4. How much more expensive are new builds vs existing properties?
5. Which towns sit at the top and bottom of the market?
 
---
 
## Key Findings
 
- UK median house prices grew consistently from 1995 to 2017, with notable slowdowns post-2008 financial crisis
- Strong regional inequality — a significant majority of counties sit **below** the national median, pulled upward by London and the South East
- **Detached houses** command the highest median price; **flats** the lowest
- New builds carry a consistent premium over existing properties throughout the entire period
- Sales volume collapsed sharply around **2008**, reflecting the financial crisis impact on market activity before gradually recovering
 
---
 

## Data Model (Star Schema)
 
```
[DateTable]
     |                    |
     | date → date        | date → date
     ↓                    ↓
[annual_summary]    [regional_summary]
                          |
                          | county → county (many to one)
                          ↓
                    [county_lookup] ── county → county ──→ [town_summary]
 
[Measures] - measures only table, no relationships
```
 
 
---
 
## Challenges & How They Were Solved
 
### 1. Query Exceeded Available Resources
**Problem:** Power BI kept crashing when trying to calculate `MEDIANX` across 22 million raw rows.
 
**Solution:** Pre-aggregated the data in Python using Pandas `groupby` and `median` before loading into Power BI. This reduced the data Power BI needed to process from 22M rows to a few thousand summary rows, eliminating the memory issue entirely. This also improved dashboard load times significantly.
 
---
 
### 2. Duplicate Value Relationship Error
**Problem:** When trying to connect `housing_regional_summary[county]` to `housing_town_summary[county]`, Power BI threw an error:
> *"Column 'county' contains a duplicate value 'AVON' and this is not allowed for columns on the one side of a many-to-one relationship"*
 
This happened because `regional_summary` has one row per county **per year** , so every county appears multiple times, making it unsuitable for the "one" side of a relationship.
 
**Solution:** Created a separate `county_lookup` table in Python with exactly one row per county. This acts as a proper dimension table in the star schema, sitting on the "one" side of relationships to both `regional_summary` and `town_summary`. This is standard dimensional modelling practice.
 
---
 
## DAX Measures used
 
```
Total Sales = COUNTROWS(housing_annual_summary)
 
Median Price = 
DIVIDE(
    SUMX(housing_annual_summary,
         housing_annual_summary[median_price] * housing_annual_summary[total_sales]),
    SUM(housing_annual_summary[total_sales]), 0)
 
Regional Median Price = AVERAGE(housing_regional_summary[median_price])
 
Town Median Price = AVERAGE(housing_town_summary[median_price])
 
New Build Median Price = 
CALCULATE(MEDIANX(housing_annual_summary, housing_annual_summary[median_price]),
          housing_annual_summary[build_type] = "New Build")
 
Existing Median Price = 
CALCULATE(MEDIANX(housing_annual_summary, housing_annual_summary[median_price]),
          housing_annual_summary[build_type] = "Existing")
 
New Build Premium % = 
DIVIDE([New Build Median Price] - [Existing Median Price],
       [Existing Median Price], 0) * 100
 
Price Growth % = 
VAR EarlyPrice = CALCULATE(AVERAGE(housing_regional_summary[median_price]),
                            housing_regional_summary[year] = 1995)
VAR LatestPrice = CALCULATE(AVERAGE(housing_regional_summary[median_price]),
                             housing_regional_summary[year] = 2017)
RETURN DIVIDE(LatestPrice - EarlyPrice, EarlyPrice, 0) * 100
```
 
---
 
## Project Structure
 
```
├── uk_housing.ipynb        # Full analysis and aggregation notebook
├── housing_annual_summary.csv       # Aggregated by year, property type, build type
├── housing_regional_summary.csv     # Aggregated by county and year
├── housing_town_summary.csv         # Aggregated by town and county
├── housing_county_lookup.csv        # One row per county (dimension table)
├── ukhousepre1.png              # Market Overview screenshot
├── ukhousepre1.png              # Regional Analysis screenshot
└── README.md
```
 
---
 
## Tools & Libraries used
 
- **Python** — Pandas, NumPy, Matplotlib, Seaborn, Plotly
- **Power BI** — DAX measures, star schema data model, interactive dashboard
- **Jupyter Notebook**
 
---
 
## Dataset
 
UK Land Registry Price Paid Data — publicly available at [GOV.UK](https://www.gov.uk/government/statistical-data-sets/price-paid-data)
 
- **22,489,348 transactions**
- **11 columns** including price, date, property type, build type, and location
- Covers England and Wales, 1995–2017
 
---
 
## Why Median Not Mean?
 
Throughout this project, **median** is used instead of mean for all price calculations. A small number of multi-million pound transactions would significantly distort the mean upward, misrepresenting what a typical buyer pays. Median is the same methodology used by Halifax, Nationwide, and the ONS in their official house price indices.
 
---
 
## Author
 
**Rutvik Randive**
MSc Data Science and Analytics — Cardiff University
[LinkedIn](https://your-linkedin-url) | [GitHub](https://github.com/YOUR-USERNAME)
