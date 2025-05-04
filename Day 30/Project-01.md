# Telecom Customer Subscription Data Processing Project with AWS Glue

This project demonstrates an incremental data loading pipeline for telecom customer subscription data using AWS Glue. Here's a detailed explanation:

## Project Overview

The project processes customer subscription data from CSV files stored in S3, simulating an incremental loading pattern where new data arrives periodically. AWS Glue jobs are used to extract, transform, and load this data into a catalog for analysis.

## Data Structure

The project works with CSV files containing customer subscription information with the following fields:
- `CustomerID`: Unique identifier for each customer
- `FirstName`, `LastName`: Customer name information
- `PlanType`: Whether the customer has a Postpaid or Prepaid plan
- `MonthlyCharge`: The monthly subscription fee
- `SubscriptionDate`: When the customer joined the service

## File Naming Convention

The CSV files follow a naming pattern that includes the date when they were created/processed:
- `customer_subscription_YYYY_MM_DD.csv`
- Example: `customer_subscription_2023_08_21.csv`

## Incremental Loading Simulation

1. **Data Files**: Three CSV files are provided with different dates (2023_08_21, 2023_08_22, 2023_08_23)
2. **Loading Process**: Files are loaded one by one into S3 to simulate incremental data arrival
3. **Manual Triggering**: Glue jobs are run manually after each file is uploaded

## AWS Glue Implementation

The project includes two Glue job scripts:

### 1. `pyspark_in_glue_demo.py`
- Basic script that:
  - Creates a Glue context
  - Reads data from the Glue Data Catalog
  - Converts the DynamicFrame to a Spark DataFrame
  - Prints the schema and shows sample data

### 2. `incremental_data_in_glue.py`
- More advanced script that:
  - Includes job initialization and commit operations
  - Uses a transformation context for tracking
  - Counts records in the DataFrame
  - Properly manages the Glue job lifecycle

Both scripts connect to the same data source:
- Catalog database: `telcom-in-catalog`
- Table name: `telcom_data_gds`

## Workflow

The typical workflow would be:

1. Upload a new CSV file to S3 (e.g., `customer_subscription_2023_08_23.csv`)
2. Run the Glue job manually to process the new data
3. The job reads from the catalog table which should be configured to point to the S3 location
4. Processed data becomes available for analysis

## Potential Enhancements

1. **Automation**: Schedule the Glue jobs or trigger them via S3 events
2. **Data Transformation**: Add data cleaning or enrichment steps
3. **Partitioning**: Organize data by date or other dimensions
4. **Error Handling**: Add robust error handling and logging
5. **Monitoring**: Implement job monitoring and alerting

This project demonstrates a common pattern for incrementally processing new data files as they arrive in S3, using AWS Glue's serverless ETL capabilities.

<br/>
<br/>

# **Input → Processing → Output Flow**

#### **1. Input Layer (Data Sources)**
- **Source**: CSV files stored in Amazon S3  
  - Example files:  
    - `customer_subscription_2023_08_21.csv`  
    - `customer_subscription_2023_08_22.csv`  
    - `customer_subscription_2023_08_23.csv`  
  - **Format**: Structured CSV with headers  
  - **Schema**:  
    ```csv
    CustomerID,FirstName,LastName,PlanType,MonthlyCharge,SubscriptionDate
    ```
- **Incremental Loading**:  
  - Files are uploaded **one by one** to simulate real-world incremental data arrival.  
  - Each file represents a new batch of customer subscription records.  

---

#### **2. Processing Layer (AWS Glue ETL)**
- **AWS Glue Job Execution**:  
  - **Trigger**: Manually executed (can be automated via EventBridge/S3 triggers).  
  - **Scripts Used**:  
    - `pyspark_in_glue_demo.py` (Basic data extraction)  
    - `incremental_data_in_glue.py` (Advanced job with commit tracking)  
  - **Steps**:  
    1. **Read Data**:  
       - Glue fetches data from the **Glue Data Catalog** (`telcom-in-catalog.telcom_data_gds`).  
       - The catalog points to the latest CSV file in S3.  
    2. **DynamicFrame Conversion**:  
       - Raw CSV data is converted into a **Glue DynamicFrame**.  
    3. **Spark DataFrame Transformation**:  
       - DynamicFrame is converted to a **Spark DataFrame** for further processing.  
    4. **Schema Validation**:  
       - `printSchema()` ensures the structure matches expectations.  
    5. **Data Preview**:  
       - `show()` displays sample records.  
    6. **Count Records (Incremental Job)**:  
       - `sparkSubsDf.count()` logs the number of new records processed.  

---

#### **3. Output Layer (Destination)**
- **Output Options** (Depending on Use Case):  
  - **AWS Glue Data Catalog**:  
    - Updated metadata for querying via Athena/Redshift.  
  - **Amazon S3 (Processed Data)**:  
    - Parquet/ORC files for analytics.  
    - Partitioned by date (e.g., `s3://output-bucket/processed_data/year=2023/month=08/`).  
  - **Amazon Redshift/Athena**:  
    - Queryable tables for BI tools (e.g., Tableau).  
  - **AWS Lake Formation**:  
    - Governed data lake with access controls.  

---

### **Flow Diagram**
```
[New CSV File Uploaded to S3]  
       ↓  
[AWS Glue Job Triggered]  
       ↓  
[Read Data → DynamicFrame → Spark DataFrame]  
       ↓  
[Schema Validation + Record Count]  
       ↓  
[Write to Processed Storage (S3/Redshift)]  
       ↓  
[Available for Analytics/Reporting]  
```

---

### **Key Features of the Flow**
1. **Incremental Processing**:  
   - Only new files trigger updates (simulated via manual uploads).  
2. **Serverless ETL**:  
   - Glue handles scaling and resource management.  
3. **Schema Flexibility**:  
   - DynamicFrame accommodates schema changes.  
4. **Reproducible Jobs**:  
   - Scripts include `job.commit()` for job bookmarks (track processed files).  

### **Potential Next Steps**
- **Automate Triggers**: Use S3 Event Notifications to run Glue jobs automatically.  
- **Add Transformations**: Calculate metrics like customer lifetime value (CLV).  
- **Partition Data**: Optimize storage/query performance by date/plan type.  

This flow ensures scalable, incremental processing of telecom subscription data with minimal manual intervention.

<br/>
<br/>

# Here's a **sample input-output flow** for processing one incremental file (`customer_subscription_2023_08_23.csv`) through the AWS Glue job:

---

### **Sample Input → Output Flow**

#### **1. Input (Before Job Execution)**
- **S3 Bucket**:  
  ```
  s3://telcom-raw-data-bucket/
      customer_subscription_2023_08_21.csv (Already processed)
      customer_subscription_2023_08_22.csv (Already processed)
      customer_subscription_2023_08_23.csv <-- NEW FILE UPLOADED
  ```
- **Glue Data Catalog**:  
  - Table `telcom_data_gds` points to the latest S3 path (e.g., `s3://telcom-raw-data-bucket/customer_subscription_2023_08_23.csv`).

---

#### **2. Glue Job Execution (`incremental_data_in_glue.py`)**
- **Step 1: Read Data**  
  ```python
  subsDf = glueContext.create_dynamic_frame.from_catalog(
      database="telcom-in-catalog",
      table_name="telcom_data_gds",
      transformation_ctx="s3_input_new"
  )
  ```
  - **Input**: `customer_subscription_2023_08_23.csv` (20 new records).  
  - **Output**: DynamicFrame with schema inferred.

- **Step 2: Convert to Spark DataFrame**  
  ```python
  sparkSubsDf = subsDf.toDF()
  ```
  - **Output**: Spark DataFrame with columns:  
    ```
    [CustomerID: int, FirstName: string, LastName: string, 
     PlanType: string, MonthlyCharge: int, SubscriptionDate: string]
    ```

- **Step 3: Print Schema & Sample Data**  
  ```python
  sparkSubsDf.printSchema()
  sparkSubsDf.show(5)
  ```
  - **Output (Console)**:
    ```
    root
      |-- CustomerID: integer (nullable = true)
      |-- FirstName: string (nullable = true)
      |-- LastName: string (nullable = true)
      |-- PlanType: string (nullable = true)
      |-- MonthlyCharge: integer (nullable = true)
      |-- SubscriptionDate: string (nullable = true)

    +----------+---------+--------+--------+-------------+----------------+
    |CustomerID|FirstName|LastName|PlanType|MonthlyCharge|SubscriptionDate|
    +----------+---------+--------+--------+-------------+----------------+
    |      1301| Benjamin|  Barnes|Postpaid|           47|      2022-02-01|
    |      1302|  Bernard|   Hayes| Prepaid|           28|      2022-03-25|
    |      1303|  Bethany|Simmons|Postpaid|           54|      2022-05-10|
    |      1304|    Betty|    Gray| Prepaid|           29|      2022-07-05|
    |      1305|  Beverly|   Woods|Postpaid|           51|      2022-09-12|
    +----------+---------+--------+--------+-------------+----------------+
    ```

- **Step 4: Count Records**  
  ```python
  print(sparkSubsDf.count())  # Output: 20
  ```

---

#### **3. Output (After Job Execution)**
- **Processed Data in S3**:  
  - Parquet files written to `s3://telcom-processed-bucket/2023/08/23/` (if configured).  
  - Example file: `part-00000.parquet` (20 records).

- **Glue Data Catalog Updates**:  
  - New partitions (if partitioned by date) or updated table metadata.

- **Athena/Redshift Query Results**:  
  ```sql
  SELECT COUNT(*) FROM telcom_processed_db.subscriptions 
  WHERE SubscriptionDate = '2023-08-23';
  ```
  - **Output**: `20` (new records).

---

### **Visual Flow**
```
[Upload customer_subscription_2023_08_23.csv to S3]  
       ↓  
[Glue Job Reads NEW 20 Records]  
       ↓  
[Convert → Validate → Count]  
       ↓  
[Write to S3 (Parquet) + Update Catalog]  
       ↓  
[Analytics Ready in Athena/Redshift]  
```

---

### **Key Points**
- **Input**: 1 new CSV → 20 records.  
- **Output**:  
  - Console logs (schema, sample data, count).  
  - Processed data in analytics-ready format (Parquet).  
  - Catalog updates for querying.  
- **Incremental**: Only the new file (`2023_08_23`) is processed.

<br/>
<br/>

## Here's a **file-by-file output flow** showing how each CSV is processed incrementally by the AWS Glue job, including sample outputs at each stage:

### **File 1: `customer_subscription_2023_08_21.csv` (Initial Load)**
**Input**:  
- 20 records (CustomerID 1101–1120).  
- S3 Path: `s3://telcom-raw-data/customer_subscription_2023_08_21.csv`.  

**Glue Job Output**:  
1. **Console Logs**:  
   ```python
   # Schema Output:
   root
    |-- CustomerID: integer
    |-- FirstName: string
    |-- LastName: string
    |-- PlanType: string
    |-- MonthlyCharge: integer
    |-- SubscriptionDate: string

   # Sample Data (sparkSubsDf.show(3)):
   +----------+---------+--------+--------+-------------+----------------+
   |CustomerID|FirstName|LastName|PlanType|MonthlyCharge|SubscriptionDate|
   +----------+---------+--------+--------+-------------+----------------+
   |      1101|   Aaron|   Smith|Postpaid|           50|      2022-01-01|
   |      1102| Abigail| Johnson| Prepaid|           30|      2022-02-15|
   |      1103|    Adam|Williams|Postpaid|           55|      2022-04-10|
   +----------+---------+--------+--------+-------------+----------------+

   # Record Count: 
   20
   ```

2. **Data Catalog**:  
   - Table `telcom_data_gds` now points to this file.  

3. **Processed Output (S3)**:  
   - Parquet files written to:  
     `s3://telcom-processed-bucket/year=2023/month=08/day=21/part-00000.parquet` (20 records).  

---

### **File 2: `customer_subscription_2023_08_22.csv` (Incremental Load)**
**Input**:  
- 10 **new** records (CustomerID 1201–1210).  
- S3 Path: `s3://telcom-raw-data/customer_subscription_2023_08_22.csv`.  

**Glue Job Output**:  
1. **Console Logs**:  
   ```python
   # Schema (unchanged):
   root
    |-- CustomerID: integer
    |-- FirstName: string
    |-- LastName: string
    |-- PlanType: string
    |-- MonthlyCharge: integer
    |-- SubscriptionDate: string

   # Sample Data (sparkSubsDf.show(3)):
   +----------+---------+--------+--------+-------------+----------------+
   |CustomerID|FirstName|LastName|PlanType|MonthlyCharge|SubscriptionDate|
   +----------+---------+--------+--------+-------------+----------------+
   |      1201|  Angela|   Evans|Postpaid|           48|      2022-01-12|
   |      1202|Angelina|    Hall| Prepaid|           29|      2022-03-15|
   |      1203|   Anita| Collins|Postpaid|           56|      2022-04-20|
   +----------+---------+--------+--------+-------------+----------------+

   # Record Count: 
   10
   ```

2. **Data Catalog**:  
   - Updated to include the new file’s metadata.  

3. **Processed Output (S3)**:  
   - New Parquet files written to:  
     `s3://telcom-processed-bucket/year=2023/month=08/day=22/part-00000.parquet` (10 records).  

4. **Aggregate Data (Athena Query)**:  
   ```sql
   SELECT COUNT(*) FROM telcom_processed_db.subscriptions 
   WHERE SubscriptionDate BETWEEN '2022-01-01' AND '2023-08-22';
   ```
   **Output**: `30` (20 + 10 records).  

---

### **File 3: `customer_subscription_2023_08_23.csv` (Incremental Load)**
**Input**:  
- 20 **new** records (CustomerID 1301–1320).  
- S3 Path: `s3://telcom-raw-data/customer_subscription_2023_08_23.csv`.  

**Glue Job Output**:  
1. **Console Logs**:  
   ```python
   # Sample Data (sparkSubsDf.show(3)):
   +----------+---------+--------+--------+-------------+----------------+
   |CustomerID|FirstName|LastName|PlanType|MonthlyCharge|SubscriptionDate|
   +----------+---------+--------+--------+-------------+----------------+
   |      1301|Benjamin|  Barnes|Postpaid|           47|      2022-02-01|
   |      1302| Bernard|   Hayes| Prepaid|           28|      2022-03-25|
   |      1303| Bethany|Simmons|Postpaid|           54|      2022-05-10|
   +----------+---------+--------+--------+-------------+----------------+

   # Record Count: 
   20
   ```

2. **Processed Output (S3)**:  
   - New Parquet files written to:  
     `s3://telcom-processed-bucket/year=2023/month=08/day=23/part-00000.parquet` (20 records).  

3. **Aggregate Data (Athena Query)**:  
   ```sql
   SELECT PlanType, COUNT(*) as Total 
   FROM telcom_processed_db.subscriptions 
   GROUP BY PlanType;
   ```
   **Output**:  
   ```
   PlanType | Total
   -----------------
   Postpaid | 25
   Prepaid  | 25
   ```
   (Total records: 50 = 20 + 10 + 20).  

---

### **Key Observations**
1. **Incremental Processing**:  
   - Each job run processes only the new file’s records.  
   - Record counts accumulate across runs (`20 → 30 → 50`).  

2. **Partitioned Output**:  
   - S3 output is partitioned by date (`year=2023/month=08/day=XX`).  

3. **Idempotent Jobs**:  
   - Re-running the job for the same file won’t duplicate data (Glue bookmarks track processed files).  

4. **Schema Consistency**:  
   - All files share the same schema, ensuring seamless merging.  

---

### **Visual Summary**
```
File 1 (2023-08-21) → 20 records → Output to /day=21/
File 2 (2023-08-22) → 10 records → Output to /day=22/ (Total: 30)
File 3 (2023-08-23) → 20 records → Output to /day=23/ (Total: 50)
``` 

This flow ensures efficient incremental processing with traceable outputs at each step.