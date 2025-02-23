# **Detailed Explanation of the Hive (Hive Day 1)**  

The document provides an **introduction to Apache Hive**, its **architecture, components, and query execution flow**. Below is a structured breakdown of the key topics covered.  

---

## **1. What is Hive?**
### **Definition:**
- **Apache Hive** is an **open-source data warehouse system** built on top of Hadoop.
- It is used for **querying and analyzing large datasets** stored in **Hadoop Distributed File System (HDFS)**.
- It eliminates the need to write **complex MapReduce jobs** by allowing users to write **SQL-like queries**.

### **Key Points:**
- Hive **uses SQL-like language** called **HiveQL (HQL)**.
- HiveQL queries are **automatically converted into MapReduce jobs**.
- Hive abstracts the **complexity of Hadoop**, making it accessible to users **without Java knowledge**.
- It organizes data into **tables** and provides **structured access** to HDFS data.

---

## **2. Hive Architecture and Components**
Hive follows a **client-server architecture** with multiple components.

### **Major Components:**
1. **Hive Clients** (Used for accessing Hive)
   - **Thrift Client:** Connects applications using Apache Thrift.
   - **JDBC Client:** Allows Java applications to connect to Hive using a **JDBC driver**.
   - **ODBC Client:** Allows applications using **ODBC** (e.g., Excel, Power BI) to connect to Hive.

2. **Hive Services** (Manage Hive operations)
   - **Beeline:** A command-line shell for submitting queries to **HiveServer2**.
   - **HiveServer2:** Manages multiple clients and processes queries.
   - **Hive Driver:** Receives and processes queries submitted by users.
   - **Hive Compiler:** Parses queries, performs **semantic analysis**, and creates an **execution plan**.
   - **Optimizer:** Improves query performance by **splitting tasks** and optimizing execution.
   - **Execution Engine:** Executes the **compiled and optimized query plan** using **Hadoop MapReduce**.

3. **Metastore** (Stores Metadata)
   - **Stores metadata** about tables, partitions, columns, and data locations.
   - Supports **two configurations**:
     - **Embedded Mode**: Direct JDBC connection.
     - **Remote Mode**: Uses **Thrift service** for external applications.

---

## **3. Hive Query Flow**
### **How Hive Processes a Query**
1. **`executeQuery`**:  
   - The query is submitted through the **Command Line Interface (CLI)** or **Web UI**.
   - The query reaches the **Hive Driver** via JDBC or ODBC.

2. **`getPlan`**:  
   - The **Hive Compiler** parses the query to check syntax and create an **execution plan**.

3. **`getMetaData`**:  
   - The compiler requests metadata from the **Metastore**.

4. **`sendMetaData`**:  
   - The **Metastore** sends metadata back to the compiler.

5. **`sendPlan`**:  
   - The **execution plan** is finalized and sent back to the **Driver**.

6. **`executePlan`**:  
   - The **execution engine** runs the query by submitting **MapReduce jobs** to Hadoop.
   - **Metadata operations** are performed with the Metastore.
   - **Job execution** occurs using **JobTracker (NameNode)** and **TaskTracker (DataNode)**.

7. **`fetchResults`**:  
   - Once execution is complete, results are **retrieved** from Hadoop.

8. **`sendResults`**:  
   - The **final output** is sent back to the user via **Hive interfaces**.

---

## **4. Summary of Hive Features**
| **Feature** | **Description** |
|------------|---------------|
| **SQL-like Language** | Uses **HiveQL (HQL)** for querying. |
| **Abstraction over Hadoop** | Translates queries into **MapReduce jobs** automatically. |
| **Metadata Storage** | Stores table/partition information in a **Metastore**. |
| **Supports Multiple Clients** | Thrift, JDBC, and ODBC clients available. |
| **Execution Optimization** | Uses **query optimization techniques** for better performance. |
| **Integration with Hadoop** | Works with HDFS, MapReduce, and other Hadoop components. |

---

## **Conclusion**
- **Hive is a powerful data warehouse system** for managing **large-scale datasets in Hadoop**.
- It provides **SQL-like access** to data stored in **HDFS**, reducing the need for writing **complex MapReduce programs**.
- The **Hive architecture** includes **clients, services, execution engine, and metastore**.
- **Hive Query Flow** follows a structured process, ensuring **optimized execution**.

<br/>
<br/>

# **Detailed Explanation of HQL Commands**

### **1. Show All Databases**
```sql
SHOW DATABASES;
```
✅ **What It Does:**  
- Lists all available **Hive databases**.

---

### **2. Create a Database**
```sql
CREATE DATABASE hive_db;
```
✅ **What It Does:**  
- Creates a new Hive database named `hive_db`.

---

### **3. Use a Database**
```sql
USE hive_db;
```
✅ **What It Does:**  
- Switches to the `hive_db` database, so all subsequent operations (table creation, queries, etc.) are executed inside this database.

---

## **Creating & Managing Tables**

### **4. Create a Table for CSV Data**
```sql
CREATE TABLE department_data (
    dept_id INT,
    dept_name STRING,
    manager_id INT,
    salary INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```
✅ **What It Does:**  
- Creates a table named `department_data` with **four columns**.
- **CSV Format** → Columns are separated by a **comma (`,`)**.
- **Hive Internal Table** → Data is stored in Hive's **warehouse directory**.

---

### **5. Get Details of a Table**
```sql
DESCRIBE FORMATTED department_data;
```
✅ **What It Does:**  
- Provides a **detailed description** of the `department_data` table, including:
  - Column names and data types.
  - Table type (Managed or External).
  - Storage format.
  - File location.

---

### **6. Load Data from Local Filesystem into Hive Table**
```sql
LOAD DATA LOCAL INPATH 'file:///home/growdataskills/depart_data.csv' INTO TABLE department_data;
```
✅ **What It Does:**  
- Loads data from a **local file** (`depart_data.csv`) into the `department_data` table.
- The `LOCAL` keyword specifies that the data comes from the **local filesystem** (not HDFS).
- **File is MOVED** (not copied) to Hive’s warehouse directory.

---

### **7. Display Column Names in Hive CLI**
```sql
SET hive.cli.print.header = true;
```
✅ **What It Does:**  
- Enables **column headers** in Hive query results.

---

### **8. Drop a Hive Table**
```sql
DROP TABLE department_data;
```
✅ **What It Does:**  
- Deletes the **table and its data** (because it's a **managed table**).

---

### **9. Load Data into a Hive Table from HDFS**
```sql
CREATE TABLE department_data (
    dept_id INT,
    dept_name STRING,
    manager_id INT,
    salary INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

LOAD DATA INPATH '/tmp/input_data/depart_data.csv' INTO TABLE department_data;
```
✅ **What It Does:**  
- Creates the `department_data` table.
- Loads data from **HDFS** (`/tmp/input_data/depart_data.csv`).
- Unlike `LOCAL`, here the file is **moved within HDFS**, not from the local system.

---

## **Working with External Tables**
### **10. Create an External Table in Hive**
```sql
CREATE EXTERNAL TABLE department_date_external (
    dept_id INT,
    dept_name STRING,
    manager_id INT,
    salary INT
) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
LOCATION '/tmp/input_data/';
```
✅ **What It Does:**  
- Creates an **external table** pointing to `/tmp/input_data/` in HDFS.
- **Data is NOT deleted** if the table is dropped (unlike managed tables).

---

## **Working with ARRAY Data Type**
### **11. Create a Table with ARRAY Data Type**
```sql
CREATE TABLE employee (
    id INT,
    name STRING,
    skills ARRAY<STRING>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
COLLECTION ITEMS TERMINATED BY ':';
```
✅ **What It Does:**  
- Defines an **ARRAY** column (`skills`).
- Array elements are **separated by `:`**.

---

### **12. Load Data into ARRAY Table**
```sql
LOAD DATA LOCAL INPATH 'file:///home/growdataskills/array_data.csv' INTO TABLE employee;
```
✅ **What It Does:**  
- Loads data from `array_data.csv` into the `employee` table.

---

### **13. Retrieve First Element from an ARRAY**
```sql
SELECT id, name, skills[0] AS prime_skill FROM employee;
```
✅ **What It Does:**  
- Fetches the **first skill** from the `skills` array for each employee.

---

### **14. ARRAY Functions in Hive**
```sql
SELECT 
    id,
    name,
    SIZE(skills) AS size_of_each_array,
    ARRAY_CONTAINS(skills, "HADOOP") AS knows_hadoop,
    SORT_ARRAY(skills) AS sorted_array
FROM employee;
```
✅ **Explanation of Functions:**
| **Function** | **Description** |
|-------------|---------------|
| `SIZE(skills)` | Returns the number of elements in the array. |
| `ARRAY_CONTAINS(skills, "HADOOP")` | Checks if "HADOOP" exists in the array (returns `true/false`). |
| `SORT_ARRAY(skills)` | Sorts the array in ascending order. |

---

## **Working with MAP Data Type**
### **15. Create a Table with MAP Data Type**
```sql
CREATE TABLE employee_map_data (
    id INT,
    name STRING,
    details MAP<STRING, STRING>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
COLLECTION ITEMS TERMINATED BY '|'
MAP KEYS TERMINATED BY ':';
```
✅ **What It Does:**  
- Defines a **MAP column (`details`)**.
- **Key-value pairs** are **separated by `:`**.
- **Multiple pairs** are **separated by `|`**.

---

### **16. Load Data into MAP Table**
```sql
LOAD DATA LOCAL INPATH 'file:///home/growdataskills/map_data.csv' INTO TABLE employee_map_data;
```
✅ **What It Does:**  
- Loads data from `map_data.csv` into the `employee_map_data` table.

---

### **17. Retrieve Values from MAP**
```sql
SELECT 
    id, 
    name, 
    details["gender"] AS employee_gender 
FROM employee_map_data;
```
✅ **What It Does:**  
- Extracts the **gender value** from the `details` map.

---

### **18. MAP Functions in Hive**
```sql
SELECT 
    id,
    name,
    details,
    SIZE(details) AS size_of_each_map,
    MAP_KEYS(details) AS distinct_map_keys,
    MAP_VALUES(details) AS distinct_map_values
FROM employee_map_data;
```
✅ **Explanation of Functions:**
| **Function** | **Description** |
|-------------|---------------|
| `SIZE(details)` | Returns the number of key-value pairs in the map. |
| `MAP_KEYS(details)` | Returns all **keys** in the map. |
| `MAP_VALUES(details)` | Returns all **values** in the map. |

---

## **Summary**
| **Command** | **Purpose** |
|------------|------------|
| `SHOW DATABASES;` | List all databases in Hive. |
| `CREATE DATABASE hive_db;` | Create a new Hive database. |
| `USE hive_db;` | Switch to a specific database. |
| `CREATE TABLE department_data ...` | Create a table for CSV data. |
| `LOAD DATA LOCAL INPATH ...` | Load data from a local file. |
| `LOAD DATA INPATH ...` | Load data from HDFS. |
| `CREATE EXTERNAL TABLE ...` | Create an external table. |
| `ARRAY Data Type` | Store multiple values in one column. |
| `MAP Data Type` | Store key-value pairs in one column. |

---

## **Conclusion**
- Hive provides **structured storage** for semi-structured data using **ARRAY and MAP**.
- **`LOAD DATA`** is useful for efficiently moving data into tables.
- Functions like **`ARRAY_CONTAINS`**, **`MAP_KEYS`**, and **`SIZE`** help in working with complex data types.

Would you like any modifications or additional details? 😊

<br/>
<br/>


# **Internal Table vs External Table in Hive**

Apache Hive provides two types of tables to store data: **Internal (Managed) Tables** and **External Tables**. The primary difference lies in how Hive manages the data stored in these tables.

---

## **1. Internal Table (Managed Table)**
### **Concept:**
- In an **internal table**, Hive **manages** both the table schema and the data.
- If you **drop** an internal table, Hive **deletes** both the table **metadata and the actual data** from HDFS.
- The data for an internal table is stored in the **Hive warehouse directory** (`/user/hive/warehouse` by default).
- Best suited when **Hive is the only tool accessing the data**.

### **Command Example:**
```sql
-- Create an internal table
CREATE TABLE employees (
    id INT,
    name STRING,
    salary FLOAT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

-- Load data into the internal table
LOAD DATA LOCAL INPATH '/home/user/employees.csv' INTO TABLE employees;

-- Drop the internal table (deletes both table structure & data)
DROP TABLE employees;
```
📌 **Note:** After `DROP TABLE`, the data is also removed from HDFS.

---

## **2. External Table**
### **Concept:**
- In an **external table**, Hive **manages only the metadata**; the data remains in **its original location**.
- If you **drop** an external table, Hive **only removes the metadata**, but **the actual data remains intact**.
- You specify the storage location explicitly using the `LOCATION` clause.
- Useful when **data is shared between multiple tools (like Spark, Pig, or external HDFS processes).**

### **Command Example:**
```sql
-- Create an external table with data stored in a custom location
CREATE EXTERNAL TABLE employees_external (
    id INT,
    name STRING,
    salary FLOAT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/external_data/employees/';

-- Drop the external table (deletes only metadata, data remains in HDFS)
DROP TABLE employees_external;
```
📌 **Note:** Even after `DROP TABLE`, the data at `/user/external_data/employees/` **is not deleted**.

---

## **Key Differences:**

| Feature            | Internal Table                            | External Table                         |
|--------------------|----------------------------------------|--------------------------------------|
| **Data Storage**    | Stored in Hive’s default warehouse (`/user/hive/warehouse/`) | Stored at a custom location specified using `LOCATION` |
| **Data Management** | Hive fully manages the data | Hive manages only metadata, not data |
| **Drop Behavior**   | Deletes both metadata & data | Deletes only metadata, not data |
| **Use Case**        | When Hive owns the data completely | When data is shared with other tools |

---

## **When to Use What?**
- **Use Internal Tables** when:
  - Hive **owns the data**.
  - You don’t need to **share the data** with other frameworks.
  - You want Hive to **manage storage and clean up** unused data automatically.

- **Use External Tables** when:
  - The data **is already stored** in HDFS or an external system.
  - You want to **share** the data across multiple systems.
  - You need **better control** over the storage.

---

<br/>
<br/>

# **Detailed Use Case for External Tables in Hive**  

External tables in Apache Hive are useful when you need to manage data stored outside Hive while still leveraging Hive's query capabilities. Below are some real-world use cases where external tables are preferred over internal tables.

---

## **1. Data Sharing Across Multiple Tools**  
### **Use Case:**  
Suppose an organization uses multiple big data tools such as **Apache Spark, Pig, HDFS CLI, and Hadoop MapReduce**, along with Hive. If data is stored as an **internal table**, it becomes tightly coupled with Hive, making it difficult for other tools to access the data efficiently.  

### **Solution with External Table:**  
- Store raw data in HDFS in a directory like `/data/sales/` and create an **external table** in Hive pointing to this location.
- Other tools like Spark or Pig can access and process this data without being restricted by Hive’s table management.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE sales_data (
    transaction_id STRING,
    customer_id STRING,
    amount FLOAT,
    date STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/data/sales/';
```
📌 **Impact:** If this table is dropped, the **metadata** in Hive is deleted, but the actual data in `/data/sales/` remains intact, allowing other applications to continue using it.

---

## **2. Data Ingestion from External Systems (ETL Pipelines)**  
### **Use Case:**  
A company receives daily transaction logs from multiple sources such as **Kafka, AWS S3, and an RDBMS**. These logs are stored in HDFS but should not be **directly modified** by Hive.  

### **Solution with External Table:**  
- An **external table** in Hive can be created to **read the logs** without moving them into the Hive warehouse.  
- This ensures that data is still available for real-time ingestion by Kafka or other applications.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE web_logs (
    ip_address STRING,
    user_agent STRING,
    url STRING,
    status_code INT,
    timestamp STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION '/data/web_logs/';
```
📌 **Benefit:** This allows **other services to continuously write logs to HDFS** while Hive can **query and analyze** them without interfering.

---

## **3. Retaining Data Even After Table Deletion**  
### **Use Case:**  
An organization stores critical financial reports in HDFS. The data should not be **accidentally deleted** by someone dropping the Hive table.  

### **Solution with External Table:**  
- An **external table** is created, pointing to the financial reports.  
- If a user accidentally executes `DROP TABLE`, only the Hive metadata is removed, but the reports remain intact in HDFS.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE financial_reports (
    report_id STRING,
    report_date STRING,
    revenue FLOAT,
    expenses FLOAT
)
STORED AS PARQUET
LOCATION '/data/finance/reports/';
```
📌 **Advantage:** This prevents critical data loss from accidental `DROP TABLE` operations.

---

## **4. Querying Large Datasets Without Moving Them**  
### **Use Case:**  
A company has **terabytes of historical data** stored in HDFS in **ORC/Parquet** format. Moving such a huge dataset into Hive’s internal warehouse is inefficient and costly.  

### **Solution with External Table:**  
- An external table can be created pointing to the dataset’s HDFS location, avoiding unnecessary duplication.  
- This allows Hive to query the data **without physically moving it**.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE historical_sales (
    order_id STRING,
    product STRING,
    quantity INT,
    price FLOAT,
    order_date STRING
)
STORED AS ORC
LOCATION '/bigdata/historical_sales/';
```
📌 **Benefit:** This reduces **storage redundancy** and speeds up queries by utilizing optimized columnar storage formats like ORC.

---

## **5. Handling Partitioned Data from Data Lake**  
### **Use Case:**  
A company stores structured logs in an **AWS S3 Data Lake**. The logs are **partitioned by date** to improve performance. Moving this data to Hive’s managed tables would be unnecessary.  

### **Solution with External Table:**  
- An external table can be created, and partitions can be added dynamically.  
- This allows Hive to query the partitioned dataset directly in S3.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE user_logs (
    user_id STRING,
    action STRING,
    timestamp STRING
)
PARTITIONED BY (date STRING)
STORED AS PARQUET
LOCATION 's3://company-datalake/user_logs/';
```
```sql
-- Add partitions dynamically
ALTER TABLE user_logs ADD PARTITION (date='2025-02-23') LOCATION 's3://company-datalake/user_logs/2025-02-23/';
```
📌 **Benefit:** Hive can efficiently query partitioned data without affecting the **S3 data lake structure**.

---

## **6. Integrating with BI Tools (Tableau, Power BI, Looker)**  
### **Use Case:**  
Business analysts use BI tools like **Tableau, Power BI, and Looker** to generate reports. The BI tools require direct access to the data, but analysts should not have permission to modify the data.  

### **Solution with External Table:**  
- An external table can be created for BI tools, ensuring they can **read but not alter the data**.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE bi_sales_data (
    product STRING,
    sales INT,
    revenue FLOAT
)
STORED AS PARQUET
LOCATION '/data/bi_reports/sales/';
```
📌 **Benefit:** BI tools can connect to Hive **without risking accidental data deletion or modification**.

---

## **7. Multi-Tenancy & Access Control for Different Teams**  
### **Use Case:**  
A company has different teams (Data Science, Marketing, and Finance), each needing access to shared data in **different locations**. Instead of creating **duplicate datasets**, external tables allow each team to **access specific data portions** with fine-grained permissions.  

### **Solution with External Table:**  
- External tables allow **RBAC (Role-Based Access Control)** to be applied to different datasets.  
- Teams can only access the required data without affecting others.  

### **Example Command:**  
```sql
CREATE EXTERNAL TABLE marketing_data (
    campaign STRING,
    clicks INT,
    conversions FLOAT
)
STORED AS PARQUET
LOCATION '/data/marketing/';
```
```sql
GRANT SELECT ON TABLE marketing_data TO GROUP marketing_team;
```
📌 **Benefit:** Ensures data security while allowing multiple teams to work efficiently.

---

## **Conclusion: When to Use External Tables?**
| **Scenario** | **Use External Table?** |
|-------------|------------------|
| Data needs to be shared across tools (Spark, Pig, BI) | ✅ Yes |
| Large datasets already stored in HDFS/S3 | ✅ Yes |
| Avoiding accidental data loss | ✅ Yes |
| Temporary tables for quick analysis | ❌ No (Use Internal) |
| Hive needs to fully manage the data | ❌ No (Use Internal) |

---

**🔹 Final Thought:**  
External tables provide **flexibility, security, and efficiency** when dealing with large-scale data, making them a **powerful choice for modern big data architectures**. 🚀

<br/>
<br/>

# **`LOAD DATA` Command in Hive (Detailed Explanation)**  

The `LOAD DATA` command in Hive is used to load data from local or HDFS storage into a Hive table. It helps in importing structured data into tables for querying and analysis.

---

## **1. Syntax of `LOAD DATA`**
```sql
LOAD DATA [LOCAL] INPATH '<file_path>' [OVERWRITE] INTO TABLE <table_name> [PARTITION (column_name=value, ...)];
```

### **Explanation of Keywords:**
- **`LOCAL`** → If specified, Hive loads data from the **local filesystem** instead of HDFS.
- **`INPATH`** → Specifies the **file or directory path** of the data source.
- **`OVERWRITE`** → If specified, **existing data** in the table is removed before loading the new data.
- **`INTO TABLE`** → The destination **Hive table** where data will be loaded.
- **`PARTITION`** → (Optional) Used when loading data into a specific partition of a **partitioned table**.

---

## **2. Loading Data into a Managed (Internal) Table**
### **Use Case:**
You have a CSV file `employees.csv` stored in HDFS or your local system, and you want to load it into a Hive table.

### **Example Commands:**
#### **a) Loading Data from Local Filesystem**
```sql
LOAD DATA LOCAL INPATH '/home/user/employees.csv' INTO TABLE employees;
```
📌 **What Happens?**
- The file is **moved** from `/home/user/employees.csv` to Hive’s warehouse directory (`/user/hive/warehouse/employees/`).
- The original file **disappears** from the local system.

---

#### **b) Loading Data from HDFS**
```sql
LOAD DATA INPATH '/hdfs_data/employees.csv' INTO TABLE employees;
```
📌 **What Happens?**
- The file is **moved** from `/hdfs_data/employees.csv` into `/user/hive/warehouse/employees/`.
- The file is **removed** from its original HDFS location.

---

#### **c) Using `OVERWRITE` to Replace Existing Data**
```sql
LOAD DATA LOCAL INPATH '/home/user/employees_new.csv' OVERWRITE INTO TABLE employees;
```
📌 **What Happens?**
- The existing data in the table **is deleted**.
- Only the new data from `employees_new.csv` is loaded.

---

## **3. Loading Data into an External Table**
### **Use Case:**
You have an external table pointing to `/data/employees/`, and you want to load data into it.

### **Example Command:**
```sql
LOAD DATA INPATH '/hdfs_data/employees.csv' INTO TABLE employees_external;
```
📌 **What Happens?**
- The file **moves** from `/hdfs_data/employees.csv` to `/data/employees/`.
- The data is not deleted if you drop the table, as it’s an **external table**.

🔹 **Note:** You usually don’t need `LOAD DATA` for external tables because Hive can directly read data from the external location.

---

## **4. Loading Data into a Partitioned Table**
### **Use Case:**
Your table `sales_data` is **partitioned by year** and you want to load data into the `year=2024` partition.

### **Example Command:**
```sql
LOAD DATA INPATH '/hdfs_data/sales_2024.csv' INTO TABLE sales_data PARTITION (year=2024);
```
📌 **What Happens?**
- The file **moves** into `/user/hive/warehouse/sales_data/year=2024/`.
- Hive automatically updates the partition metadata.

---

## **5. Loading Data into a Table with Buckets**
### **Use Case:**
You have a **bucketed table** and want to load data while ensuring it adheres to bucketing rules.

### **Example Table:**
```sql
CREATE TABLE orders (
    order_id INT,
    product STRING,
    amount FLOAT
)
CLUSTERED BY (order_id) INTO 4 BUCKETS
STORED AS ORC;
```
### **Incorrect Way (Direct `LOAD DATA`)**
```sql
LOAD DATA INPATH '/hdfs_data/orders.csv' INTO TABLE orders;
```
📌 **Issue:** Bucketing does **not** work automatically.

### **Correct Way (Using INSERT)**
```sql
INSERT OVERWRITE TABLE orders SELECT * FROM temp_orders;
```
📌 **Why?** This ensures proper bucketing via **MapReduce jobs**.

---

## **6. Checking Data After Loading**
To verify data loading, use:
```sql
SELECT * FROM employees LIMIT 5;
```
To check partitioned data:
```sql
SHOW PARTITIONS sales_data;
```

---

## **7. Common Issues & Troubleshooting**
### **a) `File does not exist` Error**
🔹 **Cause:** The file path is incorrect or the file is missing.  
✅ **Fix:** Check the file path using:
```sh
hdfs dfs -ls /hdfs_data/
```

---

### **b) `Failed to move to warehouse` Error**
🔹 **Cause:** Hive doesn’t have permission to move the file.  
✅ **Fix:** Change file ownership:
```sh
hdfs dfs -chmod 777 /hdfs_data/employees.csv
```

---

### **c) `Partition column not found` Error**
🔹 **Cause:** You are loading data into a partitioned table but forgot the `PARTITION` clause.  
✅ **Fix:** Use:
```sql
LOAD DATA INPATH '/hdfs_data/sales_2024.csv' INTO TABLE sales_data PARTITION (year=2024);
```

---

## **8. Summary Table**
| **Scenario** | **Command Example** | **What Happens?** |
|-------------|---------------------|------------------|
| Load from local filesystem | `LOAD DATA LOCAL INPATH '/home/user/data.csv' INTO TABLE my_table;` | File moves from local to Hive warehouse. |
| Load from HDFS | `LOAD DATA INPATH '/hdfs_data/data.csv' INTO TABLE my_table;` | File moves within HDFS. |
| Overwrite existing data | `LOAD DATA INPATH '/hdfs_data/data.csv' OVERWRITE INTO TABLE my_table;` | Old data is deleted before new data is loaded. |
| Load into external table | `LOAD DATA INPATH '/hdfs_data/data.csv' INTO TABLE my_ext_table;` | Moves file to external table location. |
| Load into partitioned table | `LOAD DATA INPATH '/hdfs_data/sales.csv' INTO TABLE sales PARTITION (year=2024);` | Moves file into the partition directory. |

---

## **Conclusion**
- `LOAD DATA` **moves** the file from its source location to the **Hive warehouse (internal table)** or the **specified location (external table)**.
- It is **efficient** for loading data **without running MapReduce**.
- For **partitioned or bucketed tables**, ensure data is loaded correctly using partitions and inserts.

<br/>
<br/>

# **Array-Related Functions in Hive**
Hive provides several built-in functions to work with **ARRAY** data types. Below is a list of important **ARRAY functions**, along with their **syntax**, **examples**, and explanations.

---

## **1. `SIZE()` – Get the Number of Elements**
```sql
SELECT SIZE(array_column) FROM table_name;
```
✅ **What It Does:**  
- Returns the **number of elements** in an array.

📌 **Example:**
```sql
SELECT id, name, SIZE(skills) AS skill_count FROM employee;
```
📝 **Output:**
```
+----+--------+-------------+
| id | name   | skill_count |
+----+--------+-------------+
| 101 | Amit  | 4          |
| 102 | Sumit | 5          |
| 103 | Rohit | 3          |
+----+--------+-------------+
```

---

## **2. `ARRAY_CONTAINS()` – Check if an Element Exists**
```sql
SELECT ARRAY_CONTAINS(array_column, 'value') FROM table_name;
```
✅ **What It Does:**  
- Returns `true` if the **array contains the given value**, otherwise `false`.

📌 **Example:**
```sql
SELECT id, name, ARRAY_CONTAINS(skills, 'HADOOP') AS knows_hadoop FROM employee;
```
📝 **Output:**
```
+----+--------+--------------+
| id | name   | knows_hadoop |
+----+--------+--------------+
| 101 | Amit  | true        |
| 102 | Sumit | true        |
| 103 | Rohit | false       |
+----+--------+--------------+
```

---

## **3. `SORT_ARRAY()` – Sort an Array**
```sql
SELECT SORT_ARRAY(array_column) FROM table_name;
```
✅ **What It Does:**  
- Sorts the elements of an **array in ascending order**.

📌 **Example:**
```sql
SELECT id, name, SORT_ARRAY(skills) AS sorted_skills FROM employee;
```
📝 **Output:**
```
+----+--------+-------------------------------+
| id | name   | sorted_skills                 |
+----+--------+-------------------------------+
| 101 | Amit  | ["BIG-DATA", "HADOOP", "HIVE", "SPARK"] |
| 102 | Sumit | ["HADOOP", "HIVE", "OZZIE", "SPARK", "STORM"] |
| 103 | Rohit | ["CASSANDRA", "HBASE", "KAFKA"] |
+----+--------+-------------------------------+
```

---

## **4. `ARRAY()` – Create an Array**
```sql
SELECT ARRAY('value1', 'value2', 'value3') AS new_array;
```
✅ **What It Does:**  
- Creates an array with given elements.

📌 **Example:**
```sql
SELECT ARRAY('HADOOP', 'SPARK', 'HIVE') AS new_skills;
```
📝 **Output:**
```
+--------------------------+
| new_skills               |
+--------------------------+
| ["HADOOP", "SPARK", "HIVE"] |
+--------------------------+
```

---

## **5. `CONCAT()` – Merge Two Arrays**
```sql
SELECT CONCAT(array_column1, array_column2) FROM table_name;
```
✅ **What It Does:**  
- Merges **two arrays** into a **single array**.

📌 **Example:**
```sql
SELECT id, name, CONCAT(skills, ARRAY('AI', 'ML')) AS new_skills FROM employee;
```
📝 **Output:**
```
+----+--------+-------------------------------+
| id | name   | new_skills                    |
+----+--------+-------------------------------+
| 101 | Amit  | ["HADOOP", "HIVE", "SPARK", "BIG-DATA", "AI", "ML"] |
| 102 | Sumit | ["HIVE", "OZZIE", "HADOOP", "SPARK", "STORM", "AI", "ML"] |
| 103 | Rohit | ["KAFKA", "CASSANDRA", "HBASE", "AI", "ML"] |
+----+--------+-------------------------------+
```

---

## **6. `SLICE()` – Extract a Subset of an Array**
```sql
SELECT SLICE(array_column, start_position, length) FROM table_name;
```
✅ **What It Does:**  
- Extracts a **sub-array** starting at `start_position` for `length` elements.

📌 **Example:**
```sql
SELECT id, name, SLICE(skills, 2, 2) AS subset_skills FROM employee;
```
📝 **Output:**
```
+----+--------+----------------+
| id | name   | subset_skills  |
+----+--------+----------------+
| 101 | Amit  | ["HIVE", "SPARK"] |
| 102 | Sumit | ["OZZIE", "HADOOP"] |
| 103 | Rohit | ["CASSANDRA", "HBASE"] |
+----+--------+----------------+
```
📌 **Note:**  
- Indexing starts from **1** (not 0) in Hive.

---

## **7. `POSEXPLODE()` – Convert Array Elements into Rows**
```sql
SELECT id, name, pos, skill FROM employee LATERAL VIEW POSEXPLODE(skills) t AS pos, skill;
```
✅ **What It Does:**  
- **Explodes** the array into **multiple rows**, while keeping the **original position (`pos`)**.

📌 **Example Output:**
```
+----+--------+-----+-----------+
| id | name   | pos | skill     |
+----+--------+-----+-----------+
| 101 | Amit  | 0   | HADOOP    |
| 101 | Amit  | 1   | HIVE      |
| 101 | Amit  | 2   | SPARK     |
| 101 | Amit  | 3   | BIG-DATA  |
| 102 | Sumit | 0   | HIVE      |
| 102 | Sumit | 1   | OZZIE     |
+----+--------+-----+-----------+
```

---

## **8. `EXPLODE()` – Convert Array Elements into Rows (without Index)**
```sql
SELECT id, name, skill FROM employee LATERAL VIEW EXPLODE(skills) t AS skill;
```
✅ **What It Does:**  
- **Converts an array into rows** (like `POSEXPLODE()`, but without index).

📌 **Example Output:**
```
+----+--------+-----------+
| id | name   | skill     |
+----+--------+-----------+
| 101 | Amit  | HADOOP    |
| 101 | Amit  | HIVE      |
| 101 | Amit  | SPARK     |
| 101 | Amit  | BIG-DATA  |
| 102 | Sumit | HIVE      |
| 102 | Sumit | OZZIE     |
+----+--------+-----------+
```

---

## **Summary of Hive Array Functions**
| **Function** | **Description** | **Example** |
|-------------|---------------|-------------|
| `SIZE(array_col)` | Get number of elements in the array | `SIZE(skills)` |
| `ARRAY_CONTAINS(array_col, value)` | Check if a value exists | `ARRAY_CONTAINS(skills, "HADOOP")` |
| `SORT_ARRAY(array_col)` | Sort array elements in ascending order | `SORT_ARRAY(skills)` |
| `ARRAY(value1, value2, ...)` | Create an array | `ARRAY("HADOOP", "SPARK")` |
| `CONCAT(array1, array2)` | Merge two arrays | `CONCAT(skills, ARRAY("AI", "ML"))` |
| `SLICE(array_col, start, length)` | Extract a subset of an array | `SLICE(skills, 2, 2)` |
| `POSEXPLODE(array_col)` | Convert array into rows with index | `LATERAL VIEW POSEXPLODE(skills)` |
| `EXPLODE(array_col)` | Convert array into rows | `LATERAL VIEW EXPLODE(skills)` |

---

## **Conclusion**
- Hive provides powerful **ARRAY functions** to manipulate lists efficiently.
- **`EXPLODE()` and `POSEXPLODE()`** help transform **nested data into tabular format**.
- **Sorting, filtering, and slicing** arrays can be easily done with built-in functions.

<br/>
<br/>

# **Map-Related Functions in Hive**
Hive provides several **MAP functions** to help manipulate key-value pairs efficiently. Below is a list of important **MAP functions** with their **syntax, examples, and explanations**.

---

## **1. `SIZE()` – Get the Number of Key-Value Pairs**
```sql
SELECT SIZE(map_column) FROM table_name;
```
✅ **What It Does:**  
- Returns the **number of key-value pairs** in a map.

📌 **Example:**
```sql
SELECT id, name, SIZE(details) AS map_size FROM employee_map_data;
```
📝 **Output:**
```
+----+--------+----------+
| id | name   | map_size |
+----+--------+----------+
| 101 | Amit  | 3        |
| 102 | Sumit | 4        |
| 103 | Rohit | 2        |
+----+--------+----------+
```
---

## **2. `MAP_KEYS()` – Extract All Keys**
```sql
SELECT MAP_KEYS(map_column) FROM table_name;
```
✅ **What It Does:**  
- Returns an **array of all keys** present in the map.

📌 **Example:**
```sql
SELECT id, name, MAP_KEYS(details) AS keys FROM employee_map_data;
```
📝 **Output:**
```
+----+--------+--------------------------+
| id | name   | keys                     |
+----+--------+--------------------------+
| 101 | Amit  | ["gender", "city", "role"] |
| 102 | Sumit | ["gender", "city", "role", "experience"] |
| 103 | Rohit | ["city", "role"] |
+----+--------+--------------------------+
```
---

## **3. `MAP_VALUES()` – Extract All Values**
```sql
SELECT MAP_VALUES(map_column) FROM table_name;
```
✅ **What It Does:**  
- Returns an **array of all values** present in the map.

📌 **Example:**
```sql
SELECT id, name, MAP_VALUES(details) AS values FROM employee_map_data;
```
📝 **Output:**
```
+----+--------+--------------------------+
| id | name   | values                   |
+----+--------+--------------------------+
| 101 | Amit  | ["Male", "Mumbai", "Developer"] |
| 102 | Sumit | ["Male", "Delhi", "Manager", "5 Years"] |
| 103 | Rohit | ["Bangalore", "Tester"] |
+----+--------+--------------------------+
```
---

## **4. `MAP()` – Create a Map**
```sql
SELECT MAP('key1', 'value1', 'key2', 'value2') AS new_map;
```
✅ **What It Does:**  
- Creates a **map** with given key-value pairs.

📌 **Example:**
```sql
SELECT MAP('gender', 'Male', 'city', 'Mumbai', 'role', 'Engineer') AS employee_info;
```
📝 **Output:**
```
+--------------------------------------+
| employee_info                        |
+--------------------------------------+
| {"gender":"Male", "city":"Mumbai", "role":"Engineer"} |
+--------------------------------------+
```
---

## **5. `GET()` – Retrieve a Value by Key**
```sql
SELECT map_column['key'] FROM table_name;
```
✅ **What It Does:**  
- Extracts the **value associated with a specific key**.

📌 **Example:**
```sql
SELECT id, name, details["city"] AS location FROM employee_map_data;
```
📝 **Output:**
```
+----+--------+-----------+
| id | name   | location  |
+----+--------+-----------+
| 101 | Amit  | Mumbai    |
| 102 | Sumit | Delhi     |
| 103 | Rohit | Bangalore |
+----+--------+-----------+
```
---

## **6. `POSEXPLODE()` – Convert Map into Rows with Keys & Values**
```sql
SELECT id, name, pos, key, value 
FROM employee_map_data 
LATERAL VIEW POSEXPLODE(details) t AS pos, key, value;
```
✅ **What It Does:**  
- Converts the **map into multiple rows** and provides the **position (index)** of each key-value pair.

📌 **Example Output:**
```
+----+--------+-----+--------+-----------+
| id | name   | pos | key    | value     |
+----+--------+-----+--------+-----------+
| 101 | Amit  | 0   | gender | Male      |
| 101 | Amit  | 1   | city   | Mumbai    |
| 101 | Amit  | 2   | role   | Developer |
| 102 | Sumit | 0   | gender | Male      |
| 102 | Sumit | 1   | city   | Delhi     |
| 102 | Sumit | 2   | role   | Manager   |
| 102 | Sumit | 3   | exp    | 5 Years   |
+----+--------+-----+--------+-----------+
```
---

## **7. `EXPLODE()` – Convert Map into Rows (without Index)**
```sql
SELECT id, name, key, value 
FROM employee_map_data 
LATERAL VIEW EXPLODE(details) t AS key, value;
```
✅ **What It Does:**  
- Converts the **map into multiple rows**, but **does not include index**.

📌 **Example Output:**
```
+----+--------+--------+-----------+
| id | name   | key    | value     |
+----+--------+--------+-----------+
| 101 | Amit  | gender | Male      |
| 101 | Amit  | city   | Mumbai    |
| 101 | Amit  | role   | Developer |
| 102 | Sumit | gender | Male      |
| 102 | Sumit | city   | Delhi     |
| 102 | Sumit | role   | Manager   |
| 102 | Sumit | exp    | 5 Years   |
+----+--------+--------+-----------+
```
---

## **Summary of Hive Map Functions**
| **Function** | **Description** | **Example** |
|-------------|---------------|-------------|
| `SIZE(map_col)` | Get the number of key-value pairs | `SIZE(details)` |
| `MAP_KEYS(map_col)` | Extract all keys from the map | `MAP_KEYS(details)` |
| `MAP_VALUES(map_col)` | Extract all values from the map | `MAP_VALUES(details)` |
| `MAP(k1, v1, k2, v2, ...)` | Create a map | `MAP("gender", "Male", "city", "Delhi")` |
| `map_col['key']` | Retrieve value by key | `details["city"]` |
| `POSEXPLODE(map_col)` | Convert map into rows (with index) | `LATERAL VIEW POSEXPLODE(details) t AS pos, key, value;` |
| `EXPLODE(map_col)` | Convert map into rows (without index) | `LATERAL VIEW EXPLODE(details) t AS key, value;` |

---

## **Conclusion**
- Hive **MAP functions** allow easy **storage and retrieval** of **key-value pairs**.
- **`GET()`**, **`MAP_KEYS()`**, and **`MAP_VALUES()`** provide direct access to data.
- **`POSEXPLODE()` and `EXPLODE()`** help transform **nested map data into tabular format**.

