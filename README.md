# Electro Nova – Excel Sales Analytics Dashboard

An end-to-end data analysis project built entirely in **Microsoft Excel**. Raw CSV files for customers, products and monthly orders are cleaned with **Power Query**, modelled as a **star schema in Power Pivot**, summarised with **PivotTables**, and presented in an interactive **dashboard** with slicers and a timeline.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tools & Skills](#tools--skills)
3. [Data Sources](#data-sources)
4. [Stage 1 – Data Cleaning (Power Query)](#stage-1--data-cleaning-power-query)
5. [Stage 2 – Data Modelling (Power Pivot)](#stage-2--data-modelling-power-pivot)
6. [Stage 3 – Analysis (PivotTables)](#stage-3--analysis-pivottables)
7. [Stage 4 – Dashboard](#stage-4--dashboard)
8. [Key Insights](#key-insights)
9. [Repository Structure](#repository-structure)
10. [How to Use](#how-to-use)

---

## Project Overview

**Electro Nova** is a retail dataset covering customers across five Egyptian regions, products in five categories, and 20,000 orders placed during 2024. The goal was to turn messy, multi-file raw data into a clean, reliable model and an easy-to-read dashboard that answers questions such as:

- How much did we sell in total, and how many orders, units and customers does that represent?
- Which regions and product categories drive sales?
- How does quantity sold change month by month?
- Who are the top customers and the best-selling products?

## Tools & Skills

| Area | Tool / Feature |
|---|---|
| Data cleaning & transformation | Excel Power Query (M language) |
| Data modelling | Power Pivot, star schema, relationships, Calendar table |
| Calculations | DAX measures |
| Analysis | PivotTables, PivotCharts |
| Visualisation | Excel dashboard, slicers, timeline |

## Data Sources

| File | Columns | Description |
|---|---|---|
| `Customers.csv` | CustomerID, CustomerName, Region | Customer master data (300 customers) |
| `Products.csv` | ProductID, ProductName, Category, Price | Product catalogue |
| `Orders/` (folder of monthly CSVs) | OrderID, CustomerID, ProductID, OrderDate, Quantity | Transactions, one file per month (20,000 orders in total) |

---

## Stage 1 – Data Cleaning (Power Query)

The full M code for each query is in [`power-query/applied_steps.md`](power-query/applied_steps.md).

### 1. Customers query

| Step | Purpose |
|---|---|
| Load CSV (`Csv.Document`) | Import `Customers.csv` with 3 columns |
| Promote headers | Use the first row as column names |
| Change type | `CustomerID` → whole number; `CustomerName`, `Region` → text |
| Capitalize each word + trim (`CustomerName`) | Standardise names (e.g. `mohamed  mahmoud` → `Mohamed Mahmoud`) |
| Replace value in `Region` (space → empty) | Remove stray spaces inside region values |
| Capitalize each word + trim (`Region`) | Consistent region labels (Alexandria, Aswan, Cairo, Giza, Mansoura) |

### 2. Products query

| Step | Purpose |
|---|---|
| Load CSV, promote headers | Import `Products.csv` with 4 columns |
| Change type | `ProductID` → whole number; others → text |
| Trim + capitalize (`ProductName`, `Category`) | Remove extra spaces and unify casing |
| Replace value in `Price`: `Two Hundred` → `200` | Fix a price typed as words so it can be converted to a number |
| Change type (`Price`) → Currency | Make price usable in calculations |

### 3. Orders query (combine files from a folder)

| Step | Purpose |
|---|---|
| `Folder.Files` on the `Orders` folder | Read every monthly file at once |
| Filter hidden files | Avoid system or hidden files |
| Invoke custom function (`Transform File`) + expand | Combine all monthly files into one table |
| Keep `Source.Name` | Preserve the file name, which contains the month |
| Change type | IDs and `Quantity` → whole numbers; `OrderDate` → text (formats are mixed) |
| Extract text between delimiters (`_` and `.`) | Pull the month name out of the file name (e.g. `Orders_January.csv` → `January`) |
| Rename `Source.Name` → `month` | Clear column name |
| Replace `/` with `-` in `OrderDate` | Unify date separators |
| Split `OrderDate` by `-` into 3 parts | Separate day, month and year, which appear in different orders across files |
| Conditional column `year` | The part greater than 31 must be the year |
| Custom column `day` | The part ≤ 31 that is not the month is the day |
| Remove the 3 split columns | Clean up helper columns |
| Merge `month` + `day` + `year` with `-` | Build a standard date string |
| Change type → Date, rename → `new date` | Final, reliable date column |

**Main challenge solved:** order dates came in inconsistent formats across the monthly files. They were rebuilt using the month from the file name together with logic that identifies the year and day, which produced one valid `new date` column.

---

## Stage 2 – Data Modelling (Power Pivot)

<img width="1536" height="814" alt="power query" src="https://github.com/user-attachments/assets/0530a4e1-a3ff-449f-a922-966c08e65b10" />


```
 Products (1) ───┐
                 ├──< Orders (*)  (fact table)
 Customers (1) ──┤
                 │
 Calendar (1) ───┘
```

| Table | Role | Key |
|---|---|---|
| **Orders** | Fact table | OrderID, CustomerID, ProductID, `new date`, Quantity |
| **Customers** | Dimension | CustomerID (1 → many) |
| **Products** | Dimension | ProductID (1 → many) |
| **Calendar** | Date dimension | Date ↔ Orders[`new date`] (1 → many) |

**Calendar table columns:** Date, Year, Month Number, Month, MMM-YYYY, Day Of Week Number, Day Of Week, and a **Date Hierarchy** (Year → Month → Date).

**Measures** created in the Orders table: total customers, total orders, total quantity, and total sales. Example DAX (adjust to match your own measures):

```DAX
total quantity := SUM ( Orders[Quantity] )
total order    := COUNTROWS ( Orders )
total customer := DISTINCTCOUNT ( Orders[CustomerID] )
total sales    := SUMX ( Orders, Orders[Quantity] * RELATED ( Products[Price] ) )
```

> Screenshot: `images/data_model.png`

---

## Stage 3 – Analysis (PivotTables)

<img width="1302" height="685" alt="pivot table" src="https://github.com/user-attachments/assets/c5abb3d3-b881-4172-8a42-10dd76cc7333" />


| PivotTable | Measure | Dimension |
|---|---|---|
| KPI cards | Total sales, total orders, total quantity, total customers | – |
| Sales by region | % of total sales | Region |
| Sales by category | % of total sales | Category |
| Quantity trend | Sum of Quantity | Year → Month |
| Top 10 customers | Total orders (Top 10 filter) | CustomerName |
| Top 5 best-selling products | Sum of Quantity (Top 5 filter) | ProductName |
| Customers by region | Count of CustomerID | Region |

> Screenshot: `images/pivot_tables.png`

---

## Stage 4 – Dashboard

<img width="1649" height="624" alt="Electro nova Dashboard" src="https://github.com/user-attachments/assets/4b4d3397-90f8-48ce-9b7b-91564be25a58" />


**KPI cards**

| Total Sales | Quantity | Orders | Customers |
|---|---|---|---|
| 101,146,137 | 109,108 | 20,000 | 300 |

**Visuals**
- Sales by Region (doughnut chart)
- Quantity trend by month (line chart)
- Top 10 customers (column chart)
- Sales by Category (pie chart)
- Top 5 best-selling products (bar chart)
- Customers by Region (column chart)

**Interactivity:** slicers for **Region** and **Category**, and a **Date timeline** (by month), all connected to every PivotTable so the whole dashboard updates together.

> Screenshot: `images/dashboard.png`

---

## Key Insights

- **Regional sales are balanced:** Mansoura leads with 21%, followed by Aswan, Cairo and Giza at 20% each, and Alexandria at 18%.
- **Clothing is the top category** with 29% of sales, followed by Toys (25%), Electronics (19%), Books (17%) and Furniture (10%).
- **Monthly quantity is stable** at roughly 9,000 units per month, peaking in May (9,208) and dipping in June (8,956).
- **Top products by quantity:** Tablet (4,637), Table (4,620), Ball (4,524), Camera (4,504) and Jacket (4,493).
- **Top customers by orders:** Mohamed Khaled (494), Mohamed Rashid (446) and Sara Ibrahim (437) are among the leaders.
- **Customer base** is spread evenly across regions, with 56 to 62 customers each.

---

## Repository Structure

```
Electro-Nova-Excel-Sales-Dashboard/
│
├── README.md
├── Electro_Nova_Dashboard.xlsx        # Final workbook (model, pivots, dashboard)
├── data/
│   ├── Customers.csv
│   ├── Products.csv
│   └── Orders/                        # Monthly order files
├── power-query/
│   └── applied_steps.md               # M code for Customers, Products, Orders
└── images/
    ├── data_model.png                 # Star schema (Power Pivot diagram view)
    ├── pivot_tables.png               # PivotTables sheet
    └── dashboard.png                  # Final dashboard
```

## How to Use

1. Download or clone the repository.
2. Open `Electro_Nova_Dashboard.xlsx` in Excel (Microsoft 365 or Excel 2016+ with Power Pivot enabled).
3. To refresh from source, update the file paths in **Data → Queries & Connections** to point to your local `data/` folder, then choose **Refresh All**.
4. Use the slicers and timeline to explore the dashboard.

---

## Author

Built as a hands-on data analysis portfolio project covering the full workflow: **clean → model → analyse → visualise**.
