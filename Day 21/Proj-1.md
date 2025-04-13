# Healthcare Data Pipeline Analysis

This analysis covers two files that together implement a healthcare data processing pipeline using Databricks and Delta Live Tables (DLT):

## 1. `spark_code_to_feed_tables.ipynb`

This notebook contains the setup and data loading code for the pipeline:

### Key Components:

1. **Schema Creation**:
   - Creates a schema named `default1` if it doesn't exist

2. **Diagnosis Mapping Data**:
   - Reads a CSV file (`diagnosis_mapping.csv`) from DBFS
   - Displays the diagnosis codes and descriptions
   - Writes the data to a Delta table `default1.raw_diagnosis_map` in append mode

3. **Patient Data Loading**:
   - Reads three CSV files (`patients_daily_file_*.csv`) from DBFS
   - Casts the admission_date column to date type
   - Writes the data to a Delta table `default1.raw_patients_daily` in append mode with schema merging

4. **Cleanup Operations**:
   - Truncates tables (for resetting the pipeline)
   - Drops tables (for complete cleanup)
   - Removes pipeline checkpoints (for resetting streaming state)

## 2. `healthcare_dlt_pipeline_notebook.sql`

This implements a Delta Live Tables pipeline with multiple layers of data processing:

### Pipeline Architecture:

1. **Bronze Layer (Raw Data)**:
   - `diagnostic_mapping`: Static table loaded from `default1.raw_diagnosis_map`
   - `daily_patients`: Streaming table loaded from `default1.raw_patients_daily`

2. **Silver Layer (Cleaned Data)**:
   - `processed_patient_data`: 
     - Joins patient data with diagnosis descriptions
     - Includes data quality constraints (expectations) to drop invalid rows
     - Ensures all required fields are present (non-null)

3. **Gold Layer (Aggregated Data)**:
   - `patient_statistics_by_diagnosis`: 
     - Aggregates patient counts, age statistics by diagnosis
     - Shows unique gender distribution per diagnosis
   - `patient_statistics_by_gender`: 
     - Aggregates patient counts, age statistics by gender
     - Shows unique diagnosis distribution per gender

### Data Flow:

```
Raw CSVs → Bronze Tables → Silver Table → Gold Tables
```

### Quality Controls:

1. **Table Properties**:
   - Each table is tagged with a "quality" property (bronze, silver, gold)
   - Clear comments document each table's purpose

2. **Data Validation**:
   - Silver table enforces constraints on required fields
   - Violating rows are dropped (ON VIOLATION DROP ROW)

3. **Streaming Support**:
   - Patient data is processed as a stream to handle incremental updates
   - Diagnostic mapping is static reference data

## Business Value:

This pipeline transforms raw patient and diagnosis data into valuable business insights:

1. **Operational Visibility**:
   - Tracks patient admissions with complete demographic and diagnostic information

2. **Analytical Capabilities**:
   - Enables analysis of patient distributions by diagnosis and gender
   - Provides statistical summaries (average age, age ranges) for different segments

3. **Data Quality**:
   - Ensures only complete, valid records progress to analytical layers
   - Clear lineage from raw to aggregated data

The pipeline follows medallion architecture best practices with clear separation between raw, cleaned, and aggregated data layers, making it scalable and maintainable for healthcare analytics.


<br/>
<br/>

# Detailed Explanation of the Healthcare DLT Pipeline SQL File

This SQL file implements a Delta Live Tables (DLT) pipeline for processing healthcare data, following a medallion architecture with bronze, silver, and gold layers. Here's a comprehensive breakdown:

## Bronze Layer (Raw Data Ingestion)

### 1. `diagnostic_mapping` Table
```sql
CREATE LIVE TABLE diagnostic_mapping
COMMENT "Bronze table for the diagnosis mapping file"
TBLPROPERTIES ("quality" = "bronze")
AS
SELECT *
FROM default1.raw_diagnosis_map
```

**Purpose**: 
- Ingests raw diagnosis code mappings (code → description)
- Serves as reference data for the pipeline

**Characteristics**:
- Static table (not streaming)
- Tagged with "quality=bronze" property
- Simple pass-through of raw data without transformation

**Source**: 
- `default1.raw_diagnosis_map` (loaded from CSV in the notebook)

**Data Example**:
```
diagnosis_code | diagnosis_description
------------------------------------
D001          | Hypertension
D002          | Diabetes Mellitus
...
```

### 2. `daily_patients` Table
```sql
CREATE OR REFRESH STREAMING TABLE daily_patients
COMMENT "Bronze table for daily patient data"
TBLPROPERTIES ("quality" = "bronze")
AS
SELECT *
FROM STREAM(default1.raw_patients_daily)
```

**Purpose**:
- Ingests streaming patient data from daily files
- First point of entry for patient records

**Characteristics**:
- Streaming table (handles incremental data)
- Tagged with "quality=bronze"
- Pass-through of raw patient data

**Source**:
- `default1.raw_patients_daily` (loaded from CSVs in notebook)

**Data Example**:
```
patient_id | name          | age | gender | address      | contact_number | admission_date | diagnosis_code
-----------------------------------------------------------------------------------------------------
P001       | John Doe      | 45  | M      | 123 Main St  | 555-1234       | 2024-07-01     | D001
P002       | Jane Smith    | 38  | F      | 456 Elm St   | 555-9876       | NULL           | D002
...
```

## Silver Layer (Cleaned, Enriched Data)

### 3. `processed_patient_data` Table
```sql
CREATE OR REFRESH STREAMING TABLE processed_patient_data
(CONSTRAINT valid_data EXPECT (
    patient_id IS NOT NULL and 
    `name` IS NOT NULL and 
    age IS NOT NULL and 
    gender IS NOT NULL and 
    `address` IS NOT NULL and 
    contact_number IS NOT NULL and 
    admission_date IS NOT NULL
) ON VIOLATION DROP ROW)
COMMENT "Silver table with newly joined data from bronze tables and data quality constraints"
TBLPROPERTIES ("quality" = "silver")
AS
SELECT
    p.patient_id,
    p.name,
    p.age,
    p.gender,
    p.address,
    p.contact_number,
    p.admission_date,
    m.diagnosis_description
FROM STREAM(live.daily_patients) p
LEFT JOIN live.diagnostic_mapping m
ON p.diagnosis_code = m.diagnosis_code
```

**Purpose**:
- Creates a cleaned, enriched view of patient data
- Joins patient records with diagnosis descriptions
- Enforces data quality rules

**Key Features**:
1. **Data Quality Constraints**:
   - Ensures all critical fields are non-null
   - Drops invalid rows (ON VIOLATION DROP ROW)
   - Covers: patient_id, name, age, gender, address, contact_number, admission_date

2. **Data Enrichment**:
   - Joins with diagnosis mapping to add human-readable descriptions
   - Uses LEFT JOIN to preserve patients even if diagnosis mapping is missing

3. **Streaming**:
   - Processes incremental updates from the bronze patient table

**Output Structure**:
```
patient_id | name       | age | gender | address     | contact_number | admission_date | diagnosis_description
--------------------------------------------------------------------------------------------------------------
P001       | John Doe   | 45  | M      | 123 Main St | 555-1234       | 2024-07-01     | Hypertension
P003       | Robert J.  | 50  | M      | 789 Maple St| 555-6543       | 2024-07-03     | Chronic Obstructive Pulmonary Disease
...
```

## Gold Layer (Aggregated Business Insights)

### 4. `patient_statistics_by_diagnosis` Table
```sql
CREATE LIVE TABLE patient_statistics_by_diagnosis
COMMENT "Gold table with detailed patient statistics by diagnosis description"
TBLPROPERTIES ("quality" = "gold")
AS
SELECT
    diagnosis_description,
    COUNT(patient_id) AS patient_count,
    AVG(age) AS avg_age,
    COUNT(DISTINCT gender) AS unique_gender_count,
    MIN(age) AS min_age,
    MAX(age) AS max_age
FROM live.processed_patient_data
GROUP BY diagnosis_description
```

**Purpose**:
- Provides analytical insights grouped by diagnosis
- Helps understand patient demographics for each condition

**Metrics Calculated**:
- Patient count per diagnosis
- Average patient age
- Number of distinct genders (diversity measure)
- Age range (min/max)

**Example Output**:
```
diagnosis_description               | patient_count | avg_age | unique_gender_count | min_age | max_age
-----------------------------------------------------------------------------------------
Hypertension                       | 12            | 42.5    | 2                   | 29      | 58
Diabetes Mellitus                  | 8             | 39.2    | 2                   | 31      | 50
...
```

### 5. `patient_statistics_by_gender` Table
```sql
CREATE LIVE TABLE patient_statistics_by_gender
COMMENT "Gold table with detailed patient statistics by gender"
TBLPROPERTIES ("quality" = "gold")
AS
SELECT
    gender,
    COUNT(patient_id) AS patient_count,
    AVG(age) AS avg_age,
    COUNT(DISTINCT diagnosis_description) AS unique_diagnosis_count,
    MIN(age) AS min_age,
    MAX(age) AS max_age
FROM live.processed_patient_data
GROUP BY gender
```

**Purpose**:
- Provides analytical insights grouped by gender
- Helps understand diagnosis distribution across genders

**Metrics Calculated**:
- Patient count per gender
- Average age by gender
- Number of distinct diagnoses per gender
- Age range by gender

**Example Output**:
```
gender | patient_count | avg_age | unique_diagnosis_count | min_age | max_age
----------------------------------------------------------------------------
M      | 45            | 38.7    | 8                      | 18      | 65
F      | 55            | 42.1    | 10                     | 21      | 70
...
```

## Pipeline Architecture Summary

1. **Data Flow**:
   ```
   Raw Files → Bronze → Silver → Gold
   ```

2. **Layer Responsibilities**:
   - **Bronze**: Raw data ingestion, preserving source fidelity
   - **Silver**: Cleaning, validation, and enrichment
   - **Gold**: Business-level aggregations and metrics

3. **Key Features**:
   - Streaming capabilities for incremental processing
   - Declarative data quality constraints
   - Clear metadata and documentation
   - Proper separation of concerns between layers

This pipeline transforms raw healthcare data into valuable business intelligence while maintaining data quality and enabling both operational and analytical use cases.

<br/>
<br/>

# Detailed Explanation of the `spark_code_to_feed_tables.ipynb` Notebook

This Jupyter notebook (in Databricks format) serves as the data ingestion and pipeline management component for the healthcare data processing system. It works in conjunction with the SQL pipeline file to create an end-to-end solution.

## 1. Notebook Structure Overview

The notebook contains several distinct sections with specific purposes:

1. **Schema Creation**
2. **Diagnosis Mapping Data Load**
3. **Patient Data Load**
4. **Pipeline Management Commands**
5. **Cleanup Operations**

## 2. Detailed Section Breakdown

### 2.1 Schema Creation

```python
%sql
CREATE SCHEMA IF NOT EXISTS default1;
```

**Purpose**:  
- Creates a database schema named `default1` if it doesn't already exist
- Provides a namespace for all pipeline tables to avoid polluting the default database

**Key Points**:
- Uses SQL magic command (`%sql`) to execute DDL statement
- Idempotent operation (won't fail if schema exists)

### 2.2 Diagnosis Mapping Data Load

```python
df = spark.read.option("header", "true").option("inferSchema", "true").csv("dbfs:/FileStore/shashank/raw_data/diagnosis_mapping.csv")
display(df)
df.write.format("delta").mode("append").saveAsTable("default1.raw_diagnosis_map")
```

**Components**:

1. **Data Reading**:
   - Reads CSV from DBFS path `/FileStore/shashank/raw_data/diagnosis_mapping.csv`
   - Options:
     - `header=true`: Uses first row as column names
     - `inferSchema=true`: Automatically detects data types

2. **Data Display**:
   - `display(df)` shows a preview of the loaded data
   - Sample output shows diagnosis codes and descriptions:
     ```
     diagnosis_code | diagnosis_description
     D001          | Hypertension
     D002          | Diabetes Mellitus
     ```

3. **Data Writing**:
   - Writes to Delta format (transactional storage)
   - Mode: `append` (adds to existing data)
   - Saves as managed table `default1.raw_diagnosis_map`

**Data Characteristics**:
- Static reference data (changes infrequently)
- Small dimension table (typically <1000 rows)
- Serves as lookup for diagnosis code descriptions

### 2.3 Patient Data Load

```python
path1 = "dbfs:/FileStore/shashank/raw_data/patients_daily_file_1_2024.csv"
path2 = "dbfs:/FileStore/shashank/raw_data/patients_daily_file_2_2024.csv"
path3 = "dbfs:/FileStore/shashank/raw_data/patients_daily_file_3_2024.csv"

df1 = spark.read.option("header", "true").option("inferSchema", "true").csv(f"{path3}")
df1 = df1.withColumn("admission_date", df1["admission_date"].cast("date"))
display(df1)
df1.write.format("delta").option("mergeSchema", "true").mode("append").saveAsTable("default1.raw_patients_daily")
```

**Components**:

1. **Path Definitions**:
   - Three CSV paths defined (though only path3 is used in this example)
   - Represents daily patient data files

2. **Data Reading**:
   - Similar options as diagnosis mapping (header + schema inference)
   - Additional type casting:
     - Explicitly casts `admission_date` to Date type

3. **Data Display**:
   - Shows sample patient records with schema:
     ```
     patient_id | name       | age | gender | address     | contact_number | admission_date | diagnosis_code
     ```

4. **Data Writing**:
   - Delta format with additional options:
     - `mergeSchema=true`: Allows schema evolution
     - `append` mode: Adds to existing data
   - Saves as `default1.raw_patients_daily`

**Data Characteristics**:
- Fact data (continuously growing)
- Contains PII and sensitive health information
- Sample data shows many NULL values that will be handled in silver layer

### 2.4 Pipeline Management Commands

#### Table Truncation (Reset)
```python
%sql
TRUNCATE TABLE default1.raw_diagnosis_map;
TRUNCATE TABLE default1.raw_patients_daily;
TRUNCATE TABLE default1.diagnostic_mapping;
TRUNCATE TABLE default1.daily_patients;
TRUNCATE TABLE default1.processed_patient_data;
TRUNCATE TABLE default1.patient_statistics_by_diagnosis;
TRUNCATE TABLE default1.patient_statistics_by_gender;
```

**Purpose**:
- Clears all tables while maintaining structure
- Useful for development/testing to reset pipeline state

#### Table Deletion (Cleanup)
```python
%sql
DROP TABLE default1.raw_diagnosis_map;
DROP TABLE default1.raw_patients_daily;
DROP TABLE default1.diagnostic_mapping;
DROP TABLE default1.daily_patients;
DROP TABLE default1.processed_patient_data;
DROP TABLE default1.patient_statistics_by_diagnosis;
DROP TABLE default1.patient_statistics_by_gender;
```

**Purpose**:
- Complete removal of all pipeline tables
- More thorough than truncation (deletes metadata)

#### Checkpoint Removal
```python
dbutils.fs.rm("dbfs:/pipelines/4cc036a7-f01d-48b4-a55a-5e56257a5d81/checkpoints/daily_patients", True)
dbutils.fs.rm("dbfs:/pipelines/4cc036a7-f01d-48b4-a55a-5e56257a5d81/checkpoints/processed_patient_data", True)
```

**Purpose**:
- Deletes streaming checkpoints
- Necessary when resetting streaming pipelines
- Prevents recovery of previous state after code changes

### 3. Key Technical Features

1. **Delta Lake Integration**:
   - All tables use Delta format for ACID transactions
   - Supports time travel and schema evolution

2. **Incremental Processing**:
   - Designed for streaming architecture
   - Checkpoints maintain state for fault tolerance

3. **Data Quality Handling**:
   - Raw data contains NULLs that will be filtered in silver layer
   - Explicit type casting ensures proper data types

4. **Pipeline Management**:
   - Complete lifecycle control (create/reset/delete)
   - Proper cleanup procedures for development cycles

### 4. Operational Workflow

1. **Initial Setup**:
   - Run schema creation once

2. **Daily Operation**:
   - Load new diagnosis mappings (if updated)
   - Append new patient files to raw table
   - Pipeline automatically processes through layers

3. **Maintenance**:
   - Use truncation during development
   - Full cleanup when rebuilding from scratch

### 5. Error Handling Considerations

1. **Schema Evolution**:
   - `mergeSchema=true` handles new columns
   - But may require pipeline updates for structural changes

2. **Data Issues**:
   - NULL handling delegated to silver layer
   - Type casting failures would surface during load

3. **Pipeline Recovery**:
   - Checkpoints enable resume from failure
   - Requires proper cleanup when logic changes

This notebook provides the foundational data loading and management capabilities that feed the SQL transformation pipeline, creating a complete data processing solution.