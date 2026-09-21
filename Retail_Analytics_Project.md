# Retail Analytics Project Using Databricks

## Project Summary

This project demonstrates an end-to-end Data Engineering workflow using Databricks Medallion Architecture.

We use the following source tables from the Databricks Marketplace dataset:

```text
databricks_simulated_retail_customer_data.v01.customers
databricks_simulated_retail_customer_data.v01.sales
databricks_simulated_retail_customer_data.v01.sales_orders
```

The project follows:

```text
Source Data
    ↓
Bronze Layer
    ↓
Silver Layer
    ↓
Gold Layer
    ↓
Dashboard
```

---

# Project Goal

Build a retail analytics solution that:

- Ingests data from source tables
- Stores raw data in Bronze
- Cleans and transforms data in Silver
- Creates business metrics in Gold
- Builds dashboards for reporting

---

# Project Architecture

```text
databricks_simulated_retail_customer_data
                    │
                    ▼
              Bronze Layer
        ┌───────────┼───────────┐
        │           │           │
   customers      sales    sales_orders
        │           │           │
        ▼           ▼           ▼
              Silver Layer
        ┌───────────┼───────────┐
        │           │           │
   customers      sales    sales_orders
        │           │           │
        └───────┬───┴───────────┘
                ▼
            Gold Layer
        ┌──────────────┐
        │Customer KPI  │
        │Product KPI   │
        │Daily Sales   │
        └───────┬──────┘
                ▼
           Dashboard
```

---

# Step 1: Create Catalog

```sql
CREATE CATALOG IF NOT EXISTS retail_project;
```

Verify:

```sql
SHOW CATALOGS;
```

---

# Step 2: Create Bronze Schema

```sql
CREATE SCHEMA IF NOT EXISTS retail_project.bronze;
```

Verify:

```sql
SHOW SCHEMAS IN retail_project;
```

---

# Step 3: Create Silver Schema

```sql
CREATE SCHEMA IF NOT EXISTS retail_project.silver;
```

---

# Step 4: Create Gold Schema

```sql
CREATE SCHEMA IF NOT EXISTS retail_project.gold;
```

---

# Step 5: Create Bronze Customers Table

Purpose:

Store raw customer data without modifications.

```sql
CREATE OR REPLACE TABLE retail_project.bronze.customers AS
SELECT *
FROM databricks_simulated_retail_customer_data.v01.customers;
```

Verify:

```sql
SELECT *
FROM retail_project.bronze.customers
LIMIT 10;
```

---

# Step 6: Create Bronze Sales Table

Purpose:

Store raw sales transactions.

```sql
CREATE OR REPLACE TABLE retail_project.bronze.sales AS
SELECT *
FROM databricks_simulated_retail_customer_data.v01.sales;
```

Verify:

```sql
SELECT *
FROM retail_project.bronze.sales
LIMIT 10;
```

---

# Step 7: Create Bronze Sales Orders Table

Purpose:

Store raw order information.

```sql
CREATE OR REPLACE TABLE retail_project.bronze.sales_orders AS
SELECT *
FROM databricks_simulated_retail_customer_data.v01.sales_orders;
```

Verify:

```sql
SELECT *
FROM retail_project.bronze.sales_orders
LIMIT 10;
```

---

# Bronze Layer Output

Tables Created:

```text
retail_project.bronze.customers
retail_project.bronze.sales
retail_project.bronze.sales_orders
```

No transformations are applied.

---

# Step 8: Create Silver Customers Table

Purpose:

Clean customer data.

Columns Cleaned:

```text
customer_name
city
state
postcode
number
```

Notebook: `02_silver_customers`

```python
from pyspark.sql.functions import *

silver_customers = (
    spark.table("retail_project.bronze.customers")

    .withColumn(
        "customer_name",
        trim(
            regexp_replace(
                upper(col("customer_name")),
                r"\s+",
                " "
            )
        )
    )

    .withColumn(
        "city",
        upper(trim(col("city")))
    )

    .withColumn(
        "state",
        upper(trim(col("state")))
    )

    .withColumn(
        "postcode",
        regexp_replace(
            col("postcode"),
            r"\.0$",
            ""
        )
    )

    .withColumn(
        "number",
        regexp_replace(
            col("number"),
            r"\.0$",
            ""
        )
    )
)

silver_customers.write \
.mode("overwrite") \
.saveAsTable(
    "retail_project.silver.customers"
)
```

Verify:

```sql
SELECT *
FROM retail_project.silver.customers
LIMIT 10;
```

---

# Step 9: Create Silver Sales Orders Table

Purpose:

Clean order data.

Columns Cleaned:

```text
customer_name
product_name
product_category
order_date
total_price
```

Notebook: `03_silver_sales_orders`

```python
from pyspark.sql.functions import *

silver_sales_orders = (
    spark.table("retail_project.bronze.sales_orders")

    .withColumn(
        "customer_name",
        trim(
            regexp_replace(
                upper(col("customer_name")),
                r"\s+",
                " "
            )
        )
    )

    .withColumn(
        "product_name",
        trim(col("product_name"))
    )

    .withColumn(
        "product_category",
        upper(trim(col("product_category")))
    )

    .withColumn(
        "order_date",
        to_date(col("order_date"))
    )

    .withColumn(
        "total_price",
        col("total_price").cast("double")
    )
)

silver_sales_orders.write \
.mode("overwrite") \
.saveAsTable(
    "retail_project.silver.sales_orders"
)
```

Verify:

```sql
SELECT *
FROM retail_project.silver.sales_orders
LIMIT 10;
```

---

# Step 10: Create Silver Sales Table

Purpose:

Remove duplicate records.

Notebook: `04_silver_sales`

```python
silver_sales = (
    spark.table("retail_project.bronze.sales")
    .dropDuplicates()
)

silver_sales.write \
.mode("overwrite") \
.saveAsTable(
    "retail_project.silver.sales"
)
```

Verify:

```sql
SELECT *
FROM retail_project.silver.sales
LIMIT 10;
```

---

# Silver Layer Output

Tables Created:

```text
retail_project.silver.customers
retail_project.silver.sales
retail_project.silver.sales_orders
```

Data quality improvements:

```text
✓ Trim spaces
✓ Standardize text
✓ Convert datatypes
✓ Remove duplicates
✓ Clean postcode
```

---

# Step 11: Create Gold Customer Revenue Table

Business Question:

```text
Which customers generate the highest revenue?
```

Notebook: `05_gold_customer_revenue`

```sql
CREATE OR REPLACE TABLE retail_project.gold.customer_revenue AS

SELECT
    customer_id,
    customer_name,
    SUM(total_price) AS revenue

FROM retail_project.silver.sales_orders

GROUP BY
    customer_id,
    customer_name;
```

Verify:

```sql
SELECT *
FROM retail_project.gold.customer_revenue
ORDER BY revenue DESC;
```

---

# Step 12: Create Gold Product Revenue Table

Business Question:

```text
Which products generate the highest revenue?
```

Notebook: `06_gold_product_revenue`

```sql
CREATE OR REPLACE TABLE retail_project.gold.product_revenue AS

SELECT
    product_name,
    SUM(total_price) AS revenue

FROM retail_project.silver.sales_orders

GROUP BY product_name

ORDER BY revenue DESC;
```

Verify:

```sql
SELECT *
FROM retail_project.gold.product_revenue;
```

---

# Step 13: Create Gold Daily Sales Table

Business Question:

```text
How much revenue is generated daily?
```

Notebook: `07_gold_daily_sales`

```sql
CREATE OR REPLACE TABLE retail_project.gold.daily_sales AS

SELECT
    order_date,
    SUM(total_price) AS daily_revenue

FROM retail_project.silver.sales_orders

GROUP BY order_date

ORDER BY order_date;
```

Verify:

```sql
SELECT *
FROM retail_project.gold.daily_sales;
```

---

# Gold Layer Output

Tables Created:

```text
retail_project.gold.customer_revenue
retail_project.gold.product_revenue
retail_project.gold.daily_sales
```

---

# Step 14: Create Dashboard

Dashboard Name:

```text
Retail Analytics Dashboard
```

---

## KPI 1: Total Revenue

```sql
SELECT
SUM(total_price) AS total_revenue
FROM retail_project.silver.sales_orders;
```

Visualization:

```text
KPI Card
```

---

## KPI 2: Total Customers

```sql
SELECT
COUNT(DISTINCT customer_id) AS total_customers
FROM retail_project.silver.customers;
```

Visualization:

```text
KPI Card
```

---

## KPI 3: Total Orders

```sql
SELECT
COUNT(*) AS total_orders
FROM retail_project.silver.sales_orders;
```

Visualization:

```text
KPI Card
```

---

## Chart 1: Revenue By Customer

Source Table:

```text
retail_project.gold.customer_revenue
```

Visualization:

```text
Bar Chart
```

X-Axis:

```text
customer_name
```

Y-Axis:

```text
revenue
```

---

## Chart 2: Revenue By Product

Source Table:

```text
retail_project.gold.product_revenue
```

Visualization:

```text
Horizontal Bar Chart
```

X-Axis:

```text
revenue
```

Y-Axis:
