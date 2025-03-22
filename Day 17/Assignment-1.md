This assignment involves **analyzing marketing campaign data** using **PySpark** and **Hive**. The goal is to process large-scale datasets related to ad campaigns, user profiles, and store information, perform specific analytical tasks, and store the results in **HDFS** (Hadoop Distributed File System). Finally, the processed data is exposed through **Hive external tables** for querying and further analysis.

Here’s a detailed breakdown of the assignment:

---

## **1. Objective**
The assignment requires analyzing marketing campaign data to answer specific business questions. The data includes:
- **Ad Campaign Data**: Information about ad campaigns, including events like impressions, clicks, and video ads.
- **User Profile Data**: Demographic information about users, such as gender, age group, and categories.
- **Store Data**: Information about stores and their associated place IDs.

The tasks involve:
1. Loading the data into **HDFS**.
2. Performing transformations and aggregations using **PySpark**.
3. Storing the results in **HDFS**.
4. Creating **Hive external tables** on top of the processed data for querying.

---

## **2. Tools and Technologies Used**
- **Python3**: For writing PySpark code.
- **Apache Spark**: For distributed data processing.
- **GCP (Google Cloud Platform)**: Specifically, **Dataproc** (managed Spark and Hadoop service) and **GCS** (Google Cloud Storage).
- **Jupyter Lab**: For interactive development and running PySpark code.
- **Apache Hive**: For creating external tables and querying the processed data.

---

## **3. Data Description**
The data is provided in **JSON format** and consists of three files:
1. **`ad_campaigns_data.json`**:
   - Contains information about ad campaigns, including:
     - `campaign_id`: Unique identifier for the campaign.
     - `campaign_name`: Name of the campaign.
     - `campaign_country`: Country where the campaign is running.
     - `os_type`: Operating system type (e.g., iOS, Android).
     - `device_type`: Type of device (e.g., Apple, Samsung).
     - `place_id`: Identifier for the location (store).
     - `user_id`: Identifier for the user.
     - `event_type`: Type of event (e.g., impression, click, video ad).
     - `event_time`: Timestamp of the event.

2. **`user_profile_data.json`**:
   - Contains user demographic information, including:
     - `user_id`: Unique identifier for the user.
     - `country`: Country of the user.
     - `gender`: Gender of the user.
     - `age_group`: Age group of the user.
     - `category`: Categories the user belongs to (e.g., shopper, student).

3. **`store_data.json`**:
   - Contains store information, including:
     - `store_name`: Name of the store.
     - `place_ids`: List of place IDs associated with the store.

---

## **4. Steps in the Assignment**

### **Step 1: Load Data into HDFS**
- The JSON files (`ad_campaigns_data.json`, `user_profile_data.json`, and `store_data.json`) are loaded into **HDFS**.
- PySpark is used to read these files into DataFrames.

### **Step 2: Perform Transformations and Aggregations**
- The following analytical tasks are performed using PySpark:

#### **Question 1: Analyze Data by Campaign, Date, Hour, and OS Type**
- **Objective**: For each `campaign_id`, `date`, `hour`, and `os_type`, calculate the count of each event type (`impression`, `click`, `video ad`).
- **Output Format**:
  ```json
  {
    "campaign_id": "ABCDFAE",
    "date": "2018-10-12",
    "hour": "13",
    "os_type": "android",
    "event": {
      "impression": 2,
      "click": 1,
      "video ad": 1
    }
  }
  ```

#### **Question 2: Analyze Data by Campaign, Date, Hour, and Store Name**
- **Objective**: For each `campaign_id`, `date`, `hour`, and `store_name`, calculate the count of each event type (`impression`, `click`, `video ad`).
- **Output Format**:
  ```json
  {
    "campaign_id": "ABCDFAE",
    "date": "2018-10-12",
    "hour": "13",
    "store_name": "McDonald",
    "event": {
      "impression": 2,
      "click": 1,
      "video ad": 1
    }
  }
  ```

#### **Question 3: Analyze Data by Campaign, Date, Hour, and Gender**
- **Objective**: For each `campaign_id`, `date`, `hour`, and `gender`, calculate the count of each event type (`impression`, `click`, `video ad`).
- **Output Format**:
  ```json
  {
    "campaign_id": "ABCDFAE",
    "date": "2018-10-12",
    "hour": "13",
    "gender": "male",
    "event": {
      "impression": 2,
      "click": 1,
      "video ad": 1
    }
  }
  ```

### **Step 3: Store Processed Data in HDFS**
- The results of each analytical task are saved in separate directories in **HDFS**:
  - `q1_output`: For Question 1 results.
  - `q2_output`: For Question 2 results.
  - `q3_output`: For Question 3 results.

### **Step 4: Create Hive External Tables**
- **Hive external tables** are created on top of the processed data stored in HDFS.
- The tables are created using **JSON SerDe** (Serializer/Deserializer) to handle JSON data.
- Example HQL (Hive Query Language) commands:
  ```sql
  CREATE EXTERNAL TABLE IF NOT EXISTS q1_output (
      campaign_id STRING,
      date STRING,
      hour INT,
      os_type STRING,
      impression INT,
      click INT,
      video_ad INT
  )
  ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
  LOCATION '/tmp/marketing_data/output/q1_output';
  ```

### **Step 5: Query Hive Tables**
- The Hive tables are queried to verify that the data has been correctly processed and stored.
- Example queries:
  ```sql
  SELECT * FROM q1_output;
  SELECT * FROM q2_output;
  SELECT * FROM q3_output;
  ```

---

## **5. Key PySpark Operations**
The following PySpark operations are used in the assignment:
1. **Reading JSON Data**:
   - `spark.read.json()` is used to load JSON files into DataFrames.
2. **Schema Definition**:
   - Explicit schemas are defined using `StructType` and `StructField` to ensure proper data types.
3. **Data Transformation**:
   - `withColumn()`: Used to add new columns (e.g., extracting `date` and `hour` from `event_time`).
   - `groupBy()`: Used to group data by specific columns.
   - `agg()`: Used to perform aggregations (e.g., counting events).
   - `pivot()`: Used to pivot data based on event types.
4. **Joins**:
   - `join()`: Used to join DataFrames (e.g., joining `ad_campaigns_data` with `store_data` or `user_profile_data`).
5. **Writing Data**:
   - `write.json()`: Used to save the results in JSON format to HDFS.

---

## **6. Expected Output**
The output for each question is in **JSON format** and includes:
- **Campaign ID**: Identifier for the campaign.
- **Date**: Date of the event.
- **Hour**: Hour of the event.
- **Type**: The category being analyzed (e.g., `os_type`, `store_name`, `gender`).
- **Value**: The specific value of the category (e.g., `android`, `McDonald`, `male`).
- **Event Counts**: The number of `impressions`, `clicks`, and `video ads` for the given combination.

---

## **7. Key Takeaways**
- The assignment demonstrates how to use **PySpark** for large-scale data processing and analysis.
- It highlights the importance of **schema definition** and **data transformation** in distributed computing.
- The integration of **Hive** with **HDFS** allows for efficient querying of processed data.
- The modular approach ensures that the code is reusable and maintainable.

---

## **8. Challenges and Considerations**
- **Data Volume**: The datasets are large (hundreds of GBs per day), so efficient processing and optimization are critical.
- **Data Quality**: Handling missing or malformed data is important to ensure accurate results.
- **Performance**: Proper partitioning and caching can improve the performance of PySpark jobs.
- **Scalability**: The solution must scale to handle increasing data volumes over time.

---

This assignment is a comprehensive example of how to use **PySpark** and **Hive** for real-world data analysis tasks in a distributed environment. It covers data ingestion, transformation, aggregation, and storage, providing a complete end-to-end solution for marketing campaign analysis.


<br/>
<br/>

## **Question 1: Analyze Data by Campaign, Date, Hour, and OS Type**

### **Objective**
For each `campaign_id`, `date`, `hour`, and `os_type`, calculate the count of each event type (`impression`, `click`, `video ad`).

### **Solution Steps**
1. **Extract Date and Hour**:
   - The `event_time` column is cast to a timestamp, and new columns (`date` and `hour`) are extracted from it.
   ```python
   df_campaigns = df_campaigns.withColumn("event_time", F.col("event_time").cast("timestamp"))
   df_campaigns = df_campaigns.withColumn("date", F.to_date("event_time"))
   df_campaigns = df_campaigns.withColumn("hour", F.hour("event_time"))
   ```

2. **Group and Aggregate Data**:
   - The data is grouped by `campaign_id`, `date`, `hour`, and `os_type`.
   - The count of each `event_type` is calculated using `groupBy` and `agg`.
   - The `pivot` function is used to transform the `event_type` values into columns (`impression`, `click`, `video ad`).
   ```python
   result_q1 = (
       df_campaigns.groupBy("campaign_id", "date", "hour", "os_type", "event_type")
       .agg(F.count("event_type").alias("event_count"))
       .groupBy("campaign_id", "date", "hour", "os_type")
       .pivot("event_type")
       .agg(F.first("event_count"))
       .fillna(0)
       .select(
           "campaign_id",
           "date",
           "hour",
           "os_type",
           F.struct(
               F.col("impression").alias("impression"),
               F.col("click").alias("click"),
               F.col("video ad").alias("video_ad"),
           ).alias("event"),
       )
   )
   ```

3. **Save the Result**:
   - The result is saved in JSON format to HDFS.
   ```python
   result_q1.write.json(hdfs_output_path1 + "q1_output", mode="overwrite")
   ```

### **Output Example**
```json
{
  "campaign_id": "ABCDFAE",
  "date": "2018-10-12",
  "hour": "13",
  "os_type": "android",
  "event": {
    "impression": 2,
    "click": 1,
    "video ad": 1
  }
}
```

---

## **Question 2: Analyze Data by Campaign, Date, Hour, and Store Name**

### **Objective**
For each `campaign_id`, `date`, `hour`, and `store_name`, calculate the count of each event type (`impression`, `click`, `video ad`).

### **Solution Steps**
1. **Join Campaign Data with Store Data**:
   - The `ad_campaigns_data` DataFrame is joined with the `store_data` DataFrame using the `place_id` column.
   - The `array_contains` function is used to match `place_id` with the `place_ids` array in the `store_data`.
   ```python
   result_q2 = (
       df_campaigns.join(df_stores, F.array_contains(df_stores.place_ids, df_campaigns.place_id), "inner")
   ```

2. **Group and Aggregate Data**:
   - The data is grouped by `campaign_id`, `date`, `hour`, and `store_name`.
   - The count of each `event_type` is calculated using `groupBy` and `agg`.
   - The `pivot` function is used to transform the `event_type` values into columns (`impression`, `click`, `video ad`).
   ```python
       .groupBy("campaign_id", "date", "hour", "store_name", "event_type")
       .agg(F.count("event_type").alias("event_count"))
       .groupBy("campaign_id", "date", "hour", "store_name")
       .pivot("event_type")
       .agg(F.first("event_count"))
       .fillna(0)
       .select(
           "campaign_id",
           "date",
           "hour",
           "store_name",
           F.struct(
               F.col("impression").alias("impression"),
               F.col("click").alias("click"),
               F.col("video ad").alias("video_ad"),
           ).alias("event"),
       )
   )
   ```

3. **Save the Result**:
   - The result is saved in JSON format to HDFS.
   ```python
   result_q2.write.json(hdfs_output_path2 + "q2_output", mode="overwrite")
   ```

### **Output Example**
```json
{
  "campaign_id": "ABCDFAE",
  "date": "2018-10-12",
  "hour": "13",
  "store_name": "McDonald",
  "event": {
    "impression": 2,
    "click": 1,
    "video ad": 1
  }
}
```

---

## **Question 3: Analyze Data by Campaign, Date, Hour, and Gender**

### **Objective**
For each `campaign_id`, `date`, `hour`, and `gender`, calculate the count of each event type (`impression`, `click`, `video ad`).

### **Solution Steps**
1. **Join Campaign Data with User Data**:
   - The `ad_campaigns_data` DataFrame is joined with the `user_profile_data` DataFrame using the `user_id` column.
   ```python
   result_q3 = (
       df_campaigns.join(df_users, "user_id", "inner")
   ```

2. **Group and Aggregate Data**:
   - The data is grouped by `campaign_id`, `date`, `hour`, and `gender`.
   - The count of each `event_type` is calculated using `groupBy` and `agg`.
   - The `pivot` function is used to transform the `event_type` values into columns (`impression`, `click`, `video ad`).
   ```python
       .groupBy("campaign_id", "date", "hour", "gender", "event_type")
       .agg(F.count("event_type").alias("event_count"))
       .groupBy("campaign_id", "date", "hour", "gender")
       .pivot("event_type")
       .agg(F.first("event_count"))
       .fillna(0)
       .select(
           "campaign_id",
           "date",
           "hour",
           "gender",
           F.struct(
               F.col("impression").alias("impression"),
               F.col("click").alias("click"),
               F.col("video ad").alias("video_ad"),
           ).alias("event"),
       )
   )
   ```

3. **Save the Result**:
   - The result is saved in JSON format to HDFS.
   ```python
   result_q3.write.json(hdfs_output_path3 + "q3_output", mode="overwrite")
   ```

### **Output Example**
```json
{
  "campaign_id": "ABCDFAE",
  "date": "2018-10-12",
  "hour": "13",
  "gender": "male",
  "event": {
    "impression": 2,
    "click": 1,
    "video ad": 1
  }
}
```

---

## **Key Observations**
1. **Modular Code**:
   - The code is modular, with each question's solution following a similar pattern:
     - Extract necessary columns (e.g., `date`, `hour`).
     - Join DataFrames if required.
     - Group and aggregate data.
     - Pivot the data to transform event types into columns.
     - Save the result to HDFS.

2. **Use of PySpark Functions**:
   - Functions like `withColumn`, `groupBy`, `agg`, `pivot`, and `struct` are used extensively for data transformation and aggregation.

3. **Output Format**:
   - The output is structured in a consistent JSON format, making it easy to query and analyze using Hive.

4. **Scalability**:
   - The solution is designed to handle large-scale data, as evidenced by the use of distributed processing with PySpark and storage in HDFS.

---

## **Conclusion**
The assignment demonstrates a complete workflow for analyzing marketing campaign data using **PySpark** and **Hive**. Each question's solution involves:
- **Data Ingestion**: Loading JSON data into DataFrames.
- **Data Transformation**: Extracting, joining, and aggregating data.
- **Data Storage**: Saving results in HDFS and creating Hive external tables for querying.

This approach ensures that the solution is scalable, efficient, and suitable for real-world big data scenarios.