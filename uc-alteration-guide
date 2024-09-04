# Unity Catalog External Tables: Alteration Guide

## Introduction

Unity Catalog external tables in Databricks allow you to query data stored in external locations (like S3, ADLS, etc.) without moving the data into Databricks-managed storage. While these tables offer great flexibility, they come with certain limitations when it comes to alterations. This guide explains what you can and cannot do when altering external tables in Unity Catalog.

## Allowed Alterations

### 1. Adding New Columns

You can add new columns to an existing external table.

```sql
ALTER TABLE my_catalog.my_schema.external_table ADD COLUMN new_column STRING;
```

This operation adds a new column to the table metadata but doesn't affect the underlying data.

### 2. Renaming the Table

You can rename an external table.

```sql
ALTER TABLE my_catalog.my_schema.external_table RENAME TO new_external_table;
```

This changes the table name in Unity Catalog but doesn't affect the data location.

### 3. Changing Table Properties

You can modify table properties.

```sql
ALTER TABLE my_catalog.my_schema.external_table
SET TBLPROPERTIES ('delta.columnMapping.mode' = 'name');
```

This allows you to update metadata and configure certain behaviors of the table.

## Disallowed Alterations

### 1. Adding Constraints

You cannot add constraints like PRIMARY KEY, FOREIGN KEY, or NOT NULL to external tables.

```sql
-- This will result in an error
ALTER TABLE my_catalog.my_schema.external_table ADD CONSTRAINT pk_id PRIMARY KEY (id);
```

### 2. Modifying Existing Columns

You cannot change the data type or other properties of existing columns.

```sql
-- This will result in an error
ALTER TABLE my_catalog.my_schema.external_table ALTER COLUMN value TYPE DECIMAL(10,2);
```

### 3. Dropping Columns

You cannot drop existing columns from external tables.

```sql
-- This will result in an error
ALTER TABLE my_catalog.my_schema.external_table DROP COLUMN value;
```

### 4. Changing File Format

You cannot change the file format of an external table after creation.

## Reasons for Limitations

These limitations exist because:

1. External tables are based on data stored outside of Unity Catalog.
2. Altering the physical data structure is not possible through table DDL operations.
3. Unity Catalog doesn't have direct control over the underlying data files.

## Best Practices

When working with external tables and needing to make significant structural changes:

1. Create a new external table with the desired structure.
2. Copy or transform the data into the new structure.
3. Drop the old external table and rename the new one if necessary.

```sql
-- Create new table with desired structure
CREATE EXTERNAL TABLE my_catalog.my_schema.new_external_table (
    id INT,
    name STRING,
    value DECIMAL(10,2),
    new_column STRING
)
LOCATION 's3://my-bucket/path/to/new/data'
USING PARQUET;

-- Copy and transform data (you might use Spark for this)
INSERT INTO my_catalog.my_schema.new_external_table
SELECT id, name, CAST(value AS DECIMAL(10,2)), NULL AS new_column
FROM my_catalog.my_schema.external_table;

-- Drop old table and rename new one
DROP TABLE my_catalog.my_schema.external_table;
ALTER TABLE my_catalog.my_schema.new_external_table RENAME TO external_table;
```

## Data Quality and Consistency

While you can't enforce constraints on external tables through Unity Catalog, ensure data quality and consistency by:

1. Implementing checks at the data source.
2. Enforcing rules during the ETL process that populates the external location.
3. Using Databricks' data quality features like Expectations in your data pipelines.

## Conclusion

Understanding these capabilities and limitations is crucial when working with external tables in Unity Catalog. While they offer flexibility in terms of data location, they require careful management and alternative approaches to ensure data integrity and structural changes.
