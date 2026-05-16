# Retail Inventory Optimization & Demand Forecasting
### SQL Server | Python | Power BI

Built an end-to-end retail inventory optimization and demand forecasting project using SQL, Python, and Power BI. Analyzed stockout and overstock risks, identified key demand drivers, engineered inventory metrics, and developed a Random Forest forecasting model to support inventory planning and reorder optimization.

---
## The Business Problem
A multi-store retail operation was losing revenue to two opposite problems at the same time: shelves running empty on high-demand products while other products sat unsold. The goal was to understand why, forecast what demand would look like over the next 30 days, and give store managers a clear action list.

### Business Question

> *How can a multi-store retail operation reduce stockouts and overstock situations by accurately forecasting daily product demand and optimizing reorder decisions across stores, categories, and regions?*

### Key Findings:
- 69% of all store/product combinations are critically low on stock and at risk of running out. Only 6% are in a healthy state. This is a widespread problem, not isolated to one store or category.
- Clothing runs out the fastest (14.3% stockout rate) while Furniture sits unsold the longest (20.8% overstock rate). It leads to opposite decisions to order more clothing, slow down furniture orders.
- Promotions increase demand by about 30%. Stores are not accounting for this spike when placing orders, which is likely making the stockout problem worse.
  
--- 

## Dashboard Preview

### Page 1: Inventory Health & Risk

<p align="center">
  <img src="dashboard/Inventory Health & Risk.PNG"
       width="900"
       style="border:1px solid #ddd; border-radius:6px;" />
</p>

<br>

### Page 2: Demand Drivers & Forecasting
<p align="center">
  <img src="dashboard/Demand Drivers & Forecasting.PNG"
       width="900"
       style="border:1px solid #ddd; border-radius:6px;" />
</p>


---

## 1. Project Overview

This project analyzes inventory performance and demand patterns across a multi-store retail operation to reduce stockouts and overstock situations using forecasting and inventory risk analysis.

The project combines SQL, Python, and Power BI to:
- analyze inventory health
- identify key demand drivers
- forecast future product demand
- optimize reorder decision-making across stores and categories

---

## 2. Dataset

**Dataset Source:** [Kaggle – Retail Store Inventory and Demand Forecasting](https://www.kaggle.com/datasets/atomicd/retail-store-inventory-and-demand-forecasting)

### Dataset Summary

| Metric | Value |
|---|---|
| Total Rows | 76,000 |
| Stores | 5 |
| Products | 20 |
| Categories | 5 |
| Regions | 4 |
| Date Range | Jan 2022 – Jan 2024 |
| Null Values | 0 |

### Dataset Columns

| Column | Description |
|---|---|
| `Date` | Date of the sales record |
| `Store_ID` | Unique identifier for each retail store |
| `Product_ID` | Unique identifier for each product |
| `Category` | Product category |
| `Region` | Geographic region of the store |
| `Inventory_Level` | Current inventory available |
| `Units_Sold` | Number of units sold that day |
| `Units_Ordered` | Units ordered for replenishment |
| `Price` | Product selling price |
| `Discount` | Discount applied to the product |
| `Weather_Condition` | Weather conditions during the sales period |
| `Promotion` | Indicates whether a promotion was active (1 = Yes, 0 = No) |
| `Competitor_Pricing` | Competitor price for similar products |
| `Seasonality` | Seasonal period (Winter, Spring, etc.) |
| `Epidemic` | Indicates epidemic impact period (1 = Yes, 0 = No) |
| `Demand` | Estimated daily product demand |

---

## 3. SQL Data Cleaning & Validation

### SQL Validation Checks
Checks performed:
- negative values
- logical inconsistencies
- duplicate rows
- missing values
- inventory anomalies

### SQL Feature Engineering

Created:
- `Stockout`
- `Overstock`
- `Reorder_Needed`

Example:

```sql
UPDATE sales_data
SET Stockout =
    CASE
        WHEN Units_Sold >= Inventory_Level THEN 1
        ELSE 0
    END;
```

---

## 4. Python Analysis & Feature Engineering

Python was used for:
- exploratory data analysis (EDA)
- inventory analysis
- demand trend analysis
- feature engineering
- machine learning forecasting

### Engineered Features

| Feature | Purpose |
|---|---|
| `rolling_7d_demand` | Captures short-term demand trends |
| `inventory_turnover` | Measures inventory efficiency |
| `Month_Num` | Captures monthly seasonality |
| `Day_Of_Week` | Captures weekly purchasing behavior |

### Example Feature Engineering

```python
df["rolling_7d_demand"] = (
    df.groupby(["Store_ID", "Product_ID"])["Demand"]
    .transform(lambda x: x.rolling(7).mean())
)
```

---

## 5. Exploratory Data Analysis (EDA)

### Demand Patterns
- Groceries had the highest average demand (**121**)
- Furniture had the lowest average demand (**73.6**)
- Groceries showed the highest demand volatility

### Promotion Impact

| Metric | No Promotion | Promotion |
|---|---|---|
| Average Demand | 95 | 123 |
| Average Units Sold | 81.8 | 103.1 |

Demand increased approximately **29.7%** during promotional periods.

### Regional Demand
- South and East regions showed the highest demand
- West region showed the lowest average demand

### Epidemic Impact

| Period | Average Demand |
|---|---|
| Normal Periods | ~113 |
| Epidemic Periods | ~70 |

Demand declined approximately **38%** during epidemic periods.

### Inventory Health

#### Stockout Rate
- Overall stockout rate: **10.3%**
- Worst category: Clothing (**14.3%**)
- Best category: Furniture (**5.9%**)

#### Overstock Rate
- Overall overstock rate: **15.9%**
- Worst category: Furniture (**20.8%**)
- Best category: Toys (**11.4%**)

### Inventory Turnover
- Clothing had the highest turnover (**0.52**)
- Furniture had the lowest turnover (**0.37**)

### Correlation Findings
- Demand and Units_Sold correlation: **0.83**
- Price and Competitor_Pricing correlation: **0.98**

---

## 6. Inventory Risk Analysis

A risk classification framework was developed using:
- reorder points
- inventory levels
- rolling demand trends
- weeks of supply

### Risk Classification Results

| Risk Level | Count |
|---|---|
| Critical Stockout Risk | 69 |
| At Risk | 25 |
| Healthy | 6 |
| Overstocked | 0 |

### Key Finding
69% of all store/product combinations were classified as **Critical Stockout Risk**, indicating widespread understocking across multiple store locations.

---

## 7. Demand Forecasting Model

A Random Forest Regressor was developed to forecast retail demand.

### Features Used
- inventory levels
- price
- discount
- promotions
- competitor pricing
- rolling 7-day demand
- seasonal indicators

### Model Evaluation

| Metric | Result |
|---|---|
| MAE | 17.92 |
| RMSE | 24.60 |
| R² Score | 0.69 |
| MAPE | 26% |

### Model Interpretation
- Predictions were off by approximately **18 units on average**
- The model explained **69% of demand variance**
- Forecasts tracked actual demand reasonably well

### Top Demand Drivers

| Feature | Importance |
|---|---|
| rolling_7d_demand | 46% |
| Price | 11.7% |
| Promotion | 7.8% |
| Competitor_Pricing | 6.9% |

### Forecast Results
30-day forecasts predicted:
- Clothing as the highest-demand category
- Furniture as the lowest-demand category


---

## 8. Business Recommendations

| Recommendation | Reason |
|---|---|
| Increase safety stock for Clothing | Highest stockout rate |
| Reduce Furniture inventory levels | Highest overstock rate |
| Expand promotion strategies selectively | Promotions increased demand by ~30% |
| Monitor epidemic-sensitive categories closely | Demand dropped 38% during epidemic periods |
| Use rolling demand trends for reorder decisions | Strongest forecasting driver |

---

## Repository Structure

```
├── sql_queries/
│   ├── DataCleaning_1.sql
│   ├── FeatureEngineering_2.sql
│   └── Analysis_3.sql
├── notebook/
│   └── InventoryOptimizationAndDemandForecasting.ipynb
├── data/
│   ├── sales_data.csv
│   ├── sales_data_final.csv
│   ├── actuals_vs_predicted.csv
│   ├── forecast_30day.csv
│   ├── model_metrics.csv
│   └── risk_table.csv
├── dashboard/
│   ├── inventory_analysis_dashboard.pbix
│   ├── Inventory_Health_Risk.PNG
│   └── Demand_Drivers_Forecasting.PNG
└── README.md
```

