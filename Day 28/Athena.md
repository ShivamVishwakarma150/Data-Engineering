# **Querying S3 Data with AWS Athena: Step-by-Step Guide**

AWS Athena allows you to run SQL queries directly on data stored in Amazon S3 without needing to load it into a database. Here's a **detailed step-by-step process** to use Athena with S3:

---

## **Step 1: Prepare Your Data in S3**
✅ **Data Format**: Athena supports:
   - **Structured**: CSV, JSON, Parquet, ORC, Avro
   - **Best for Performance**: **Parquet/ORC** (columnar, compressed)
   
✅ **Data Organization**:
   - Store files in a logical folder structure (e.g., `s3://my-bucket/sales/year=2023/month=01/`).
   - Use **Hive-style partitioning** (e.g., `year=2023/month=01`) for faster queries.

**Example S3 Structure**:
```
s3://my-data-bucket/
├── sales/
│   ├── year=2023/
│   │   ├── month=01/
│   │   │   ├── data.parquet
│   │   ├── month=02/
│   │   │   ├── data.parquet
```

---

## **Step 2: Set Up AWS Glue Data Catalog (Optional but Recommended)**
Athena uses **AWS Glue Data Catalog** as its metastore (alternative: built-in Athena catalog).

1. **Go to AWS Glue Console** → **Databases** → **Add database** (e.g., `sales_db`).
2. **Run a Crawler** to auto-detect schema:
   - **Data source**: Your S3 path (`s3://my-data-bucket/sales/`).
   - **IAM role**: Grant Glue permissions to read S3.
   - **Output**: A new table in `sales_db`.

---

## **Step 3: Create a Table in Athena (If Not Using Glue Crawler)**
If you skip Glue, define the table manually in Athena:

1. **Open Athena Console** (in AWS Management Console).
2. **Run a DDL Query** (example for Parquet data):
   ```sql
   CREATE EXTERNAL TABLE sales_data (
     order_id STRING,
     customer_id STRING,
     sale_amount DOUBLE,
     sale_date TIMESTAMP
   )
   PARTITIONED BY (year STRING, month STRING)
   STORED AS PARQUET
   LOCATION 's3://my-data-bucket/sales/'
   ```
3. **Load Partitions** (if using Hive-style partitioning):
   ```sql
   MSCK REPAIR TABLE sales_data;
   ```

---

## **Step 4: Run Queries in Athena**
1. **Select your database** (`sales_db`) in the Athena console.
2. **Execute SQL queries**:
   ```sql
   -- Query all 2023 sales
   SELECT * FROM sales_data 
   WHERE year = '2023' AND month = '01';
   
   -- Aggregation example
   SELECT customer_id, SUM(sale_amount) as total_spent
   FROM sales_data
   GROUP BY customer_id;
   ```
3. **View Results**:
   - Results appear below the query editor.
   - Output is also saved to an S3 location (configurable in **Settings**).

---

## **Step 5: Optimize Performance**
✅ **Use Partitioning**:  
   - Query only relevant partitions (e.g., `WHERE year = '2023'`).  
✅ **Convert to Columnar Formats**:  
   - Convert CSV/JSON → **Parquet/ORC** (reduces scan costs).  
✅ **Compress Data**:  
   - Use **Snappy/Gzip** compression for Parquet/ORC.  
✅ **Limit Scanned Data**:  
   - Use `LIMIT` in exploratory queries.  

---

## **Step 6: Schedule Queries (Optional)**
Use **Amazon EventBridge + Lambda** to run Athena queries on a schedule:
1. **Create a Lambda function** (Python example):
   ```python
   import boto3
   client = boto3.client('athena')
   
   response = client.start_query_execution(
       QueryString='SELECT * FROM sales_data WHERE year = '2023';',
       QueryExecutionContext={'Database': 'sales_db'},
       ResultConfiguration={'OutputLocation': 's3://query-results/'}
   )
   ```
2. **Set up EventBridge Rule** to trigger daily.

---

## **Step 7: Visualize Results (Optional)**
Connect Athena to BI tools:
- **Amazon QuickSight**: Native integration.
- **Tableau/Power BI**: Use Athena JDBC driver.

---

## **Key Considerations**
⚠ **Cost**: $5 per **TB of data scanned** (use partitioning to minimize scans).  
⚠ **File Size**: Avoid too many small files (merge into larger files).  
⚠ **Concurrency Limits**: Default of **20 concurrent queries**.  

---

## **Summary**
1. **Store data** in S3 (preferably Parquet/ORC).  
2. **Define schema** (Glue Crawler or manual DDL).  
3. **Query** using standard SQL.  
4. **Optimize** with partitioning/compression.  

Athena is ideal for **ad-hoc analytics** on S3 data without managing servers! 🚀

<br/>
<br/>

# **Querying CSV Data in S3 with AWS Athena: Step-by-Step Guide**

AWS Athena makes it easy to analyze CSV files stored in Amazon S3 using standard SQL. Here's a detailed walkthrough:

## **Step 1: Prepare Your CSV Files in S3**

### **CSV File Requirements**
✅ **File Format**: Standard CSV with optional header row  
✅ **Encoding**: UTF-8 recommended  
✅ **File Size**: 
   - Avoid many small files (merge into larger files >128MB if possible)
   - Max file size: 5TB
✅ **Storage Location**: Organized folder structure (e.g., `s3://my-bucket/data/raw/`)

### **Example CSV Format**
```
order_id,customer_id,order_date,amount
1001,45678,2023-01-15,125.50
1002,45679,2023-01-16,89.99
```

## **Step 2: Create a Database in Athena**

1. Open **AWS Athena Console**
2. In the query editor, run:
   ```sql
   CREATE DATABASE sales_db;
   ```
3. Select your new database from the dropdown

## **Step 3: Create a Table for Your CSV Data**

```sql
CREATE EXTERNAL TABLE sales_orders (
  order_id STRING,
  customer_id STRING,
  order_date DATE,
  amount DECIMAL(10,2)
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  'separatorChar' = ',',
  'quoteChar' = '"',
  'escapeChar' = '\\',
  'skip.header.line.count' = '1'
)
STORED AS TEXTFILE
LOCATION 's3://my-bucket/data/raw/'
TBLPROPERTIES ('has_encrypted_data'='false');
```

### **Key Parameters Explained**
- `OpenCSVSerde`: Special serializer for CSV files
- `separatorChar`: Field delimiter (comma)
- `skip.header.line.count`: Skips header row
- `LOCATION`: S3 path containing your CSV files

## **Step 4: Query Your CSV Data**

Run SQL queries like:
```sql
-- Basic query
SELECT * FROM sales_orders LIMIT 10;

-- Aggregation example
SELECT 
  customer_id,
  COUNT(*) as order_count,
  SUM(amount) as total_spent
FROM sales_orders
WHERE order_date BETWEEN DATE '2023-01-01' AND DATE '2023-01-31'
GROUP BY customer_id
ORDER BY total_spent DESC;
```

## **Step 5: Optimize CSV Performance**

### **1. Convert to Columnar Formats**
For better performance, convert to Parquet:
```sql
CREATE TABLE sales_orders_parquet
WITH (
  format = 'PARQUET',
  external_location = 's3://my-bucket/data/parquet/'
) AS 
SELECT * FROM sales_orders;
```

### **2. Partition Your Data**
For large datasets, partition by date:
```sql
CREATE EXTERNAL TABLE sales_orders_partitioned (
  order_id STRING,
  customer_id STRING,
  amount DECIMAL(10,2)
)
PARTITIONED BY (order_date DATE)
STORED AS PARQUET
LOCATION 's3://my-bucket/data/partitioned/';

-- Load partitions manually
ALTER TABLE sales_orders_partitioned ADD PARTITION (order_date='2023-01-01') 
LOCATION 's3://my-bucket/data/partitioned/order_date=2023-01-01/';
```

### **3. Compress Your Data**
Athena works well with Gzip-compressed CSVs:
```sql
CREATE EXTERNAL TABLE sales_orders_compressed (...) 
STORED AS TEXTFILE
LOCATION 's3://my-bucket/data/compressed/'
TBLPROPERTIES ('compressionType'='gzip');
```

## **Troubleshooting Common CSV Issues**

❌ **Problem**: "HIVE_BAD_DATA: Error parsing field value"
✅ **Solution**: Verify all rows match the schema and have proper quoting

❌ **Problem**: "No files found in location"
✅ **Solution**: Check S3 path permissions and file extensions (.csv)

❌ **Problem**: Slow queries on large CSV files
✅ **Solution**: Convert to Parquet/ORC format

## **Best Practices for CSV in Athena**

1. **Always include column headers** in your CSV files
2. **Use consistent formatting** across all files
3. **Avoid reserved characters** in field values
4. **Compress files** (Gzip) to reduce storage and query costs
5. **Consider AWS Glue Crawlers** for automatic schema detection

## **Cost Considerations**
- Athena charges $5 per TB of data scanned
- CSV typically scans more data than Parquet/ORC
- Example cost for 10GB CSV:
  - 10GB = 0.01TB → $0.05 per query

By following these steps, you can efficiently query and analyze CSV data in S3 using Athena while optimizing for performance and cost.