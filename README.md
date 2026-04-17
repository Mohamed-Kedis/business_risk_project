# 🏦 Business Risk Sales — Data Engineering Project

> **From Flat Tables → Star Schema → Galaxy Schema**  
> A complete Data Engineering pipeline built on real Egyptian retail sales data.

---

## 📊 Project Overview

| Item | Value |
|---|---|
| **Data Range** | Jan 2023 – Feb 2024 (14 months) |
| **Total Orders** | 20,000 |
| **Total Revenue** | EGP 830.3 Million |
| **Net Financed** | EGP 407.3 Million |
| **NPL Rate** | 2.9% (DPD > 90) |
| **Cities** | Cairo, Giza, Alexandria, Mansoura |
| **Currency** | EGP only |
| **Channel** | Retail only |
| **Interest Rate** | 15% fixed |

---

## 🗂️ Project Structure

```
business_risk_project/
│
├── data/
│   ├── raw/                    # Original source files (CSV)
│   │   ├── Fact_Sales.csv
│   │   ├── Dim_Customers.csv
│   │   ├── Dim_Products.csv
│   │   ├── Dim_Branches.csv
│   │   └── Dim_Agents.csv
│   │
│   └── processed/              # Transformed DWH tables (auto-generated)
│       ├── Fact_Sales_Star.csv
│       ├── Fact_Collections.csv
│       ├── Fact_Risk_Snapshot.csv
│       ├── Dim_Date.csv
│       ├── Dim_Payment.csv
│       └── Dim_Risk.csv
│
├── scripts/
│   ├── 01_data_modeling.py     # ⭐ Main: Flat → Star → Galaxy
│   ├── 02_etl_pipeline.py      # ETL: Extract → Validate → Transform → Load
│   └── 03_analytics.py         # KPIs, aggregations, reports
│
├── sql/
│   └── star_galaxy_schema.sql  # DDL + Analytics queries
│
├── docs/
│   └── data_modeling_concept.md
│
└── README.md
```

---

## 🧠 Data Modeling Concept

### The Problem with Flat Tables

```
❌ FLAT TABLE — Everything in one place:
sale_id | customer_name | customer_job | product | branch | city | quantity | price
```

**Why it fails:**
- **Data Duplication** — Customer name stored once per order (1,000+ times for frequent buyers)
- **Update Anomaly** — Change 1 name = update thousands of rows
- **Slow Queries** — Full table scans on every aggregation
- **No Single Source of Truth**

---

### Step 1 — Think in Facts & Dimensions

| Concept | Definition | Examples |
|---|---|---|
| **Facts** | Measurable events | Revenue, Net_Financed, DPD, Installment |
| **Dimensions** | Context (Who/What/Where/When) | Customer, Product, Branch, Date |

---

### Step 2 — Star Schema ⭐

```
         Dim_Customer        Dim_Product        Dim_Branch
              │                   │                  │
              └───────────────────┼──────────────────┘
                                  │
                         ┌────────▼────────┐
                         │   FACT_SALES    │ ← Center
                         │  Order_ID (PK)  │
                         │  Customer_ID FK │
                         │  Product_ID  FK │
                         │  Branch_ID   FK │
                         │  Agent_ID    FK │
                         │  Date_Key    FK │
                         │  ─────────────  │
                         │  Total_Value    │ ← Measures
                         │  Net_Financed   │
                         │  Remaining_Bal  │
                         │  Days_Past_Due  │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼──────────────────┐
              │                   │                  │
          Dim_Agent           Dim_Date          Dim_Payment
```

**Benefits:**
- ✅ Simple, fast joins
- ✅ BI tools work natively
- ✅ Aggregations 10x faster than flat

---

### Step 3 — Galaxy Schema 🌌

When the business grows and needs **multiple business processes**:

```
Dim_Customer ─┬─ Dim_Branch ─┬─ Dim_Date ─┬─ Dim_Agent
              │              │             │
         FACT_SALES      FACT_COLLECTIONS  FACT_RISK_SNAPSHOT
         (Revenue)       (Collections)     (Monthly Risk)
         20,000 rows     11,900 rows        ~1,200 rows
```

**Power of Galaxy:**
```sql
-- "Revenue vs Collections vs NPL per branch per month"
-- Joins across 3 facts via shared Dim_Branch + Dim_Date
SELECT b.City_Location, d.Period_Label,
       SUM(fs.Total_Value)         AS Revenue,
       SUM(fc.Amount_Collected)    AS Collected,
       SUM(rs.NPL_Rate_Pct)        AS NPL_Rate
FROM Fact_Sales fs
JOIN Fact_Collections fc   ON fs.Order_ID  = fc.Order_ID
JOIN Fact_Risk_Snapshot rs ON fs.Branch_ID = rs.Branch_ID
JOIN Dim_Branches b        ON fs.Branch_ID = b.Branch_ID
JOIN Dim_Date d            ON fs.Date_Key  = d.Date_Key
GROUP BY b.City_Location, d.Period_Label;
```

---

## 📦 Data Schema

### Fact Tables

| Table | Grain | Rows | Description |
|---|---|---|---|
| `Fact_Sales` | One row per order | 20,000 | Sales & revenue |
| `Fact_Collections` | One row per installment order | ~11,900 | Payment collections |
| `Fact_Risk_Snapshot` | Branch × Month × DPD | ~1,200 | Risk monitoring |

### Dimension Tables

| Table | Rows | Key Attributes |
|---|---|---|
| `Dim_Customers` | 19,902 | Name, Job, Age, Credit_Rating, Collateral |
| `Dim_Products` | 16 | Model, Brand, Category |
| `Dim_Branches` | 9 | Branch_Name, City_Location |
| `Dim_Agents` | 6 | Sales_Agent |
| `Dim_Date` | ~396 | Year, Quarter, Month, Week, Is_Weekend |
| `Dim_Payment` | ~6 | Method, Platform, Channel |
| `Dim_Risk` | 5 | DPD value, Classification, Provisioning% |

---

## ⚡ Key KPIs

| KPI | Value |
|---|---|
| Total Revenue | **EGP 830.3M** |
| Net Financed | **EGP 407.3M** |
| Remaining Balance | **EGP 198.1M** |
| Cash Orders | **40.5%** |
| NPL Rate | **2.9%** |
| Avg Order Value | **EGP 41.5K** |

---

## 🚀 How to Run

### 1. Clone the repo

```bash
git clone https://github.com/MostafaAbdelkaderSelim/project-data-engineer-2.git
cd project-data-engineer-2
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the data modeling pipeline

```bash
# Step 1: Build Star + Galaxy Schema
python scripts/01_data_modeling.py

# Step 2: Run ETL pipeline
python scripts/02_etl_pipeline.py

# Step 3: Generate analytics
python scripts/03_analytics.py
```

### 4. Apply SQL schema (optional — needs PostgreSQL)

```bash
psql -U postgres -d your_db -f sql/star_galaxy_schema.sql
```

---

## 🔧 Technologies Used

| Tool | Purpose |
|---|---|
| **Python 3.10+** | ETL pipeline, data modeling |
| **CSV / Pandas** | Data processing |
| **PostgreSQL** | Data warehouse target |
| **SQL** | DDL, analytics queries, window functions |
| **Git / GitHub** | Version control |

---

## 📈 Topics Covered

- ✅ **ETL Pipeline** — Extract → Validate → Transform → Load
- ✅ **Star Schema** — 1 Fact + 7 Dimensions
- ✅ **Galaxy Schema** — 3 Facts sharing conformed dimensions
- ✅ **Data Quality** — DQ Score 98.1%, anomaly detection
- ✅ **SCD Type 2** — Dim_Customers & Dim_Branches
- ✅ **Incremental Load** — Watermark-based upsert
- ✅ **SQL Analytics** — Window functions, materialized views, partitioning
- ✅ **Risk Modeling** — DPD classification, NPL detection, provisioning %

---

## 👤 Author

**Mostafa Abdelkader Selim**  
GitHub: [@MostafaAbdelkaderSelim](https://github.com/MostafaAbdelkaderSelim)

---

## 📄 License

MIT License — free to use, fork, and build on.
