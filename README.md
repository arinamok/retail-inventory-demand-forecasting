# Retail Inventory Optimization & Demand Forecasting
SQL Server | Python | Power BI

An end-to-end retail inventory and demand forecasting project using SQL, Python, and Power BI. Analyzed stockout and overstock risks across a multi-store retail operation, engineered inventory metrics, and built a Random Forest model to forecast 30-day product demand and support reorder decisions.

---

## The Business Problem
A multi-store retail operation was losing revenue to two opposite problems at the same time: shelves running empty on high-demand products, while other products sat unsold. The goal was to understand why, forecast what demand would look like over the next 30 days, and give store managers a clear action list.

**Key Findings:**
- 69% of all store/product combinations are critically low on stock and at risk of running out. Only 6% are in a healthy state.
- Clothing runs out the fastest (14.3% stockout rate) while Furniture sits unsold the longest (20.8% overstock rate).
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

## Dataset

- **Source:** [Kaggle – Retail Store Inventory and Demand Forecasting](https://www.kaggle.com/datasets/atomicd/retail-store-inventory-and-demand-forecasting)
- **Rows:** 76,000
- **Stores:** 5
- **Products:** 20
- **Categories:** 5 (Clothing, Electronics, Furniture, Groceries, Toys)
- **Regions:** 4
- **Date Range:** Jan 2022 – Jan 2024
- **Null Values:** 0

| Column | Description |
|---|---|
| `Date` | Date of the sales record |
| `Store_ID` | Unique store identifier |
| `Product_ID` | Unique product identifier |
| `Category` | Product category |
| `Region` | Geographic region of the store |
| `Inventory_Level` | Current inventory available |
| `Units_Sold` | Units sold that day |
| `Units_Ordered` | Units ordered for replenishment |
| `Price` | Product selling price |
| `Discount` | Discount applied |
| `Weather_Condition` | Weather during the sales period |
| `Promotion` | Whether a promotion was active (1 = Yes, 0 = No) |
| `Competitor_Pricing` | Competitor price for similar products |
| `Seasonality` | Seasonal period (Winter, Spring, etc.) |
| `Epidemic` | Whether an epidemic period was active (1 = Yes, 0 = No) |
| `Demand` | Estimated daily product demand |

---

## SQL: Data Validation

The dataset came in clean from Kaggle. Validation confirmed:
- 76,000 rows, 0 nulls, 0 duplicates
- No negative values across inventory, units sold, or demand
- No logical errors (units sold exceeding inventory level returned 0 rows)
- 5 stores, 20 products, 5 categories, 4 regions confirmed

---

## SQL: Feature Engineering

Three inventory flags and two time features were created directly in SQL before passing the data to Python.

**Stockout:** flagged when units sold met or exceeded the inventory level 
AND demand exceeded inventory, meaning the product genuinely ran out:
```sql
UPDATE sales_data
SET Stockout =
    CASE
        WHEN Units_Sold >= Inventory_Level
             AND Demand > Inventory_Level
        THEN 1
        ELSE 0
    END;
```

**Overstock:** flagged when inventory level exceeded 5x average daily 
demand for that store/product combination:
```sql
WITH demand_calc AS (
    SELECT Store_ID, Product_ID,
           AVG(Demand) AS avg_daily_demand
    FROM sales_data
    GROUP BY Store_ID, Product_ID
)
UPDATE s
SET Overstock =
    CASE
        WHEN s.Inventory_Level > (d.avg_daily_demand * 5)
        THEN 1 ELSE 0
    END
FROM sales_data s
JOIN demand_calc d
    ON s.Store_ID = d.Store_ID
    AND s.Product_ID = d.Product_ID;
```

**Reorder_Needed** - flagged when inventory level fell below 5x average 
daily demand, signaling it was time to reorder:
```sql
WITH demand_calc AS (
    SELECT Store_ID, Product_ID,
           AVG(Demand) AS reorder_point
    FROM sales_data
    GROUP BY Store_ID, Product_ID
)
UPDATE s
SET Reorder_Needed =
    CASE
        WHEN s.Inventory_Level < (d.reorder_point * 5)
        THEN 1 ELSE 0
    END
FROM sales_data s
JOIN demand_calc d
    ON s.Store_ID = d.Store_ID
    AND s.Product_ID = d.Product_ID;
```

**Month_Num** and **Day_Of_Week** were added to capture seasonality and weekly purchasing patterns for the forecasting model.

---

## SQL: Business Analysis

- Overall stockout rate: **10.3%** | Overall overstock rate: **15.9%**
- Clothing had the highest stockout rate (14.3%), Furniture the lowest (5.9%)
- Furniture had the highest overstock rate (20.8%), Toys the lowest (11.4%)
- Promotions increased average demand from 95 to 123 (+29.7%) and average 
  units sold from 81.8 to 103.1
- South and East regions showed the highest demand, and West the lowest
- Stockout rate during promotions was higher, confirming stores are not 
  stocking up ahead of promotional periods

---

## Python: EDA & Inventory Analysis

- Groceries had the highest average demand (121) and the highest volatility
- Furniture had the lowest average demand (73.6)
- Demand dropped approximately 38% during epidemic periods (~113 normal 
  vs ~70 epidemic)
- Demand and Units_Sold correlation: 0.83
- Price and Competitor_Pricing correlation: 0.98 (prices move together 
  closely across competitors)
- Clothing had the highest inventory turnover (0.52), Furniture the 
  lowest (0.37)

### Inventory Risk Classification

A risk framework was built using reorder points, inventory levels, rolling demand trends, and weeks of supply:

| Risk Level | Count | Share |
|---|---|---|
| Critical Stockout Risk | 69 | 69% |
| At Risk | 25 | 25% |
| Healthy | 6 | 6% |
| Overstocked | 0 | 0% |

69% of all store/product combinations were classified as Critical Stockout 
Risk, indicating widespread understocking across the operation.

---

## Python: Feature Engineering

| Feature | Purpose |
|---|---|
| `rolling_7d_demand` | Captures short-term demand trends |
| `inventory_turnover` | Measures inventory efficiency |
| `Month_Num` | Monthly seasonality (from SQL) |
| `Day_Of_Week` | Weekly purchasing behavior (from SQL) |

---

## Demand Forecasting Model

A Random Forest Regressor was trained to forecast daily product demand 
over a 30-day horizon.

**Features used:** rolling 7-day demand, price, discount, promotions, 
competitor pricing, inventory levels, seasonal indicators

### Model Performance

| Metric | Result |
|---|---|
| MAE | 17.92 |
| RMSE | 24.60 |
| R² Score | 0.69 |
| MAPE | 26% |

Predictions were off by about 18 units on average. The model explained 
69% of demand variance and tracked actual demand reasonably well.

### Top Demand Drivers

| Feature | Importance |
|---|---|
| rolling_7d_demand | 46% |
| Price | 11.7% |
| Promotion | 7.8% |
| Competitor_Pricing | 6.9% |

Recent sales history (rolling 7-day demand) was by far the strongest 
predictor, meaning last week's demand is the best signal for this week's 
orders.

### 30-Day Forecast Results
- Clothing projected as highest-demand category
- Furniture projected as the lowest-demand category

---

## Recommendations

| Recommendation | Reason |
|---|---|
| Increase safety stock for Clothing | Highest stockout rate (14.3%) |
| Reduce Furniture order frequency | Highest overstock rate (20.8%) |
| Stock up before promotions go live | Promotions spike demand by ~30% |
| Monitor epidemic-sensitive categories | Demand drops 38% during epidemic periods |
| Use rolling 7-day demand for reorder triggers | Strongest forecasting driver at 46% importance |

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
