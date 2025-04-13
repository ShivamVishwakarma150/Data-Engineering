# Booking Data Processing Project Analysis

## Project Requirements Based on Code

### 1. Data Ingestion
- **Input Sources**:
  - Daily booking data in CSV format (`bookings_YYYY-MM-DD.csv`)
  - Daily customer data in CSV format (`customers_YYYY-MM-DD.csv`)
- **Parameters**:
  - Accepts a date parameter (`current_date`) to process data for specific days
- **Reading Data**:
  - Uses Spark to read CSV files with proper handling of headers, schema inference, quotes, and multi-line fields

### 2. Data Quality Checks
- **Booking Data Checks**:
  - Verifies data has rows (`hasSize`)
  - Ensures `booking_id` is unique
  - Validates completeness of `customer_id` and `amount`
  - Checks non-negativity of `amount`, `quantity`, and `discount`
  
- **Customer Data Checks**:
  - Verifies data has rows (`hasSize`)
  - Ensures `customer_id` is unique
  - Validates completeness of `customer_name`, `customer_address`, `phone_number`, and `email`

### 3. Data Processing
- **Booking Data**:
  - Adds ingestion timestamp
  - Joins with customer data
  - Calculates `total_cost` (amount - discount)
  - Filters records with positive quantity
  - Aggregates by `booking_type` and `customer_id` to calculate sums of `total_cost` and `quantity`

- **Customer Data**:
  - Implements SCD Type 2 pattern to track historical changes
  - Updates existing records by setting `valid_to` date when changes occur
  - Appends new records with current date as `valid_from` and '9999-12-31' as `valid_to`

### 4. Data Storage
- **Fact Table** (`booking_fact`):
  - Stores aggregated booking metrics
  - Uses Delta format for reliability and performance
  - Overwrites existing data with complete refreshed aggregates
  
- **Dimension Table** (`customer_scd`):
  - Stores customer information with historical tracking
  - Uses Delta format
  - Implements merge logic for SCD Type 2 updates

## Detailed Explanation

### Data Pipeline Architecture

This project implements a data processing pipeline that:

1. **Ingests** daily snapshots of booking and customer data
2. **Validates** the data meets quality standards before processing
3. **Transforms** the raw data into analytical models
4. **Loads** the results into Delta tables for consumption

### Key Components

1. **Data Quality Framework**:
   - Uses PyDeequ for comprehensive data validation
   - Implements checks at different levels (Error level shown)
   - Provides clear validation results with status messages
   - Stops processing if critical checks fail (as seen with the customer data phone_number check)

2. **Incremental Processing**:
   - Designed to handle daily incremental loads
   - Maintains state by reading existing tables when available
   - For bookings, performs re-aggregation of historical + new data
   - For customers, implements proper SCD Type 2 pattern

3. **Business Logic**:
   - Calculates meaningful metrics like `total_cost`
   - Filters invalid records (zero quantity)
   - Creates aggregated views for analysis

4. **SCD Type 2 Implementation**:
   - Properly handles customer dimension changes over time
   - Updates `valid_to` of previous records when changes occur
   - Inserts new records with current version markers
   - Uses '9999-12-31' to indicate currently active records

### Error Handling

The pipeline includes robust error handling:
- Explicit validation of data quality checks
- Raises ValueError with descriptive messages when checks fail
- Prevents loading of invalid data into target tables

### Technical Implementation Details

- **Spark-based processing** for scalability
- **Delta Lake** for reliable storage with ACID transactions
- **Parameterized execution** via Databricks widgets
- **Schema handling** with inference from CSV files
- **Multi-line support** for address fields containing newlines

### Business Value

This pipeline provides:
- Reliable daily updates of booking metrics
- Historical tracking of customer information changes
- Data quality assurance before loading to analytics
- Aggregated views for reporting and analysis
- Foundation for customer behavior analysis and booking trends

The implementation follows data engineering best practices while addressing common challenges in incremental processing and slowly changing dimensions.

<br/>
<br/>

# Detailed Code Explanation: Booking Data Processing Pipeline

## 1. Initial Setup and Imports

```python
from pyspark.sql.functions import col, lit, current_timestamp, sum as _sum
from delta.tables import DeltaTable
from pydeequ.checks import Check, CheckLevel
from pydeequ.verification import VerificationSuite, VerificationResult
import os
```

- **Spark Functions**: Import essential Spark SQL functions for data manipulation
  - `col`: For column references
  - `lit`: For creating literal values
  - `current_timestamp`: For getting current timestamp
  - `sum`: For aggregation (aliased to avoid conflict with Python's built-in sum)

- **Delta Lake**: `DeltaTable` for managing Delta tables with ACID transactions

- **PyDeequ**: Framework for data quality testing
  - `Check`: Define data quality constraints
  - `CheckLevel`: Severity levels for checks
  - `VerificationSuite`: Execute validation checks
  - `VerificationResult`: Process validation results

- **os**: For accessing environment variables

## 2. Environment Configuration

```python
print(os.environ['SPARK_VERSION'])
```

- Prints the Spark version being used, helpful for debugging and compatibility checks

## 3. Parameter Handling

```python
date_str = dbutils.widgets.get("current_date")
# date_str = "2024-07-26"
```

- Gets the processing date from a Databricks widget (parameter)
- Comment shows hardcoded alternative for testing

## 4. Data Path Configuration

```python
booking_data = f"dbfs:/DataEngineering/bookings_daily_data/bookings_{date_str}.csv"
customer_data = f"dbfs:/DataEngineering/customers_daily_data/customers_{date_str}.csv"
print(booking_data)
print(customer_data)
```

- Constructs dynamic file paths using the date parameter
- Prints paths for verification and logging

## 5. Data Ingestion - Booking Data

```python
booking_df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .option("quote", "\"") \
    .option("multiLine", "true") \
    .load(booking_data)

booking_df.printSchema()
display(booking_data)
```

- Reads CSV with specific configurations:
  - `header=true`: First row contains column names
  - `inferSchema=true`: Automatically detect data types
  - `quote="\""`: Handle quoted fields
  - `multiLine=true`: Support fields with newlines
- Prints schema for validation
- Displays the data path (not the DataFrame itself)

## 6. Data Ingestion - Customer Data

```python
customer_df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .option("quote", "\"") \
    .option("multiLine", "true") \
    .load(customer_data)

customer_df.printSchema()
display(customer_df)
```

- Same configuration as booking data
- Displays the actual customer DataFrame for inspection

## 7. Data Quality Checks - Booking Data

```python
check_incremental = Check(spark, CheckLevel.Error, "Booking Data Check") \
    .hasSize(lambda x: x > 0) \
    .isUnique("booking_id", hint="Booking ID is not unique throughout") \
    .isComplete("customer_id") \
    .isComplete("amount") \
    .isNonNegative("amount") \
    .isNonNegative("quantity") \
    .isNonNegative("discount")
```

- **Size Check**: Ensures dataset isn't empty
- **Uniqueness**: Verifies booking_id is unique
- **Completeness**: customer_id and amount must not be null
- **Non-negative**: Validates amount, quantity, discount ≥ 0

## 8. Data Quality Checks - Customer Data

```python
check_scd = Check(spark, CheckLevel.Error, "Customer Data Check") \
    .hasSize(lambda x: x > 0) \
    .isUnique("customer_id") \
    .isComplete("customer_name") \
    .isComplete("customer_address") \
    .isComplete("phone_number") \
    .isComplete("email")
```

- Similar structure to booking checks
- Focuses on customer dimension attributes
- All checks at "Error" level - will fail processing if violated

## 9. Quality Check Execution

```python
booking_dq_check = VerificationSuite(spark) \
    .onData(booking_df) \
    .addCheck(check_incremental) \
    .run()

customer_dq_check = VerificationSuite(spark) \
    .onData(customer_df) \
    .addCheck(check_scd) \
    .run()
```

- Creates verification suites for each dataset
- Attaches respective checks
- Executes validation

## 10. Quality Check Results

```python
booking_dq_check_df = VerificationResult.checkResultsAsDataFrame(spark, booking_dq_check)
display(booking_dq_check_df)

customer_dq_check_df = VerificationResult.checkResultsAsDataFrame(spark, customer_dq_check)
display(customer_dq_check_df)
```

- Converts results to DataFrames for visualization
- Displays detailed check outcomes

## 11. Quality Gate Enforcement

```python
if booking_dq_check.status != "Success":
    raise ValueError("Data Quality Checks Failed for Booking Data")

if customer_dq_check.status != "Success":
    raise ValueError("Data Quality Checks Failed for Customer Data")
```

- Stops processing if any check fails
- Provides clear error messages

## 12. Data Transformation - Booking Data

```python
booking_df_incremental = booking_df.withColumn("ingestion_time", current_timestamp())
```

- Adds ingestion timestamp for auditing

```python
df_joined = booking_df_incremental.join(customer_df, "customer_id")
```

- Enriches booking data with customer information

```python
df_transformed = df_joined \
    .withColumn("total_cost", col("amount") - col("discount")) \
    .filter(col("quantity") > 0)
```

- Calculates net cost (amount - discount)
- Filters out invalid records (quantity ≤ 0)

## 13. Aggregation

```python
df_transformed_agg = df_transformed \
    .groupBy("booking_type", "customer_id") \
    .agg(
        _sum("total_cost").alias("total_amount_sum"),
        _sum("quantity").alias("total_quantity_sum")
    )
```

- Groups by booking type and customer
- Calculates sum metrics for analysis

## 14. Fact Table Management

```python
fact_table_path = "gds_de_bootcamp.default.booking_fact"
fact_table_exists = spark._jsparkSession.catalog().tableExists(fact_table_path)
```

- Checks if target table exists

```python
if fact_table_exists:
    df_existing_fact = spark.read.format("delta").table(fact_table_path)
    df_combined = df_existing_fact.unionByName(df_transformed_agg, allowMissingColumns=True)
    df_final_agg = df_combined \
        .groupBy("booking_type", "customer_id") \
        .agg(
            _sum("total_amount_sum").alias("total_amount_sum"),
            _sum("total_quantity_sum").alias("total_quantity_sum")
        )
else:
    df_final_agg = df_transformed_agg
```

- **If exists**: Reads current data, combines with new data, re-aggregates
- **If new**: Uses current aggregation as initial load

```python
df_final_agg.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(fact_table_path)
```

- Writes final results as Delta table
- Overwrites completely (consider changing to merge for incremental)

## 15. SCD Type 2 Implementation

```python
scd_table_path = "gds_de_bootcamp.default.customer_scd"
scd_table_exists = spark._jsparkSession.catalog().tableExists(scd_table_path)
```

- Checks if SCD table exists

```python
if scd_table_exists:
    scd_table = DeltaTable.forName(spark, scd_table_path)
    display(scd_table.toDF())
    
    scd_table.alias("scd") \
        .merge(
            source=customer_df.alias("updates"),
            condition="scd.customer_id = updates.customer_id and scd.valid_to = '9999-12-31'"
        ) \
        .whenMatchedUpdate(set={
            "valid_to": "updates.valid_from",
        }) \
        .execute()

    customer_df.write.format("delta").mode("append").saveAsTable(scd_table_path)
else:
    customer_df.write.format("delta").mode("overwrite").saveAsTable(scd_table_path)
```

- **If exists**:
  - Finds current active versions (valid_to = '9999-12-31')
  - Updates their end date to new record's start date
  - Appends new versions
- **If new**: Creates initial table

This pipeline demonstrates a complete ETL process with:
- Robust data validation
- Proper dimensional modeling
- Incremental processing patterns
- Comprehensive error handling


<br/>
<br/>

Here's a detailed flow of the code with sample inputs, outputs, and multiple scenarios:

### Pipeline Flow Overview

1. **Input Retrieval** → 2. **Data Loading** → 3. **Data Validation** → 4. **Processing** → 5. **Storage**

---

## Scenario 1: First-Time Execution (Empty Database)

### Sample Input (bookings_2024-07-26.csv)
```
booking_id,customer_id,booking_date,amount,booking_type,quantity,discount,booking_status
1001,5001,2024-07-26,150,Hotel,2,10,Confirmed
1002,5002,2024-07-26,200,Flight,1,0,Pending
```

### Sample Input (customers_2024-07-26.csv)
```
customer_id,customer_name,customer_address,phone_number,email,valid_from,valid_to
5001,John Doe,"123 Main St",5551234,john@email.com,2024-07-26,9999-12-31
5002,Jane Smith,"456 Oak Ave",5555678,jane@email.com,2024-07-26,9999-12-31
```

### Execution Flow

1. **Data Loading**
   - Reads both CSV files successfully
   - Infers schemas automatically

2. **Data Validation**
   - All checks pass (size >0, unique IDs, complete fields, etc.)

3. **Processing**
   - Calculates: total_cost = amount - discount
   - Filters out any quantity ≤ 0
   - Aggregates by booking_type and customer_id

4. **Storage**
   - Creates new Delta tables since none exist
   - Fact table contains aggregated metrics
   - Customer table stores initial records with valid_to=9999-12-31

### Output Tables

**booking_fact**
```
booking_type|customer_id|total_amount_sum|total_quantity_sum
Hotel       |5001       |280             |4
Flight      |5002       |200             |1
```

**customer_scd**
```
customer_id|customer_name|...|valid_from  |valid_to
5001      |John Doe     |...|2024-07-26 |9999-12-31
5002      |Jane Smith   |...|2024-07-26 |9999-12-31
```

---

## Scenario 2: Incremental Load with Updates

### Next Day Input (bookings_2024-07-27.csv)
```
1003,5001,2024-07-27,300,Hotel,1,20,Confirmed
1004,5003,2024-07-27,150,Flight,2,0,Confirmed
```

### Customer Updates (customers_2024-07-27.csv)
```
5001,John Doe,"123 Main St Apt 2",5551234,john.new@email.com,2024-07-27,9999-12-31
5003,Alice Brown,"789 Pine Rd",5559012,alice@email.com,2024-07-27,9999-12-31
```

### Execution Flow

1. **Data Validation**
   - Phone number check fails for Alice (missing)
   - Pipeline stops with "Data Quality Checks Failed" error

### Resolution
- Fix customer data by adding phone number
- Re-run pipeline

### Successful Output

**booking_fact** (updated)
```
booking_type|customer_id|total_amount_sum|total_quantity_sum
Hotel       |5001       |580             |5
Flight      |5002       |200             |1
Flight      |5003       |300             |2
```

**customer_scd** (updated)
```
customer_id|customer_name|...|valid_from  |valid_to
5001      |John Doe     |...|2024-07-26 |2024-07-27
5001      |John Doe     |...|2024-07-27 |9999-12-31
5002      |Jane Smith   |...|2024-07-26 |9999-12-31
5003      |Alice Brown  |...|2024-07-27 |9999-12-31
```

---

## Scenario 3: Data Quality Failure

### Corrupted Input (bookings_2024-07-28.csv)
```
booking_id,customer_id,amount,booking_type,quantity,discount
1005,5004,-100,Hotel,1,50
1006,,300,Flight,0,0
```

### Execution Flow

1. **Data Validation** fails on:
   - Negative amount (-100)
   - Null customer_id
   - Zero quantity
   - Completeness check fails

2. **Pipeline Behavior**
   - Displays detailed validation failures
   - Raises "Data Quality Checks Failed for Booking Data"
   - No data written to tables

### Error Output
```
Booking Data Check - Error - Failure
Constraint: amount is non-negative - Failed
Constraint: customer_id is complete - Failed
Constraint: quantity is non-negative - Failed
```

---

## Scenario 4: Schema Evolution

### New Input with Additional Column (bookings_2024-07-29.csv)
```
booking_id,customer_id,amount,booking_type,quantity,discount,booking_status,payment_method
1007,5002,400,Hotel,2,30,Confirmed,Credit
```

### Execution Flow

1. **Schema Handling**
   - `inferSchema` detects new payment_method column
   - `overwriteSchema=true` allows table schema update

2. **Output**
   - Fact table now includes payment_method in schema
   - Existing data has null for new column

---

## Key Processing Patterns

1. **Incremental Aggregation**
   - New fact data merged with historical aggregates
   - Example: Hotel totals accumulate across runs

2. **SCD Type 2 Implementation**
   - Customer address change creates versioned records:
     ```
     5001 | 123 Main St       | 2024-07-26 | 2024-07-27
     5001 | 123 Main St Apt 2 | 2024-07-27 | 9999-12-31
     ```

3. **Quality Gates**
   - Stops pipeline on any Error-level validation failure
   - Provides detailed constraint violation reports

4. **Idempotent Operation**
   - Can be safely re-run with same inputs
   - Delta Lake ensures atomic writes

This implementation handles both initial loads and incremental updates while maintaining data integrity through rigorous validation checks.

<br/>
<br/>


## Here's the code separated into two distinct scripts - one for processing booking data (Fact Table) and another for processing customer data (Dimension Table with SCD Type 2):


### **1. Booking Data Processing (Fact Table)**
```python
from pyspark.sql.functions import col, lit, current_timestamp, sum as _sum
from delta.tables import DeltaTable
from pydeequ.checks import Check, CheckLevel
from pydeequ.verification import VerificationSuite, VerificationResult
import os

print(os.environ['SPARK_VERSION'])

# Get job parameters from Databricks
date_str = dbutils.widgets.get("current_date")
# date_str = "2024-07-26"

# Define file paths
booking_data = f"dbfs:/DataEngineering/bookings_daily_data/bookings_{date_str}.csv"
print(booking_data)

# Read booking data
booking_df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .option("quote", "\"") \
    .option("multiLine", "true") \
    .load(booking_data)

booking_df.printSchema()
display(booking_df)

# Data Quality Checks
check_incremental = Check(spark, CheckLevel.Error, "Booking Data Check") \
    .hasSize(lambda x: x > 0) \
    .isUnique("booking_id") \
    .isComplete("customer_id") \
    .isComplete("amount") \
    .isNonNegative("amount") \
    .isNonNegative("quantity") \
    .isNonNegative("discount")

# Run verification
booking_dq_check = VerificationSuite(spark) \
    .onData(booking_df) \
    .addCheck(check_incremental) \
    .run()

booking_dq_check_df = VerificationResult.checkResultsAsDataFrame(spark, booking_dq_check)
display(booking_dq_check_df)

# Quality gate
if booking_dq_check.status != "Success":
    raise ValueError("Data Quality Checks Failed for Booking Data")

# Add ingestion timestamp
booking_df_incremental = booking_df.withColumn("ingestion_time", current_timestamp())

# Business transformations
df_transformed = booking_df_incremental \
    .withColumn("total_cost", col("amount") - col("discount")) \
    .filter(col("quantity") > 0)

# Aggregation
df_transformed_agg = df_transformed \
    .groupBy("booking_type", "customer_id") \
    .agg(
        _sum("total_cost").alias("total_amount_sum"),
        _sum("quantity").alias("total_quantity_sum")
    )

# Fact table management
fact_table_path = "gds_de_bootcamp.default.booking_fact"
fact_table_exists = spark._jsparkSession.catalog().tableExists(fact_table_path)

if fact_table_exists:
    df_existing_fact = spark.read.format("delta").table(fact_table_path)
    df_combined = df_existing_fact.unionByName(df_transformed_agg, allowMissingColumns=True)
    df_final_agg = df_combined \
        .groupBy("booking_type", "customer_id") \
        .agg(
            _sum("total_amount_sum").alias("total_amount_sum"),
            _sum("total_quantity_sum").alias("total_quantity_sum")
        )
else:
    df_final_agg = df_transformed_agg

display(df_final_agg)

# Write to Delta
df_final_agg.write \
    .format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable(fact_table_path)
```

---

### **2. Customer Data Processing (SCD Type 2 Dimension)**
```python
from pyspark.sql.functions import col, lit, current_timestamp
from delta.tables import DeltaTable
from pydeequ.checks import Check, CheckLevel
from pydeequ.verification import VerificationSuite, VerificationResult
import os

print(os.environ['SPARK_VERSION'])

# Get job parameters
date_str = dbutils.widgets.get("current_date")
# date_str = "2024-07-26"

# Define file path
customer_data = f"dbfs:/DataEngineering/customers_daily_data/customers_{date_str}.csv"
print(customer_data)

# Read customer data
customer_df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .option("quote", "\"") \
    .option("multiLine", "true") \
    .load(customer_data)

customer_df.printSchema()
display(customer_df)

# Data Quality Checks
check_scd = Check(spark, CheckLevel.Error, "Customer Data Check") \
    .hasSize(lambda x: x > 0) \
    .isUnique("customer_id") \
    .isComplete("customer_name") \
    .isComplete("customer_address") \
    .isComplete("phone_number") \
    .isComplete("email")

# Run verification
customer_dq_check = VerificationSuite(spark) \
    .onData(customer_df) \
    .addCheck(check_scd) \
    .run()

customer_dq_check_df = VerificationResult.checkResultsAsDataFrame(spark, customer_dq_check)
display(customer_dq_check_df)

# Quality gate
if customer_dq_check.status != "Success":
    raise ValueError("Data Quality Checks Failed for Customer Data")

# SCD Type 2 implementation
scd_table_path = "gds_de_bootcamp.default.customer_scd"
scd_table_exists = spark._jsparkSession.catalog().tableExists(scd_table_path)

if scd_table_exists:
    scd_table = DeltaTable.forName(spark, scd_table_path)
    display(scd_table.toDF())
    
    # Update existing records and append new versions
    scd_table.alias("scd") \
        .merge(
            source=customer_df.alias("updates"),
            condition="scd.customer_id = updates.customer_id and scd.valid_to = '9999-12-31'"
        ) \
        .whenMatchedUpdate(set={
            "valid_to": "updates.valid_from",
        }) \
        .execute()

    # Append new records
    customer_df.write.format("delta").mode("append").saveAsTable(scd_table_path)
else:
    # Initial load
    customer_df.write.format("delta").mode("overwrite").saveAsTable(scd_table_path)
```

---

### Key Separation Points:

1. **Booking Data Script** focuses on:
   - Fact table processing
   - Business metrics calculation (total_cost)
   - Aggregation by booking_type and customer_id
   - Complete refresh of fact table

2. **Customer Data Script** focuses on:
   - SCD Type 2 implementation
   - Historical versioning
   - Merge operations for dimension updates
   - Data quality checks specific to customer attributes

3. **Shared Components** (replicated in both):
   - Spark environment setup
   - Date parameter handling
   - Data quality framework (PyDeequ)
   - Delta Lake integration

This separation allows you to:
- Run each pipeline independently
- Schedule them at different frequencies
- Scale resources separately
- Maintain cleaner code organization
- Troubleshoot issues more easily


<br/>
<br/>

# Booking Data Processing Flow Analysis

## Code Flow Overview

This PySpark code processes daily booking data, performs data quality checks, applies transformations, and updates a fact table in Delta format. Here's the detailed flow:

1. **Initialization**: Imports necessary libraries and gets job parameters
2. **Data Loading**: Reads booking data from a CSV file for a specific date
3. **Data Quality Checks**: Runs PyDeequ checks on the booking data
4. **Quality Gate**: Fails the job if data quality checks don't pass
5. **Transformations**: Adds ingestion timestamp and calculates total cost
6. **Aggregation**: Groups data by booking type and customer ID
7. **Fact Table Management**: Updates existing fact table or creates new one
8. **Final Write**: Saves aggregated results to Delta table

## Sample Input and Output Scenarios

### Scenario 1: First Run (New Fact Table)

**Input CSV (`bookings_2024-07-26.csv`)**:
```
booking_id,customer_id,booking_type,amount,quantity,discount
B001,C1001,FLIGHT,500.00,2,50.00
B002,C1002,HOTEL,300.00,1,0.00
B003,C1003,FLIGHT,450.00,1,25.00
B004,C1001,CAR,150.00,1,10.00
```

**Processing Steps**:
1. Passes all data quality checks
2. Calculates `total_cost` (amount - discount)
3. Aggregates by booking_type and customer_id

**Output Delta Table**:
```
| booking_type | customer_id | total_amount_sum | total_quantity_sum |
|--------------|-------------|------------------|--------------------|
| FLIGHT       | C1001       | 450.00           | 2                  |
| HOTEL        | C1002       | 300.00           | 1                  |
| FLIGHT       | C1003       | 425.00           | 1                  |
| CAR          | C1001       | 140.00           | 1                  |
```

### Scenario 2: Incremental Update

**Existing Fact Table**:
```
| booking_type | customer_id | total_amount_sum | total_quantity_sum |
|--------------|-------------|------------------|--------------------|
| FLIGHT       | C1001       | 450.00           | 2                  |
| HOTEL        | C1002       | 300.00           | 1                  |
| FLIGHT       | C1003       | 425.00           | 1                  |
| CAR          | C1001       | 140.00           | 1                  |
```

**New Input CSV (`bookings_2024-07-27.csv`)**:
```
booking_id,customer_id,booking_type,amount,quantity,discount
B005,C1002,FLIGHT,600.00,2,60.00
B006,C1003,HOTEL,400.00,2,40.00
B007,C1004,CAR,200.00,1,0.00
```

**Processing Steps**:
1. Passes data quality checks
2. Calculates new records' total_cost
3. Unions with existing fact table
4. Re-aggregates all data

**Updated Fact Table**:
```
| booking_type | customer_id | total_amount_sum | total_quantity_sum |
|--------------|-------------|------------------|--------------------|
| FLIGHT       | C1001       | 450.00           | 2                  |
| HOTEL        | C1002       | 300.00           | 1                  |
| FLIGHT       | C1003       | 425.00           | 1                  |
| CAR          | C1001       | 140.00           | 1                  |
| FLIGHT       | C1002       | 540.00           | 2                  |
| HOTEL        | C1003       | 360.00           | 2                  |
| CAR          | C1004       | 200.00           | 1                  |
```

### Scenario 3: Data Quality Failure

**Input CSV with Issues (`bookings_2024-07-28.csv`)**:
```
booking_id,customer_id,booking_type,amount,quantity,discount
B008,,FLIGHT,500.00,2,50.00
B009,C1005,HOTEL,-300.00,1,0.00
B008,C1006,FLIGHT,450.00,1,25.00
```

**Processing Steps**:
1. Fails data quality checks because:
   - Duplicate booking_id (B008)
   - Missing customer_id (empty)
   - Negative amount (-300.00)
2. Quality gate raises ValueError
3. Job fails without writing to fact table

**Verification Result**:
```
| check               | constraint          | status  |
|---------------------|---------------------|---------|
| Size Constraint     | Size > 0            | Success |
| Uniqueness Constraint | booking_id unique  | Failure |
| Completeness Constraint | customer_id complete | Failure |
| Completeness Constraint | amount complete    | Success |
| Compliance Constraint | amount >= 0        | Failure |
| Compliance Constraint | quantity >= 0      | Success |
| Compliance Constraint | discount >= 0      | Success |
```

### Scenario 4: Zero Quantity Filtering

**Input CSV (`bookings_2024-07-29.csv`)**:
```
booking_id,customer_id,booking_type,amount,quantity,discount
B010,C1007,FLIGHT,500.00,0,50.00
B011,C1008,HOTEL,300.00,1,0.00
B012,C1009,FLIGHT,450.00,-1,25.00
```

**Processing Steps**:
1. Passes initial data quality checks (negative quantity caught by PyDeequ)
2. Filters out records with quantity <= 0
3. Only B011 remains for processing

**Output (if standalone run)**:
```
| booking_type | customer_id | total_amount_sum | total_quantity_sum |
|--------------|-------------|------------------|--------------------|
| HOTEL        | C1008       | 300.00           | 1                  |
```

## Key Features of the Code

1. **Robust Data Quality**: Comprehensive checks before processing
2. **Incremental Processing**: Handles both new and existing fact tables
3. **Business Logic**: Calculates total cost and filters invalid quantities
4. **Idempotent Writes**: Overwrites the fact table completely each time
5. **Auditability**: Adds ingestion timestamp to track when data was processed

The code demonstrates a complete ETL pipeline with quality gates, transformations, and fact table management suitable for a data warehouse implementation.

<br/>
<br/>

# Customer Data Processing (SCD Type 2 Dimension) Flow Analysis

## Code Flow Overview

This PySpark code processes customer dimension data with SCD (Slowly Changing Dimension) Type 2 implementation. Here's the detailed flow:

1. **Initialization**: Imports libraries and gets job parameters
2. **Data Loading**: Reads customer data from CSV for a specific date
3. **Data Quality Checks**: Runs PyDeequ checks on customer data
4. **Quality Gate**: Fails job if data quality checks don't pass
5. **SCD Type 2 Implementation**:
   - Checks if dimension table exists
   - For existing table: Updates current records and appends new versions
   - For new table: Creates initial load
6. **Final Write**: Maintains historical versions with validity dates

## Sample Input and Output Scenarios

### Scenario 1: Initial Load (Table Doesn't Exist)

**Input CSV (`customers_2024-07-26.csv`)**:
```
customer_id,customer_name,customer_address,phone_number,email,valid_from
C1001,John Doe,123 Main St,555-0101,john@example.com,2024-07-26
C1002,Jane Smith,456 Oak Ave,555-0102,jane@example.com,2024-07-26
```

**Processing Steps**:
1. Passes all data quality checks
2. Creates new SCD table with all records as current (valid_to=9999-12-31)

**Output Delta Table**:
```
| customer_id | customer_name | customer_address | phone_number | email            | valid_from | valid_to   |
|-------------|---------------|------------------|--------------|------------------|------------|------------|
| C1001       | John Doe      | 123 Main St      | 555-0101     | john@example.com | 2024-07-26 | 9999-12-31 |
| C1002       | Jane Smith    | 456 Oak Ave      | 555-0102     | jane@example.com | 2024-07-26 | 9999-12-31 |
```

### Scenario 2: Update with Address Change

**Existing SCD Table**:
```
| customer_id | customer_name | customer_address | phone_number | email            | valid_from | valid_to   |
|-------------|---------------|------------------|--------------|------------------|------------|------------|
| C1001       | John Doe      | 123 Main St      | 555-0101     | john@example.com | 2024-07-26 | 9999-12-31 |
| C1002       | Jane Smith    | 456 Oak Ave      | 555-0102     | jane@example.com | 2024-07-26 | 9999-12-31 |
```

**New Input CSV (`customers_2024-07-27.csv`)**:
```
customer_id,customer_name,customer_address,phone_number,email,valid_from
C1001,John Doe,789 Pine Rd,555-0101,john@example.com,2024-07-27
C1002,Jane Smith,456 Oak Ave,555-0102,jane@example.com,2024-07-27
C1003,Robert Brown,321 Elm St,555-0103,robert@example.com,2024-07-27
```

**Processing Steps**:
1. Passes data quality checks
2. For C1001 (address change):
   - Closes current version (sets valid_to=2024-07-27)
   - Adds new version with new address (valid_from=2024-07-27, valid_to=9999-12-31)
3. For C1002 (no changes):
   - Keeps existing record (no action needed)
4. For C1003 (new customer):
   - Adds as new record

**Updated SCD Table**:
```
| customer_id | customer_name | customer_address | phone_number | email            | valid_from | valid_to   |
|-------------|---------------|------------------|--------------|------------------|------------|------------|
| C1001       | John Doe      | 123 Main St      | 555-0101     | john@example.com | 2024-07-26 | 2024-07-27 |
| C1001       | John Doe      | 789 Pine Rd      | 555-0101     | john@example.com | 2024-07-27 | 9999-12-31 |
| C1002       | Jane Smith    | 456 Oak Ave      | 555-0102     | jane@example.com | 2024-07-26 | 9999-12-31 |
| C1003       | Robert Brown  | 321 Elm St       | 555-0103     | robert@example.com | 2024-07-27 | 9999-12-31 |
```

### Scenario 3: Data Quality Failure

**Input CSV with Issues (`customers_2024-07-28.csv`)**:
```
customer_id,customer_name,customer_address,phone_number,email,valid_from
C1004,,101 Maple Dr,555-0104,mary@example.com,2024-07-28
C1001,John Doe,789 Pine Rd,555-0101,,2024-07-28
C1005,Chris Lee,202 Cedar Ln,555-0105,chris@example.com,2024-07-28
C1004,Mary Johnson,101 Maple Dr,555-0104,mary@example.com,2024-07-28
```

**Processing Steps**:
1. Fails data quality checks because:
   - Duplicate customer_id (C1004)
   - Missing customer_name (empty)
   - Missing email (empty)
2. Quality gate raises ValueError
3. Job fails without modifying SCD table

**Verification Result**:
```
| check               | constraint          | status  |
|---------------------|---------------------|---------|
| Size Constraint     | Size > 0            | Success |
| Uniqueness Constraint | customer_id unique | Failure |
| Completeness Constraint | customer_name complete | Failure |
| Completeness Constraint | customer_address complete | Success |
| Completeness Constraint | phone_number complete | Success |
| Completeness Constraint | email complete     | Failure |
```

### Scenario 4: Multiple Changes Over Time

**Initial Table State**:
```
| customer_id | customer_name | customer_address | phone_number | email            | valid_from | valid_to   |
|-------------|---------------|------------------|--------------|------------------|------------|------------|
| C1001       | John Doe      | 123 Main St      | 555-0101     | john@example.com | 2024-07-26 | 9999-12-31 |
```

**Sequence of Updates**:

1. **First Update (2024-08-01)** - Phone number change:
   ```
   C1001,John Doe,123 Main St,555-0199,john@example.com,2024-08-01
   ```

2. **Second Update (2024-08-15)** - Name change:
   ```
   C1001,Johnathan Doe,123 Main St,555-0199,john@example.com,2024-08-15
   ```

**Final SCD Table**:
```
| customer_id | customer_name  | customer_address | phone_number | email            | valid_from | valid_to   |
|-------------|----------------|------------------|--------------|------------------|------------|------------|
| C1001       | John Doe       | 123 Main St      | 555-0101     | john@example.com | 2024-07-26 | 2024-08-01 |
| C1001       | John Doe       | 123 Main St      | 555-0199     | john@example.com | 2024-08-01 | 2024-08-15 |
| C1001       | Johnathan Doe  | 123 Main St      | 555-0199     | john@example.com | 2024-08-15 | 9999-12-31 |
```

## Key Features of the Code

1. **SCD Type 2 Implementation**: Maintains full history of dimension changes
2. **Data Quality Enforcement**: Strict checks before processing
3. **Merge Logic**: Properly handles versioning with valid_from/valid_to dates
4. **Idempotent Operations**: Can be safely rerun
5. **Incremental Processing**: Only updates changed records

The code demonstrates a robust dimension table implementation that tracks historical changes while ensuring data quality, suitable for data warehouse environments.