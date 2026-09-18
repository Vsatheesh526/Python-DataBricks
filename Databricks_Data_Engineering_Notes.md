# Databricks Data Engineering Complete Notes

---

# 1. Your Databricks Workspace

## What is a Databricks Workspace?

A Databricks Workspace is the environment where data engineers, analysts, and data scientists collaborate.

Think of it as your **Data Engineering Office**.

### Main Components

| Component | Purpose |
|------------|---------|
| Workspace | Stores notebooks and folders |
| Cluster | Executes code |
| Catalog | Organizes data assets |
| Notebook | Develop code and analysis |
| Jobs | Schedule workflows |
| Dashboards | Visualize data |

### Real Example

Imagine you are working for an e-commerce company.

- Workspace = Office Building
- Notebook = Your desk
- Cluster = Workers doing calculations
- Tables = Filing cabinets
- Dashboard = Management reports

---

# 2. Data Engineering Fundamentals

## What is Data Engineering?

Data Engineering is the process of:

1. Collecting data
2. Transforming data
3. Storing data
4. Making data available

### Example Flow

Customer Orders CSV
      ↓
Read into Spark
      ↓
Clean Data
      ↓
Store in Delta Table
      ↓
Used by Reports

### ETL

ETL = Extract + Transform + Load

Example:

Extract:
customers.csv

Transform:
Remove null values

Load:
Delta Table

---

# 3. Spark DataFrames Mastery

## What is a DataFrame?

A DataFrame is a table-like structure distributed across a cluster.

Example:

| emp_id | name | salary |
|---------|--------|--------|
|101|Ravi|50000|
|102|Anil|60000|

### Create DataFrame

```python
data = [
(101,"Ravi",50000),
(102,"Anil",60000)
]

df = spark.createDataFrame(
data,
["emp_id","name","salary"]
)

df.show()
```

### Select Columns

```python
df.select("name","salary")
```

### Filter

```python
df.filter(df.salary > 50000)
```

### Add Column

```python
from pyspark.sql.functions import col

df.withColumn(
"bonus",
col("salary")*0.10
)
```

---

# 4. SQL Query Fundamentals

## Why Spark SQL?

Most companies heavily use SQL.

### Select

```sql
SELECT * FROM employees;
```

### Filter

```sql
SELECT *
FROM employees
WHERE salary > 50000;
```

### Sort

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

### Aggregate

```sql
SELECT department,
AVG(salary)
FROM employees
GROUP BY department;
```

---

# 5. Data Transformation Techniques

Transformations modify data.

### Example Dataset

| emp_id | name | city |
|---------|--------|---------|
|101|Ravi|hyderabad|

### Convert to Upper Case

```python
from pyspark.sql.functions import upper

df.withColumn(
"city",
upper("city")
)
```

### Remove Duplicates

```python
df.dropDuplicates()
```

### Null Handling

```python
df.fillna("Unknown")
```

---

# 6. Custom Function Development (UDF)

## What is a UDF?

UDF = User Defined Function

Used when Spark lacks a built-in function.

### Example

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def grade(salary):

    if salary > 60000:
        return "High"

    return "Medium"

grade_udf = udf(
grade,
StringType()
)

df.withColumn(
"grade",
grade_udf("salary")
)
```

---

# 7. Data Integration & Analytics

## Joins

### Employee Table

| emp_id | dept_id |
|---------|----------|
|101|1|

### Department Table

| dept_id | dept_name |
|----------|-----------|
|1|IT|

### Inner Join

```python
employees.join(
departments,
"dept_id",
"inner"
)
```

### Left Join

```python
employees.join(
departments,
"dept_id",
"left"
)
```

### Aggregation

```python
df.groupBy("department")\
  .sum("salary")
```

---

# 8. Enterprise Data Ingestion

## What is Data Ingestion?

Loading data into Databricks.

### Supported Sources

- CSV
- JSON
- Parquet
- Delta
- SQL Database
- Azure Storage
- AWS S3

### Read CSV

```python
df = spark.read.format("csv")\
.option("header","true")\
.load("/Volumes/workspace/default/employees.csv")
```

---

# 9. No-Code Data Prep

Visual tools help clean data without coding.

Examples:

- Rename columns
- Filter rows
- Join tables
- Remove duplicates

Useful for business users.

---

# 10. Ingestion & ETL Pipelines

## Pipeline Flow

Raw Data
↓
Validation
↓
Transformation
↓
Delta Tables
↓
Reporting

Example:

```python
raw_df = spark.read.csv(path)

clean_df = raw_df.dropDuplicates()

clean_df.write.format("delta").save(path)
```

---

# 11. Delta Lake Architecture

## What is Delta Lake?

Delta Lake adds:

- ACID Transactions
- Versioning
- Time Travel
- Schema Enforcement

### Create Delta Table

```python
df.write.format("delta")\
.save("/delta/employees")
```

### Read Delta Table

```python
spark.read.format("delta")\
.load("/delta/employees")
```

---

# 12. Schema Management & Evolution

## Problem

File 1

```text
id,name
```

File 2

```text
id,name,email
```

New column appears.

### Solution

Schema Evolution

```python
df.write \
.option("mergeSchema","true") \
.format("delta") \
.mode("append") \
.save(path)
```

---

# 13. Performance Optimization

## Why Optimization?

Faster jobs = Lower Cost

### Techniques

### Cache

```python
df.cache()
```

### Repartition

```python
df.repartition(4)
```

### Broadcast Join

```python
from pyspark.sql.functions import broadcast

large.join(
broadcast(small),
"id"
)
```

---

# 14. Storage & File Optimization

## File Formats

### CSV

Pros:
- Human readable

Cons:
- Slow

### Parquet

Pros:
- Compressed
- Fast

### Delta

Pros:
- Fast
- ACID
- Time Travel

Recommended:
Delta > Parquet > CSV

---

# 15. Real-Time Streaming Processing

## Batch vs Streaming

Batch:
Process once daily.

Streaming:
Process continuously.

### Example

Bank Transactions

New Transaction
→ Process instantly
→ Fraud Detection

```python
stream_df = spark.readStream \
.format("delta") \
.load(path)
```

---

# 16. Data Quality Engineering

Data quality ensures trusted data.

### Checks

- No duplicate records
- No null IDs
- Valid dates
- Positive salary

### Example

```python
df.filter(df.emp_id.isNull())
```

---

# 17. Change Data Capture (CDC)

## What is CDC?

Captures only changed records.

Example:

Before:

|id|salary|
|--|--|
|101|50000|

After:

|id|salary|
|--|--|
|101|60000|

Only update is processed.

Benefits:

- Faster
- Less storage
- Less compute

---

# 18. Historical Data Management (SCD)

## Slowly Changing Dimensions

Maintains history.

### SCD Type 1

Overwrite old value.

Before:

|id|city|
|--|--|
|101|Hyderabad|

After:

|101|Bangalore|

Old value lost.

### SCD Type 2

Keep history.

|id|city|active|
|--|--|--|
|101|Hyderabad|N|
|101|Bangalore|Y|

Used in most enterprise systems.

---

# 19. Pipeline Testing & Validation

## Why Testing?

Prevent production failures.

### Verify Row Count

```python
assert df.count() > 0
```

### Column Check

```python
assert "emp_id" in df.columns
```

---

# 20. Observability & Debugging

## Monitoring

Monitor:

- Job failures
- Slow queries
- Missing data

### View Execution Plan

```python
df.explain()
```

### Display Data

```python
df.show()
```

---

# 21. Lakehouse Architecture Design

## Medallion Architecture

### Bronze Layer

Raw Data

Example

```text
employees.csv
```

### Silver Layer

Cleaned Data

Example

```text
Nulls removed
Duplicates removed
```

### Gold Layer

Business-ready Data

Example

```text
Department Salary Report
```

Flow:

Bronze → Silver → Gold

---

# 22. Production Orchestration

## Databricks Workflows

Used for scheduling jobs.

Example:

Job A:
Read Data

↓

Job B:
Transform Data

↓

Job C:
Load Data

↓

Dashboard Refresh

Benefits:

- Automation
- Monitoring
- Retry Mechanisms

---

# 23. Databricks Apps

Databricks Apps help create applications directly in Databricks.

Examples:

- Analytics Portal
- Chat Applications
- Data Quality Dashboard
- AI Applications

---

# 24. Data Governance & Security

## Unity Catalog

Central governance solution.

Controls:

- Users
- Roles
- Permissions
- Data Access

Example:

```sql
GRANT SELECT
ON TABLE employees
TO analyst_group;
```

Benefits:

- Security
- Compliance
- Auditing

---

# 25. Genie Spaces & Natural Language Analytics

Users ask questions in plain English.

Example:

Question:

```text
Show total sales for August.
```

Genie converts it into SQL automatically.

Useful for:

- Business Analysts
- Managers
- Executives

---

# 26. ML Pipeline Development

## Machine Learning Workflow

Data Collection
↓
Cleaning
↓
Feature Engineering
↓
Training
↓
Evaluation
↓
Deployment

Example:

Predict Employee Attrition.

Features:

- Salary
- Experience
- Department

Output:

- Stay
- Leave

---

# 27. AI Gateway, Serving & Lakebase

## AI Gateway

Controls AI model access.

Benefits:

- Security
- Monitoring
- Cost Control

## Model Serving

Deploy trained models as APIs.

Example:

```text
Input:
Employee Data

Output:
Attrition Risk
```

## Lakebase

Database layer for operational AI workloads.

---

# 28. Configuration Reference

Important Settings

```python
spark.conf.get("spark.app.name")
```

```python
spark.conf.set(
"spark.sql.shuffle.partitions",
200
)
```

---

# 29. BI Tool Integration

Databricks integrates with:

- Power BI
- Tableau
- Looker
- Excel

Example Flow

Delta Table
↓
Power BI
↓
Dashboard
↓
Management Reports

---

# Databricks Interview Quick Revision

## What is Spark?

Distributed data processing engine.

## What is DataFrame?

Distributed table structure.

## What is Delta Lake?

Storage layer with ACID transactions.

## What is Medallion Architecture?

Bronze → Silver → Gold.

## Difference Between Parquet and Delta?

Parquet:
Storage Format

Delta:
Parquet + ACID + Time Travel

## What is CDC?

Capture only changed records.

## What is Unity Catalog?

Centralized governance and security.

## What is Streaming?

Continuous real-time data processing.

## What is SCD Type 2?

Maintains history of records.

## Most Common Databricks Workflow

Read CSV
→ DataFrame
→ Transform
→ Delta Table
→ Silver Layer
→ Gold Layer
→ Dashboard
