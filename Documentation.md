# Retail Sales Dashboard — Documentation

## 1. Project Aim / Goal

The goal of this project is to turn raw, transaction-level retail sales data into a decision-ready Power BI dashboard that lets a business user quickly answer:

- Which regions, cities, and channels drive the most sales and profit
- Which product categories and brands perform best
- How sales and profit trend year-over-year and month-over-month
- How performance breaks down once filtered by region, city, category, month, or year

The dashboard is built as a portfolio piece demonstrating end-to-end BI skills: data modeling (star schema), data cleaning (Power Query), DAX measure design, and dashboard/report design in Power BI.

---

## 2. Dataset Description

**Source file:** `retail-store-multiyear-dataset.xlsx`
**Sheet:** `Retail_Sales_MultiYear`
**Size:** 5,000 order-line records × 20 columns
**Time span:** January 1, 2022 – December 31, 2025 (4 years)

| Column | Description |
|---|---|
| OrderID | Unique order line identifier |
| OrderDate | Date of the transaction |
| CustomerID / CustomerName | Customer identifier and name (500 unique customers) |
| City / Region | Customer location — 5 cities (Bangalore, Chennai, Delhi, Kolkata, Mumbai) across 4 regions (North, South, East, West) |
| ProductID / ProductName | Product identifier and name (200 unique products) |
| Category / Brand | 5 categories (Beauty, Clothing, Electronics, Grocery, Home) across 5 brands (Brand A–E) |
| StoreID / StoreName | Store identifier and name (5 stores) |
| Channel | Sales channel — Online or Store |
| SalespersonID | Salesperson associated with the order |
| PaymentMode | Card, Cash, or UPI |
| Quantity | Units sold |
| UnitPrice | Price per unit |
| DiscountPct | Discount applied (0%, 5%, 10%, 15%, 20%, 25%) |
| CostPrice | Cost per unit |
| ReturnFlag | 1 if the order was returned, 0 otherwise |

The source data was found to be complete — no missing values in any of the 20 columns across the 5,000 records.

---

## 3. Data Model

Built as a **star schema** to keep the model fast, simple to navigate, and easy to filter — one central fact table connected to four descriptive dimension tables, each in a one-to-many relationship with Fact Sales.

```
                Dim Customer
                     │
Dim Calender ── Fact Sales ── Dim Product
                     │
                 Dim Store
```

| Table | Role | Key Columns |
|---|---|---|
| **Fact Sales** | Transactions | OrderID, OrderDate, CustomerID, ProductID, StoreID, Sales, Profit, Quantity, CostPrice, DiscountPct, PaymentMode, ReturnFlag, SalespersonID |
| **Dim Customer** | Customer attributes | CustomerID, CustomerName, City, Region |
| **Dim Product** | Product attributes | ProductID, ProductName, Category, Brand |
| **Dim Store** | Store attributes | StoreID, StoreName, Channel, SalespersonID, Store Key |
| **Dim Calender** | Date intelligence | Date, Month, Quarter, Weekday, Year |

Relationships are all single-direction, one-to-many, from each dimension's key into Fact Sales — the standard pattern for reliable slicer and filter behavior.

---

## 4. Data Cleaning & Preparation Process

Performed in Power Query before loading into the model:

1. **Type validation** — confirmed OrderDate as a proper Date type, numeric fields (Quantity, UnitPrice, DiscountPct, CostPrice) as decimal/whole number, and ID fields as text to prevent unwanted aggregation.
2. **Null check** — profiled all 20 columns; confirmed zero missing values, so no imputation was required.
3. **Table split** — normalized the single flat export into the star schema by extracting Customer, Product, Store, and Calendar attributes into their own dimension tables, keeping Fact Sales limited to transactional measures and foreign keys.
4. **Calendar table** — generated a continuous date table (`Dim Calender`) spanning 2022–2025 with Month, Quarter, Weekday, and Year columns to support time-intelligence DAX.
5. **Relationship setup** — connected each dimension table to Fact Sales on its key column, validating cardinality as one-to-many.
6. **Column renaming** — standardized field names for clarity (e.g., consistent casing, removing abbreviations) before exposing them to the report layer.

---

## 5. Key Measures (DAX)

The following calculated measures power the KPI cards and charts:

- `Total Sales` = SUM(Fact Sales[Sales])
- `Total Profit` = SUM(Fact Sales[Profit])
- `Total Orders` = DISTINCTCOUNT(Fact Sales[OrderID])
- `Total Customers` = DISTINCTCOUNT(Fact Sales[CustomerID])
- `Total Products` = DISTINCTCOUNT(Fact Sales[ProductID])
- `Current Year Sales` / `Previous Year Sales` — time-intelligence measures using `Dim Calender[Year]` to power the YoY line chart
- `Current Month Profit` / `Previous Month Profit` — time-intelligence measures for the MoM profit comparison
- `Total Sales(%)` / `Total Profit(%)` — each category's share of overall sales/profit
- `Online Sales` — Sales filtered to `Dim Store[Channel] = "Online"`, used in the quarterly trend chart

---

## 6. Dashboard Design

### Page 1 — Overview
Purpose: a snapshot of where sales and profit come from right now.

| Visual | Fields |
|---|---|
| KPI cards | Total Sales, Total Profit, Total Orders, Total Customers, Total Products |
| Total Sales by Region | Region (axis) × Sales (value) |
| Total Profit by Region | Region (legend) × Profit (value) |
| Total Sales by Brand and City | Brand (axis), City (legend), Sales (value) |
| Category table | Category, Total Profit, Total Sales, Sum of Quantity |
| Sum of Sales by Channel | Channel (legend) × Sales (value) |
| Total Sales by Product ID | ProductID (axis) × Sales (value) |

### Page 2 — Time Series Analysis
Purpose: how performance is trending over time.

| Visual | Fields |
|---|---|
| KPI cards | Same five headline metrics, repeated for context |
| Sales: Current Year vs Previous Year | Year (axis), Current/Previous Year Sales (values) |
| Profit: Current Month vs Previous Month | Month (axis), Current/Previous Month Profit (values) |
| Total Sales(%) and Total Profit(%) by Category | Category (axis), two % measures |
| Online Sales by Quarter | Quarter (axis) × Online Sales (value) |

### Slicers (both pages)
Region, City, Category, Month, Year — kept identical on both pages so a filter selection carries through the whole report.

### How the Slicers Were Added — Step by Step
1. Click an empty area of the canvas.
2. In the **Visualizations** pane, select the **Slicer** icon.
3. Drag the relevant field into the **Field** well (e.g., `Dim Customer.Region` for the Region slicer).
4. Resize and position the slicer in the right-hand filter rail.
5. Repeat for City, Category, Month, and Year.
6. Format each slicer under the **Format** pane (List/Dropdown style, single vs multi-select).
7. Recreate (or copy/paste) the same five slicers on Page 2, and enable **View → Sync Slicers** so a selection made on one page automatically applies to the other.

---

## 7. Final Result

The finished dashboard is a two-page report:

- **Overview** — a KPI + composition view answering "where is performance coming from today" (region, brand, city, channel, category breakdown).
- **Time Series Analysis** — a trend view answering "how is performance moving" (year-over-year, month-over-month, quarterly).

Both pages stay synchronized through shared slicers, so a user can, for example, filter to Electronics in the South region and immediately see both the composition (Page 1) and the trend (Page 2) update together.

---

## 8. Business Questions & Insights

**Q1: Which product category drives the most profit?**
Clothing leads (₹23.77L profit), closely followed by Electronics (₹23.62L) and Home (₹23.45L). Grocery (₹22.73L) and Beauty (₹18.72L) trail behind — Beauty is the clear underperformer on profit despite reasonable sales volume.

**Q2: How do sales trend across years and quarters?**
The Current Year vs Previous Year line chart shows sales are cyclical rather than steadily climbing — performance rises and falls year to year instead of following a flat upward trend, which is worth investigating against seasonality or promotions.

**Q3: Which channel is used most, and how does it affect profit?**
Store and Online are nearly balanced — Store holds a slight edge at 50.6% of sales versus Online's 49.4%. Channel choice does not appear to be a major profit lever on its own; the split is close enough that category and region matter more.

**Q4: What is the average discount per category, and how does it impact profit?**
Discounts in the dataset range from 0% to 25% in 5-point increments. The current dashboard tracks profit and profit % by category but does not yet break discount out as its own visual — flagged below as a natural next addition.

**Q5: Which cities/states generate the highest sales?**
The Brand-by-City breakdown on Page 1 shows sales concentrated around a few leading brand/city combinations, with Brand D standing out across multiple cities. A dedicated "Total Sales by City" ranking visual would make this trend even easier to read at a glance.

---

## 9. Outcomes

- Delivered a two-page Power BI dashboard on a clean star-schema model covering 5,000 transactions across 4 years, 5 cities, and 5 categories.
- Reduced ad-hoc analysis effort by centralizing 5 KPIs and 10 visuals behind 5 shared, synced slicers.
- Identified Clothing and Electronics as the strongest profit contributors and Beauty as the weakest, directly actionable for merchandising or promotion decisions.
- Surfaced a near-even Online/Store split, indicating channel strategy should be driven by category and region rather than channel alone.

---

## 10. Future Improvements

- Add a dedicated **Average Discount % by Category** visual with its profit impact.
- Add a **Total Sales by City/State** ranked bar chart for a direct geographic read.
- Add a **Top 10 Products by Sales** visual (planned in the original design, not yet in the current build).
- Incorporate **Return Rate** analysis using the existing `ReturnFlag` field.
- Publish to Power BI Service and add a scheduled refresh against a live data source.

---

## 11. Tools Used

- **Power BI Desktop** — modeling, DAX, report design
- **Power Query** — data cleaning and transformation
- **Excel** — original dataset format
