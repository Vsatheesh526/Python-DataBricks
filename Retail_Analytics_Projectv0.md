# Retail Analytics Pipeline using Databricks

## Problem Statement

Build an end-to-end retail analytics pipeline on Databricks that ingests raw retail data (customers, sales, and sales orders) from the Marketplace dataset, cleans and transforms it through a Medallion Architecture (Bronze → Silver → Gold), and delivers business insights through an interactive dashboard.

The project covers:

- Workspace setup
- Unity Catalog governance
- Data ingestion
- Data transformation
- Pipeline orchestration
- Job scheduling
- Dashboard creation

---

# Dataset Overview

The Databricks Marketplace listing provides data through a Delta Sharing share:

```text
dbacademy_dataset_retail
```

## Schema: v01

### customers

Type:

```text
Table
```

Description:

```text
US customers who purchase finished goods
```

### sales

Type:

```text
Table
```

Description:

```text
Item-level sales transactions
```

### sales_orders

Type:

```text
Table
```

Description:

```text
Originating purchase orders
```

### source_files

Type:

```text
Volume
```

Description:

```text
Raw CSV files

customers.csv
sales.csv
sales_orders.csv
```

### retail-pipeline

Type:

```text
Volume
```

Description:

```text
Streaming JSON files

customers
orders
status
```

---

## Schema: v02

### subsidiary_daily_orders

Type:

```text
Volume
```

Description:

```text
7 Days of Daily Order Drops

CSV + JSON

3 Subsidiaries
```

### business_daily_events

Type:

```text
Volume
```

Description:

```text
Unified Business Event Stream
```

### customer_changes_daily

Type:

```text
Volume
```

Description:

```text
7 Days Customer CDC Events

Used for SCD Type 2 implementations
```

---

# Step 1: Install Marketplace Dataset

## Navigate to Marketplace Listing

```text
Simulated Retail Customer Data
```

### Actions

1. Open Databricks Marketplace
2. Search for:

```text
Simulated Retail Customer Data
```

3. Click:

```text
Install
```

4. Accept Terms and Conditions

5. Provide a Catalog Name

Example:

```text
retail_data
```

6. Click Install

### Output

A Unity Catalog catalog is created.

Example:

```text
retail_data
```

---

# Step 2: Explore Installed Data

Open:

```text
Catalog Explorer
```

Navigate:

```text
retail_data
    └── v01
```

Review:

```text
customers
sales
sales_orders
```

Check:

- Schema
- Columns
- Sample Data

---

## Explore Volumes

Navigate to:

```text
retail_data
    └── v01
```

### source_files

Contains:

```text
customers.csv
sales.csv
sales_orders.csv
```

### retail-pipeline

Contains:

```text
customers JSON
orders JSON
status JSON
```

---

# Step 3: Create Project Workspace

Go To:

```text
Workspace
```

Create Folder:

```text
retail_analytics_project
```

Create Subfolders

```text
retail_analytics_project
│
├── notebooks
├── dashboards
└── pipelines
```

Purpose:

```text
notebooks  → Development notebooks
dashboards → Dashboard assets
pipelines  → Pipeline configurations
```

---

# Step 4: Create Unity Catalog Structure

Create Catalog

```sql
CREATE CATALOG retail_project;
```

---

## Create Bronze Schema

```sql
CREATE SCHEMA retail_project.bronze;
```

Purpose:

```text
Store raw ingested data
```

---

## Create Silver Schema

```sql
CREATE SCHEMA retail_project.silver;
```

Purpose:

```text
Store cleaned and validated data
```

---

## Create Gold Schema

```sql
CREATE SCHEMA retail_project.gold;
```

Purpose:

```text
Store business-level aggregations
```

---

## Grant Permissions

Grant:

```text
USE
ALL PRIVILEGES
```

on:

```text
Catalog
Schemas
Tables
```

---

# Step 5: Bronze Layer Data Ingestion

Create Notebook

```text
01_bronze_ingestion
```

---

## Option A: Load from Installed Marketplace Tables

### Customers

```sql
CREATE OR REPLACE TABLE retail_project.bronze.customers AS
SELECT *
FROM retail_data.v01.customers;
```

---

### Sales

```sql
CREATE OR REPLACE TABLE retail_project.bronze.sales AS
SELECT *
FROM retail_data.v01.sales;
```

---

### Sales Orders

```sql
CREATE OR REPLACE TABLE retail_project.bronze.sales_orders AS
SELECT *
FROM retail_data.v01.sales_orders;
```

---

## Option B: Load from Raw CSV Files

### Customers CSV

```python
df_customers = spark.read.csv(
    "/Volumes/retail_data/v01/source_files/customers.csv",
    header=True,
    inferSchema=True
)

df_customers.write.saveAsTable(
    "retail_project.bronze.customers"
)
```

---

### Output

Bronze Tables:

```text
retail_project.bronze.customers

retail_project.bronze.sales

retail_project.bronze.sales_orders
```

---

# Step 6: Silver Layer Data Cleaning

Create Notebook:

```text
02_silver_cleaning
```

Purpose:

```text
Clean
Validate
Standardize
Deduplicate
```

---

## Customer Cleaning Example

```python
from pyspark.sql.functions import *

silver_customers = (
    spark.table("retail_project.bronze.customers")
    .withColumn(
        "customer_name",
        trim(upper(col("customer_name")))
    )
    .dropDuplicates(["customer_id"])
    .filter(col("customer_id").isNotNull())
)

silver_customers.write.mode("overwrite").saveAsTable(
    "retail_project.silver.customers"
)
```

---

## Cleaning Rules

### Customers

```text
Remove Null customer_id
Trim spaces
Standardize names
Remove duplicates
```

---

### Sales

```text
Remove duplicate transactions
Validate customer IDs
Clean numeric columns
```

---

### Sales Orders

```text
Trim values
Standardize dates
Convert datatypes
Remove duplicates
```

---

## Referential Integrity Validation

Verify:

```text
sales.customer_id
exists in
customers.customer_id
```

Example:

```sql
SELECT *
FROM retail_project.silver.sales s
LEFT ANTI JOIN retail_project.silver.customers c
ON s.customer_id = c.customer_id;
```

Output should be:

```text
0 records
```

---

# Step 7: Gold Layer Business Aggregations

Create Notebook

```text
03_gold_aggregations
```

---

## Customer Sales Summary

Business Question:

```text
How much revenue is generated by each customer?
```

```sql
CREATE OR REPLACE TABLE
retail_project.gold.customer_sales_summary AS

SELECT
    c.customer_id,
    c.customer_name,
    COUNT(s.sales_id) AS total_orders,
    SUM(s.amount) AS total_revenue
FROM retail_project.silver.customers c
LEFT JOIN retail_project.silver.sales s
ON c.customer_id = s.customer_id
GROUP BY
    c.customer_id,
    c.customer_name;
```

---

## Daily Sales Trends

Business Question:

```text
How does revenue change over time?
```

```sql
CREATE OR REPLACE TABLE
retail_project.gold.daily_sales_trends AS

SELECT
    DATE(order_date) AS sale_date,
    COUNT(*) AS order_count,
    SUM(amount) AS daily_revenue
FROM retail_project.silver.sales
GROUP BY DATE(order_date)
ORDER BY sale_date;
```

---

## Top Products

Business Question:

```text
Which products generate the highest revenue?
```

```sql
CREATE OR REPLACE TABLE
retail_project.gold.top_products AS

SELECT
    product_name,
    SUM(amount) AS total_revenue,
    COUNT(*) AS units_sold
FROM retail_project.silver.sales
GROUP BY product_name
ORDER BY total_revenue DESC;
```

---

# Step 8: Create Lakeflow Spark Declarative Pipeline

Navigate:

```text
Pipelines
→ Create Pipeline
```

Pipeline Name:

```text
retail_silver_pipeline
```

Configuration:

```text
Compute Type : Serverless
Notebook     : 02_silver_cleaning
```

Purpose:

```text
Automate Silver Layer Processing
```

Pipeline Responsibilities:

```text
Data Quality Checks
Materialized Views
Streaming Tables
Transformations
```

Run Pipeline

Verify:

```text
Silver tables populated successfully
```

---

# Step 9: Create Lakeflow Job

Navigate:

```text
Jobs
→ Create Job
```

Job Name:

```text
retail_daily_pipeline
```

---

## Task Flow

### Task 1

```text
Bronze Ingestion
```

Notebook:

```text
01_bronze_ingestion
```

---

### Task 2

```text
Silver Cleaning
```

Notebook:

```text
02_silver_cleaning
```

Depends On:

```text
Bronze Ingestion
```

---

### Task 3

```text
Gold Aggregations
```

Notebook:

```text
03_gold_aggregations
```

Depends On:

```text
Silver Cleaning
```

---

## Scheduling

Schedule:

```text
Daily
06:00 AM
```

Configure:

```text
Failure Alerts
Email Notifications
```

Run Job

Verify successful completion.

---

# Step 10: Create Dashboard

Navigate:

```text
Dashboards
→ Create Dashboard
```

Dashboard Name:

```text
Retail Sales Analytics
```

---

## Visualization 1

### Revenue by Customer

Source:

```text
gold.customer_sales_summary
```

Visualization:

```text
Bar Chart
```

---

## Visualization 2

### Daily Sales Trend

Source:

```text
gold.daily_sales_trends
```

Visualization:

```text
Line Chart
```

---

## Visualization 3

### Top Products

Source:

```text
gold.top_products
```

Visualization:

```text
Table
or
Horizontal Bar Chart
```

---

## KPI Cards

### Total Revenue

```sql
SELECT SUM(amount)
FROM retail_project.silver.sales;
```

---

### Total Customers

```sql
SELECT COUNT(DISTINCT customer_id)
FROM retail_project.silver.customers;
```

---

### Total Orders

```sql
SELECT COUNT(*)
FROM retail_project.silver.sales_orders;
```

---

## Dashboard Filters

Add Filters:

```text
Date Range

Customer Segment

Product Category

State

Region
```

---

# End-to-End Flow

```text
Marketplace Install
        │
        ▼
Catalog Explorer
        │
        ▼
Bronze Layer
(Raw Data)
        │
        ▼
Silver Layer
(Clean & Validate)
        │
        ▼
Gold Layer
(Business Aggregations)
        │
        ▼
Lakeflow Pipeline
        │
        ▼
Lakeflow Job
        │
        ▼
Dashboard
        │
        ▼
Business Insights
```

---

# Expected Outputs

## Bronze

```text
customers
sales
sales_orders
```

Raw copies of source tables.

---

## Silver

```text
customers
sales
sales_orders
```

Cleaned and validated data.

---

## Gold

```text
customer_sales_summary

daily_sales_trends

top_products
```

Business-ready reporting tables.

---

## Dashboard

```text
Total Revenue

Total Customers

Total Orders

Revenue by Customer

Daily Sales Trend

Top Products
```

---

# Final Project Deliverables

✅ Databricks Marketplace Dataset Installed

✅ Unity Catalog Structure Created

✅ Bronze Layer Implemented

✅ Silver Layer Implemented

✅ Gold Aggregation Layer Implemented

✅ Lakeflow Pipeline Created

✅ Lakeflow Job Scheduled

✅ Retail Analytics Dashboard Published

✅ End-to-End Retail Analytics Data Engineering Project Completed
