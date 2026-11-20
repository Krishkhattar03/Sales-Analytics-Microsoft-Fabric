# 📊 Sales Analytics: End-to-End Data Engineering on Microsoft Fabric

An end-to-end sales analytics solution built on **Microsoft Fabric**. Raw sales CSV data is ingested into a Lakehouse, refined through a **Medallion Architecture** (Bronze → Silver → Gold), loaded into a **star-schema Warehouse**, and served through a semantic model to an interactive **Power BI** dashboard using **DirectLake**.

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Screenshots](#-screenshots)
- [Data Model](#-data-model)
- [Pipeline Walkthrough](#-pipeline-walkthrough)
- [Dashboard Highlights](#-dashboard-highlights)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Key Learnings](#-key-learnings)
- [Future Improvements](#-future-improvements)
- [Author](#-author)

---

## 🔍 Overview

This project demonstrates a complete analytics workflow:

1. **Ingest** a raw sales CSV (via HTTP) into a Fabric Lakehouse using a Copy Data activity
2. **Transform** data through Bronze, Silver, and Gold layers using notebooks
3. **Model** the data into a star schema (one fact table and four dimensions) in a Fabric Warehouse
4. **Serve** it through a Power BI semantic model with measures, relationships, and RLS
5. **Visualize** revenue, orders, and regional performance in a Power BI report using DirectLake

The whole process is orchestrated by a single Fabric Data Pipeline: `pl_ingest_sales`.

---

## 🏗️ Architecture

![Architecture Diagram](assets/architecture.png)

### Data flow at a glance

| Stage | Component | What happens |
|-------|-----------|--------------|
| **Source** | Sales CSV (HTTP) | Raw file ingested by the pipeline's Copy Data activity |
| **Bronze** | `bronze_sales` (Delta, in `lh_sales`) | Raw, unmodified, append-only |
| **Silver** | `silver_sales` (Delta, in `lh_sales`) | Deduplicated, typed, and standardised via `nb_transform_bronze_to_silver` |
| **Dimensions** | `dim_product`, `dim_region` | Surrogate keys assigned via `nb_build_dimensions` |
| **Gold** | `gold_sales_summary` (in `lh_sales`) | Monthly revenue by region and category via `nb_build_gold` |
| **Warehouse** | `wh_sales` (star schema) | Fact and dimension tables loaded using Copy Data activities |
| **Semantic model** | `SM_sales_analytics` | Measures, relationships, and RLS, connected live to the Warehouse |
| **Report** | Power BI report (DirectLake) | Published in the `ws_sales_analytics` workspace |

---

## 🛠️ Tech Stack

- **Microsoft Fabric**: unified analytics platform
- **Fabric Lakehouse** (`lh_sales`): storage with Delta tables
- **Fabric Data Pipeline** (`pl_ingest_sales`): orchestration
- **Notebooks (PySpark)**: Bronze → Silver → Gold transformations
- **Dataflow Gen2** (`Dataflow_sales`): data preparation
- **Fabric Warehouse** (`wh_sales`): star-schema serving layer
- **Power BI**: semantic model (`SM_sales_analytics`) and report
- **DirectLake**: fast, import-like performance without data duplication
- **Delta Lake**: table format

---

## 📸 Screenshots

### Power BI Dashboard
![Sales Analytics Dashboard](assets/dashboard.png)

### Data Pipeline Orchestration
![Fabric Data Pipeline](assets/pipeline.png)

### Lakehouse: Bronze Layer Data Preview
![Lakehouse Bronze Data](assets/lakehouse.png)

---

## 🗄️ Data Model

### Star schema in `wh_sales`

```
                    dim_date
                       │
   dim_customer ───  fact_sales  ─── dim_product
                       │
                   dim_region
```

| Table | Type | Key Columns |
|-------|------|-------------|
| `fact_sales` | Fact | `order_date_key`, `customer_key`, `product_key`, `region_key`, `revenue`, `quantity` |
| `dim_date` | Dimension | date key, `month_name`, `year`, quarter |
| `dim_customer` | Dimension | `customer_key`, `region` |
| `dim_product` | Dimension | `product_key`, `category` |
| `dim_region` | Dimension | `region_key` |

### Lakehouse tables in `lh_sales`

| Table | Layer | Description |
|-------|-------|-------------|
| `bronze_sales` | Bronze | Raw ingested records, append-only |
| `silver_sales` | Silver | Deduplicated, typed, standardised data |
| `gold_sales_summary` | Gold | Monthly revenue by region and category |
| `dim_*`, `fact_sales` | Model | Star-schema tables copied to the Warehouse |

### Source schema (`bronze_sales`)

| Column | Description |
|--------|-------------|
| `OrderID` | Unique order identifier (e.g., ORD-01255) |
| `OrderDate` | Date of the order |
| `CustomerName` | Name of the customer |
| `Region` | North / South / East / West |
| `ProductCategory` | Electronics / Furniture / Office Supplies |
| `Revenue` | Order revenue |
| `Quantity` | Units sold |
| `Status` | Active / Returned / Cancelled |

---

## ⚙️ Pipeline Walkthrough

The pipeline `pl_ingest_sales` runs these activities in sequence:

1. **CopyRawsalesCSV** (Copy data): pulls the raw CSV from the HTTP source into the Lakehouse
2. **CopydatatoDeltatabel** (Notebook): lands the raw data in the `bronze_sales` Delta table
3. **TransformBronzeToSilver** (Notebook, `nb_transform_bronze_to_silver`): removes duplicates, fixes data types, standardises values into `silver_sales`
4. **BuildGoldLayer** (Notebook, `nb_build_gold`): builds `gold_sales_summary` (monthly revenue by region and category)
5. **TransformSilverToWarehouse** (Copy data): loads the fact and dimension tables into the `wh_sales` Warehouse

The dimension tables (`dim_product`, `dim_region`) are built with `nb_build_dimensions`, which assigns surrogate keys.

---

## 📈 Dashboard Highlights

The **Sales Analytics Overview** report includes:

**KPIs**
- 💰 Total Revenue: ₹13.6M (≈ $164.4K)
- 🧾 Total Orders: 1,268
- 📦 Total Quantity Sold: 10.043K
- 📊 Avg Revenue per Order: ₹10.9K
- 📊 Avg Quantity per Order: 8.0

**Visuals**
- Revenue and orders by region (North, South, East, West)
- Orders and quantity sold by product category (donut charts)
- Total revenue by category (Electronics leads at ₹9.0M)
- Top 10 customers by revenue
- Slicers for Date, Month, Quarter, and Year

---

## 🚀 Getting Started

### Prerequisites
- A Microsoft Fabric workspace (trial or paid capacity)
- Power BI Desktop or the Fabric web interface

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/<your-repo-name>.git
   cd <your-repo-name>
   ```

2. **Create a Fabric workspace** named `ws_sales_analytics`

3. **Create a Lakehouse** named `lh_sales`

4. **Create a Warehouse** named `wh_sales`

5. **Import the notebooks** from the `notebooks/` folder into your workspace:
   - `nb_transform_bronze_to_silver`
   - `nb_build_dimensions`
   - `nb_build_gold`

6. **Create the pipeline** `pl_ingest_sales` and add the activities listed in the [Pipeline Walkthrough](#-pipeline-walkthrough). Point the first Copy Data activity to your sales CSV source.

7. **Run the pipeline** to populate the Bronze, Silver, Gold, and Warehouse tables

8. **Create the semantic model** `SM_sales_analytics` on top of `wh_sales`:
   define relationships (fact → dimensions), add measures, and configure RLS if needed

9. **Build or open the Power BI report** and connect it to the semantic model (DirectLake)

---

## 📁 Project Structure

```
├── assets/
│   ├── architecture.png
│   ├── dashboard.png
│   ├── pipeline.png
│   └── lakehouse.png
├── data/
│   └── sales_raw.csv
├── notebooks/
│   ├── nb_transform_bronze_to_silver
│   ├── nb_build_dimensions
│   └── nb_build_gold
├── powerbi/
│   └── Sales_Analytics.pbix
└── README.md
```

> Update this tree to match your actual repository layout.

---

## 💡 Key Learnings

- Designing a Medallion Architecture (Bronze → Silver → Gold) in Microsoft Fabric
- Orchestrating multi-step workflows with Data Pipelines
- Building a star schema with surrogate keys in a Fabric Warehouse
- Creating a semantic model with measures, relationships, and RLS
- Using DirectLake for fast Power BI reporting

---

## 🔮 Future Improvements

- Add incremental data loading
- Add data quality checks and logging
- Schedule automatic pipeline refreshes
- Add forecasting and trend analysis visuals

---

## 👤 Author

**Your Name**
- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [Your Name](https://linkedin.com/in/your-profile)

---

⭐ If you found this project useful, consider giving it a star!
