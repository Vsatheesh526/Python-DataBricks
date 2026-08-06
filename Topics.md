# Quick Notes: Topics to Learn for Python + Databricks + Snowflake

## 1. SQL (Highest Priority) ⭐⭐⭐⭐⭐

Learn:

- SELECT
- WHERE
- ORDER BY
- GROUP BY
- HAVING
- COUNT, SUM, AVG, MIN, MAX
- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- UNION
- CASE WHEN
- Subqueries
- CTE (WITH)
- ROW_NUMBER()
- RANK()
- DENSE_RANK()

---

## 2. Python ⭐⭐⭐⭐

Learn:

- Variables
- Data Types
- Lists
- Tuples
- Dictionaries
- Sets
- If / Else
- Loops
- Functions
- Exception Handling
- File Handling
- Modules & Packages
- OOP Basics (Class, Object)

### Pandas Basics

- read_csv()
- head()
- filter()
- sort_values()
- groupby()
- merge()
- dropna()
- fillna()

---

## 3. PySpark ⭐⭐⭐⭐⭐

Learn:

- Spark Session
- DataFrames
- Read CSV/JSON
- show()
- select()
- filter()
- withColumn()
- drop()
- distinct()
- groupBy()
- agg()
- join()
- orderBy()
- write()

Functions:

- col()
- when()
- lit()
- concat()
- current_date()
- current_timestamp()

---

## 4. Databricks ⭐⭐⭐⭐⭐

Learn:

- Workspace
- Notebooks
- Clusters
- Jobs
- DBFS
- dbutils
- Notebook Scheduling
- Delta Lake
- Delta Tables
- MERGE
- OPTIMIZE
- VACUUM

Architecture Understanding:

```text
Notebook
   ↓
Cluster
   ↓
PySpark Code
   ↓
Delta Table
