# Databricks Free Edition Workspace Notes

- https://bricksnotes.com/lessons/workspace-essentials

## Lesson 0.5 - Getting Started

### Objective

This lesson provides a complete overview of the Databricks Free Edition interface and explains how different components work together in a data engineering workflow.

After completing this lesson, you should be able to:

- Understand the Databricks workspace layout
- Know where to write code
- Know where data is stored
- Understand how compute works
- Learn how data flows through Databricks
- Start working with notebooks, tables, and sample datasets

---

# Databricks Architecture Overview

Databricks consists of five main areas:

```text
Home
   ↓
Workspace
   ↓
Compute
   ↓
Catalog
   ↓
SQL Analytics
```

Each component serves a specific purpose.

---

# Sidebar Navigation

The left sidebar is the command center of Databricks.

Important sections:

```text
Home
Workspace
Recents
Catalog
Compute
Jobs & Pipelines
SQL Editor
Queries
Dashboards
Genie
Alerts
Query History
Playground
Experiments
Models
Serving
```

---

# Home

## Purpose

Home is the landing page when you log in.

### Features

- Recent notebooks
- Getting started guides
- Quick actions
- Recently opened assets

### Usage

Use Home to quickly access your recent work.

---

# Workspace

## Purpose

Workspace is where all development takes place.

### Used For

- Creating notebooks
- Writing code
- Organizing folders
- Managing files
- Collaboration

### Example Structure

```text
Workspace
│
├── Data Engineering
│   ├── Notebook1
│   ├── Notebook2
│
├── SQL Practice
│
└── Projects
```

### Important

Everything you create is stored inside Workspace.

---

# Recents

## Purpose

Provides quick access to recently opened items.

### Benefits

- Open notebooks quickly
- Access recent dashboards
- Reopen recent queries

---

# Catalog

## Purpose

Catalog is Databricks' data browser.

It stores:

- Catalogs
- Schemas
- Tables
- Volumes

---

## Namespace Structure

Databricks follows a three-level hierarchy:

```text
catalog.schema.table
```

### Example

```text
workspace.default.employees
```

| Level | Example | Description |
|---------|---------|---------|
| Catalog | workspace | Top-level container |
| Schema | default | Group of tables |
| Table | employees | Actual dataset |

---

## Accessing Tables

### SQL

```sql
SELECT *
FROM workspace.default.employees;
```

### PySpark

```python
df = spark.table("workspace.default.employees")
```

---

# Sample Datasets

Databricks Free Edition provides built-in datasets for practice.

---

## nyctaxi

Contains:

- Taxi trip information
- Pickup details
- Drop details
- Fare information

Used for:

- SQL practice
- Aggregations
- Analytics

---

## tpch

Contains:

- Customers
- Orders
- Products
- Line Items

Used for:

- SQL joins
- Reporting
- Analytics

---

## bakehouse

Contains:

- sales_customers
- sales_suppliers
- sales_franchises
- transactions

Used for:

- Business analytics
- Dashboard projects
- End-to-end scenarios

---

# Adding Data

Navigate to:

```text
Catalog
   ↓
+ Add
```

Available options:

### Add Data

Upload files such as:

- CSV
- JSON
- Parquet
- Excel

---

### Ingest via Partner

Load data using external integrations.

---

### Upload to Volume

Upload files directly into Unity Catalog volumes.

---

### Create Catalog

Create a new data container.

---

### Create Volume

Create managed file storage.

---

### Create Connection

Connect databases and external systems.

---

### Create Credential

Store secure credentials.

---

# Volumes

## What is a Volume?

A Volume is a storage location for files.

### Example

```text
/Volumes/workspace/default/book_data/
```

### Stored Files

```text
employees.csv
sales.csv
customers.json
```

### Full Path Example

```text
/Volumes/workspace/default/book_data/employees.csv
```

---

# Why Use Volumes?

Volumes replace traditional DBFS storage.

Advantages:

- Better governance
- Unity Catalog integration
- Easier access control
- Recommended modern approach

---

# Compute

## What is Compute?

Compute is the processing power that runs your code.

---

# SQL Warehouses

Used for:

- SQL queries
- Reporting
- Dashboards

Databricks Free Edition provides:

```text
Serverless Starter Warehouse
```

### Characteristics

- Starts automatically
- No manual setup
- Fully managed

### Example

```sql
SELECT *
FROM workspace.default.employees;
```

The Starter Warehouse automatically executes the query.

---

# All-Purpose Compute

Used for:

- Python
- PySpark
- SQL notebooks

### How It Works

```text
Run Notebook Cell
         ↓
Connecting...
         ↓
Serverless Compute Starts
         ↓
Code Executes
```

No cluster creation is required.

---

# Jobs & Pipelines

Used to automate data workflows.

---

## Ingestion Pipeline

Purpose:

Load data from:

- Files
- APIs
- Databases
- Applications

Into Databricks tables.

---

## ETL Pipeline

Purpose:

Transform data using:

- SQL
- Python
- PySpark

ETL:

```text
Extract
Transform
Load
```

---

## Jobs

Purpose:

Orchestrate:

- Notebooks
- Queries
- Pipelines

Into a complete workflow.

---

# Runs

Used for monitoring executions.

Shows:

- Success status
- Failure status
- Run duration
- Error messages

Useful for debugging.

---

# SQL Analytics

Databricks includes powerful SQL tools.

---

# SQL Editor

Interactive SQL environment.

### Features

- SQL execution
- Auto-complete
- Syntax highlighting
- Query history
- Schema browser

### Example

```sql
SELECT *
FROM employees
LIMIT 10;
```

---

# Queries

Used to save SQL commands.

Benefits:

- Reuse queries
- Organize in folders
- Share with teams
- Use in dashboards

---

# Dashboards

Used for visualization and reporting.

### Supports

- Tables
- Charts
- KPIs
- Filters
- Reports

### Purpose

Convert SQL results into visual insights.

---

# Alerts

Used to notify users based on conditions.

Examples:

```text
Revenue below target
High error count
New records detected
```

Works together with scheduled queries.

---

# Query History

Stores all executed SQL commands.

Useful for:

- Finding old queries
- Performance analysis
- Troubleshooting

---

# Genie and Genie Code

Databricks AI assistants.

---

# Genie Spaces

Designed for business users.

Allows users to ask questions in plain language.

Example:

```text
What was last month's revenue?
```

Genie automatically generates SQL and returns results.

---

# Genie Code

Designed for developers.

Can help with:

- Python
- SQL
- PySpark
- Scala
- R

Functions:

- Generate code
- Debug errors
- Search workspace
- Create assets
- Manage catalog objects

---

# AI and Machine Learning

---

# Playground

Used to experiment with AI models.

Features:

- Prompt testing
- Model comparison
- Parameter tuning

---

# AI Gateway

Provides access to large language models directly inside Databricks.

Benefits:

- No API setup
- Integrated AI access
- Easy experimentation

---

# Experiments (MLflow)

Tracks machine learning training runs.

Stores:

- Parameters
- Metrics
- Model artifacts

Example:

```text
Learning Rate = 0.01
Epochs = 10
Accuracy = 95%
```

---

# Feature Store

Stores reusable machine learning features.

Purpose:

Use the same features across multiple models.

---

# Models (Model Registry)

Stores trained models.

Supports:

- Versioning
- Tracking
- Management

---

# Serving

Deploys ML models as APIs.

Example:

```text
Application
      ↓
REST API
      ↓
Model Prediction
```

---

# Discover

Search engine inside Databricks.

Used to find:

- Tables
- Dashboards
- Notebooks
- Queries

---

# How Data Flows Through Databricks

```text
Data Sources
(CSV, JSON, APIs, Databases)
            ↓
     Ingestion Pipeline
            ↓
        Catalog
            ↓
  Notebooks / SQL Editor
            ↓
    Transformations
            ↓
Dashboards / Reports / ML Models
```

### Important Concept

Everything revolves around the Catalog.

The Catalog is the center of the Databricks platform.

---

# First Steps in Databricks

## Step 1: Create a Volume

```text
Catalog
  ↓
workspace
  ↓
default
  ↓
Create Volume
```

Volume Name:

```text
book_data
```

---

## Step 2: Create a Notebook

```text
Workspace
   ↓
Create
   ↓
Notebook
```

Suggested Name:

```text
Chapter 01 - Getting Started
```

Language:

```text
Python
```

---

## Step 3: Run First Program

```python
print("Hello, Databricks!")
```

---

## Step 4: Explore Sample Data

```sql
%sql

SELECT *
FROM samples.nyctaxi.trips
LIMIT 10;
```

---

## Step 5: Organize Work

Create folders:

```text
Data Engineering Book
│
├── Core Concepts
├── Data Ingestion
├── Data Persistence
├── Performance
├── Machine Learning
└── Projects
```

---

# Compute Auto-Start

Databricks Free Edition is fully serverless.

### What Happens?

```text
Run Notebook
      ↓
Compute Starts Automatically
      ↓
Execution Begins
```

### SQL Execution

```text
Run Query
      ↓
Starter Warehouse Starts
      ↓
Query Executes
```

### Auto Termination

Compute automatically stops after inactivity.

Benefits:

- Saves resources
- No manual management
- Cost efficient

---

# Free Edition Summary

Available Features:

- Workspace
- Catalog
- Volumes
- SQL Editor
- Dashboards
- Jobs & Pipelines
- Genie
- Playground
- MLflow Experiments

Limited/Enterprise Features:

- Advanced Security
- Feature Store Capabilities
- Advanced Scheduling
- Production Governance Features

---

# Quick Revision

## Workspace

Write and organize code.

## Catalog

Stores tables and data.

## Volume

Stores files.

## Compute

Runs code.

## SQL Warehouse

Runs SQL queries.

## SQL Editor

Write SQL.

## Dashboards

Build reports.

## Jobs

Automate workflows.

## Genie

AI Assistant.

## Experiments

Track ML models.

## Serving

Deploy ML models.

---

# Final Takeaway

```text
Workspace  → Write Code
Catalog    → Store Data
Volume     → Store Files
Compute    → Run Code
SQL Editor → Query Data
Dashboards → Visualize Data
Jobs       → Automate Processes
Genie      → AI Assistance
```

Databricks combines data storage, processing, analytics, AI, and machine learning into a single unified platform for data engineers, analysts, and data scientists.
