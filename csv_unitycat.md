# Databricks Unity Catalog Guide
## Loading CSV Files and Table Management

### 1. Loading CSV Files to Unity Catalog

There are two primary methods to load CSV files into Unity Catalog tables:

#### Method 1: Using PySpark DataFrame
```python
# Import required libraries
from pyspark.sql import SparkSession

# Read CSV file
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("/path/to/your/csv/file.csv")

# Write to Unity Catalog table
df.write.format("delta") \
    .mode("overwrite")  # or "append" depending on your needs
    .saveAsTable("catalog_name.schema_name.table_name")
```

#### Method 2: Using SQL
```sql
CREATE TABLE catalog_name.schema_name.table_name
USING CSV
OPTIONS (
    path '/path/to/your/csv/file.csv',
    header 'true',
    inferSchema 'true'
)
```

**Important Notes:**
- Replace `catalog_name`, `schema_name`, and `table_name` with your actual Unity Catalog hierarchy
- Ensure you have proper permissions in the Unity Catalog
- The path to your CSV file can be:
  - DBFS path (e.g., `dbfs:/FileStore/...`)
  - External location registered in Unity Catalog
  - Cloud storage path (e.g., `s3://`, `abfss://`, `gs://`)

### 2. Managed vs External Tables

#### Managed Tables

**When to Use:**
- Data lifecycle management should be handled by Databricks
- Need for ACID transactions
- Internal data processing pipelines
- Critical business data requiring version control

**Example:**
```sql
-- Creating a managed table
CREATE TABLE catalog_name.schema_name.managed_table (
    id INT,
    name STRING,
    value DOUBLE
) USING DELTA;

-- Landing zone for processed data
CREATE TABLE catalog_name.schema_name.sales_facts
USING DELTA
AS SELECT * FROM raw_sales;

-- Critical business data requiring ACID
CREATE TABLE catalog_name.schema_name.customer_orders
USING DELTA
PARTITIONED BY (order_date);
```

**Benefits:**
- Automatic data management (files deleted when table dropped)
- Better performance optimizations
- Easier backup and restore
- ACID transactions guaranteed
- Metadata and data stored together

#### External Tables

**When to Use:**
- Data already exists in cloud storage
- Data needs to be shared across multiple platforms
- Direct control over data files required
- Large historical datasets rarely accessed

**Example:**
```sql
-- Creating an external table
CREATE EXTERNAL TABLE catalog_name.schema_name.external_table
USING DELTA
LOCATION 's3://your-bucket/path/to/data';

-- Raw data from external sources
CREATE EXTERNAL TABLE catalog_name.schema_name.raw_logs
LOCATION 's3://data-lake/raw/logs'
USING DELTA;

-- Data shared with other systems
CREATE EXTERNAL TABLE catalog_name.schema_name.shared_analytics
LOCATION 'abfss://analytics@storage.dfs.core.windows.net/shared'
USING DELTA;
```

**Benefits:**
- Data persists after table drop
- Direct file access for other tools
- More control over storage costs
- Easier data sharing across platforms

### Best Practices

#### For Managed Tables:
- Use for refined/processed data
- Critical business data requiring ACID
- Data that needs version control
- Internal analytics datasets

#### For External Tables:
- Raw data ingestion
- Data shared across platforms
- Compliance requirements needing specific storage
- Large historical datasets rarely accessed
