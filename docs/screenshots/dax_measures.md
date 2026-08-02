# 📐 DAX Measures & Data Model Documentation

This document logs all explicit DAX measures and calculated tables built for the **Retail Sales Performance & Profitability Analysis** Power BI dashboard.

---

## 1. Data Model & Dimension Tables

### `Dim_Date` (Calendar Dimension)
* **Type:** Calculated Table
* **Purpose:** Provides a continuous date range for Time Intelligence calculations and temporal cross-filtering across transaction gaps.

```dax
Dim_Date = 
VAR MinYear = YEAR(MIN(retail_store_sales[Transaction Date]))
VAR MaxYear = YEAR(MAX(retail_store_sales[Transaction Date]))
RETURN
ADDCOLUMNS (
    CALENDAR(DATE(MinYear, 1, 1), DATE(MaxYear, 12, 31)),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMM"),
    "Month Year", FORMAT([Date], "MMM YYYY"),
    "Month Year Sort", YEAR([Date]) * 100 + MONTH([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Day of Week", FORMAT([Date], "dddd"),
    "Day Number", DAY([Date])
)
```

## 2. Core Financial KPIs & DAX Measures

### `Total Sales`
* **Description:** Calculates total net revenue generated across all transactions.
* **Format:** Currency (`$`)

```dax
Total Sales = SUM(retail_store_sales[Total Spent])
```

### `Total Orders`
* **Description:** Counts unique order transactions.
* **Format:** Whole Number (123)

```dax
Total Orders = DISTINCTCOUNT(retail_store_sales[Transaction ID])
```

### `Total Quantity Sold`
* **Description:** Sums total units sold across all line items.
* **Format:** Whole Number (123)

```dax
Total Quantity Sold = SUM(retail_store_sales[Quantity])
```

### `AOV (Average Order Value)`
* **Description:** Calculates the average revenue generated per distinct transaction using division-by-zero protection.
* **Format:** Currency ($)

```dax
AOV = DIVIDE([Total Sales], [Total Orders], 0)
```

### `Sales PY (Prior Year Sales)`
* **Description:** Evaluates [Total Sales] shifted back exactly 1 year using DAX Time Intelligence.
* **Format:** Currency ($)

```dax
Sales PY = 
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

### `YoY Sales Growth %`
* **Description:** Calculates percentage revenue growth compared against the prior year's equivalent time period.
* **Format:** Percentage (%) with 2 decimal places

```dax
YoY Sales Growth % = 
VAR CurrentSales = [Total Sales]
VAR PriorSales = [Sales PY]
RETURN
    DIVIDE(CurrentSales - PriorSales, PriorSales, 0)
```