# **Healthcare Data Analysis with Spark and Cassandra**

This assignment involves building a **Spark application** to process healthcare data from daily CSV files, perform transformations, and load the transformed data into **Cassandra** tables. The goal is to analyze healthcare data incrementally, ensuring that the system can handle daily data loads, perform necessary transformations, and store the results in a structured manner in Cassandra. Below is a detailed explanation of the assignment:

---

### **Objective**
The primary objective is to create a **Spark application** that:
1. Processes daily CSV files containing healthcare data.
2. Performs data validation and cleaning.
3. Executes specific transformations on the data.
4. Stores the transformed data in **Cassandra** tables.
5. Archives the processed files.
6. Handles backfilling for failed days.

---

### **Key Steps in the Assignment**

#### **1. Data Ingestion**
- The Spark application reads daily CSV files from a **HDFS folder** (or in this case, a **Google Cloud Storage (GCS) bucket**).
- The files are named in the format: `health_data_<YYYYMMDD>.csv`, where `<YYYYMMDD>` represents the date.
- The CSV files contain the following columns:
  - `patient_id`
  - `age`
  - `gender`
  - `diagnosis_code`
  - `diagnosis_description`
  - `diagnosis_date`

#### **2. Data Validation and Cleaning**
- The application performs **data validation** to ensure:
  - All mandatory columns are present.
  - The data is in the correct format (e.g., `patient_id` is unique, `age` is numeric, etc.).
  - Handles missing or inconsistent data (e.g., null values, incorrect data types).

#### **3. Data Transformations**
The application performs the following transformations on the data:

##### **a. Disease Gender Ratio**
- For each disease (`diagnosis_code`), calculate the **gender ratio** (ratio of male to female patients).
- This helps identify if a particular disease is more prevalent in a specific gender.

##### **b. Most Common Diseases**
- Identify the **top 3 most common diseases** in the dataset.
- This helps in understanding the most prevalent diseases in the dataset.

##### **c. Age Category**
- Divide patients into **age categories** (e.g., '30-40', '41-50', '51-60', '61+').
- Calculate the number of patients in each age category for each disease.
- This helps in understanding the **age distribution** of different diseases.

##### **d. Flag for Senior Patients**
- Flag patients who are **senior citizens** (age >= 60).
- This information is useful for healthcare providers to prioritize care for senior patients.

##### **e. Disease Trend Over the Week**
- If more than a week's data is available, calculate the number of cases for each disease per day of the week.
- This helps in identifying trends (e.g., more cases on certain days).

#### **4. Data Loading into Cassandra**
- The transformed data is loaded into **Cassandra tables** in **upsert mode** (update if exists, insert if new).
- Separate tables are created for each type of transformation:
  - **Stage tables**: Used for raw data before transformation.
  - **Target tables**: Used for storing the transformed data.
- Appropriate **keys** are selected for each table to ensure efficient upsert operations.

#### **5. Archiving Processed Files**
- After successful processing, the input CSV file is moved to an **archive folder** in HDFS (or GCS).
- This ensures that the same file is not processed again.

#### **6. Data Backfilling**
- The system should handle **backfilling** for any failed days.
- If a day's processing fails, the system should be able to reprocess that day's data along with the current day's data.

---

### **Implementation Details**

#### **1. Tools and Technologies Used**
- **Python3**: Used for scripting and data manipulation.
- **Databricks**: Used as the Spark environment for running the application.
- **DataStax Astra**: A cloud-based Cassandra database used for storing the transformed data.
- **Google Cloud Storage (GCS)**: Used for storing input and archive files.

#### **2. Data Ingestion and Validation**
- The Spark application reads the CSV files from the GCS bucket.
- Data validation is performed using Spark DataFrame operations:
  - Check for null values in mandatory columns.
  - Ensure data types are correct (e.g., `age` is numeric, `diagnosis_date` is a valid date).

#### **3. Transformations**
- **Disease Gender Ratio**: The gender ratio is calculated using Spark's `groupBy` and `pivot` operations.
- **Most Common Diseases**: The top 3 diseases are identified using Spark SQL's `ROW_NUMBER()` function.
- **Age Category**: Age categories are created using Spark SQL's `CASE` statements.
- **Flag for Senior Patients**: A new column (`senior_citizen_flag`) is added to flag senior citizens.
- **Disease Trend Over the Week**: If more than a week's data is available, the number of cases per day is calculated.

#### **4. Data Loading into Cassandra**
- The application connects to **Cassandra** using the **DataStax Astra** secure connect bundle and token.
- Data is loaded into Cassandra using **CQL (Cassandra Query Language)**.
- The application checks if the table exists in Cassandra. If it does, the table is truncated before loading new data. If it doesn't, a new table is created.

#### **5. Archiving Files**
- After processing, the input CSV file is moved to the archive folder using Databricks' `dbutils.fs.mv` function.

#### **6. Workflow and Notifications**
- A **workflow** is created in Databricks to run the Spark application daily.
- The workflow consists of two jobs:
  - **Stage Process**: Processes the raw data and loads it into stage tables.
  - **Target Process**: Loads data from stage tables into target tables in upsert mode.
- Notifications are set up to alert in case of job failures.

---

### **Challenges Faced**
1. **Cassandra Setup**:
   - Cassandra cannot run locally; it requires a cloud-based setup (e.g., DataStax Astra).
   - Setting up the connection between Databricks and Cassandra was challenging, especially with the Cassandra-Spark connector.

2. **Data Loading**:
   - The Cassandra-Spark connector did not work as expected, so the data had to be loaded row by row using `session.execute`.

3. **Backfilling**:
   - Implementing backfilling for failed days required careful handling of file names and processing logic.

---

### **Conclusion**
This assignment demonstrates a complete **data pipeline** for healthcare data analysis using **Spark** and **Cassandra**. It involves data ingestion, validation, transformation, and loading into a NoSQL database. The system is designed to handle daily incremental loads, backfilling, and archiving, making it a robust solution for healthcare data analysis.

<br/>
<br/>

# **Flow of Project**

The **Healthcare Data Analysis** project is a **data pipeline** that processes daily healthcare data, performs transformations, and stores the results in **Cassandra**. Below is a detailed explanation of the **flow of the project**, including how the different components (scripts, notebooks, and tools) work together:

---

### **1. Data Generation**
- **Script**: `mock_data_generator.py`
- **Purpose**: Generates mock healthcare data for multiple days.
- **Flow**:
  1. The script generates **100 patient records per day** for a predefined list of dates (e.g., `2023-08-01`, `2023-08-02`, etc.).
  2. Each record contains fields such as `patient_id`, `age`, `gender`, `diagnosis_code`, `diagnosis_description`, and `diagnosis_date`.
  3. The data is saved in CSV files named in the format: `health_data_<YYYYMMDD>.csv` (e.g., `health_data_20230801.csv`).
  4. These CSV files are stored in a local directory or uploaded to a **Google Cloud Storage (GCS)** bucket for further processing.

---

### **2. Data Ingestion**
- **Notebook**: `stage_healthcare_analysis.ipynb`
- **Purpose**: Reads the daily CSV files from GCS and performs data validation and transformations.
- **Flow**:
  1. The notebook initializes a **Spark session** and configures it to connect to **GCS** and **Cassandra**.
  2. It reads the CSV files from the GCS bucket using Spark's `read.csv` function.
  3. The data is loaded into a **Spark DataFrame** for further processing.

---

### **3. Data Validation**
- **Notebook**: `stage_healthcare_analysis.ipynb`
- **Purpose**: Ensures the data is clean and consistent before performing transformations.
- **Flow**:
  1. The notebook checks for **null values** in each column of the DataFrame.
  2. It verifies the **data types** of each column (e.g., `age` should be numeric, `diagnosis_date` should be a valid date).
  3. If any issues are found, the notebook handles them appropriately (e.g., filling null values, correcting data types).

---

### **4. Data Transformations**
- **Notebook**: `stage_healthcare_analysis.ipynb`
- **Purpose**: Performs specific transformations on the data to derive insights.
- **Flow**:
  The notebook performs the following transformations:

  #### **a. Disease Gender Ratio**
  - Groups the data by `diagnosis_code` and `gender`.
  - Calculates the **gender ratio** (ratio of male to female patients) for each disease.
  - Stores the results in a new DataFrame.

  #### **b. Most Common Diseases**
  - Identifies the **top 3 most common diseases** in the dataset using Spark SQL's `ROW_NUMBER()` function.
  - Stores the results in a new DataFrame.

  #### **c. Age Category**
  - Divides patients into **age categories** (e.g., '30-40', '41-50', '51-60', '61+').
  - Calculates the number of patients in each age category for each disease.
  - Stores the results in a new DataFrame.

  #### **d. Flag for Senior Patients**
  - Flags patients who are **senior citizens** (age >= 60).
  - Adds a new column (`senior_citizen_flag`) to the DataFrame to indicate whether a patient is a senior citizen.

---

### **5. Data Loading into Cassandra**
- **Notebook**: `stage_healthcare_analysis.ipynb`
- **Purpose**: Loads the transformed data into **Cassandra** tables.
- **Flow**:
  1. The notebook connects to **Cassandra** using the **DataStax Astra** secure connect bundle and token.
  2. It checks if the table exists in Cassandra. If it does, the table is truncated before loading new data. If it doesn't, a new table is created.
  3. The transformed data is loaded into Cassandra using **CQL (Cassandra Query Language)**.
  4. Separate tables are created for each type of transformation:
     - **Stage tables**: Used for raw data before transformation.
     - **Target tables**: Used for storing the transformed data.

---

### **6. Archiving Processed Files**
- **Notebook**: `stage_healthcare_analysis.ipynb`
- **Purpose**: Moves the processed CSV files to an **archive folder** in GCS.
- **Flow**:
  1. After processing, the notebook moves the input CSV file to the **archive folder** using Databricks' `dbutils.fs.mv` function.
  2. This ensures that the same file is not processed again.

---

### **7. Target Table Processing**
- **Notebook**: `target_healthcare_analysis.ipynb` (not provided in the files, but mentioned in the documentation)
- **Purpose**: Loads data from stage tables into target tables in **upsert mode**.
- **Flow**:
  1. The notebook connects to **Cassandra** and checks if the target table exists.
  2. If the target table exists, it performs an **upsert operation** (update if exists, insert if new).
  3. If the target table does not exist, it creates a new table and loads the data from the stage table.

---

### **8. Workflow and Notifications**
- **Tool**: Databricks Workflows
- **Purpose**: Automates the execution of the Spark jobs and handles dependencies between them.
- **Flow**:
  1. A **workflow** is created in Databricks to run the Spark jobs daily.
  2. The workflow consists of two jobs:
     - **Stage Process**: Processes the raw data and loads it into stage tables.
     - **Target Process**: Loads data from stage tables into target tables in upsert mode.
  3. The second job is triggered only when the first job (stage process) is completed successfully.
  4. Notifications are set up to alert in case of job failures.

---

### **9. Data Backfilling**
- **Purpose**: Handles failed days by reprocessing the data.
- **Flow**:
  1. If a day's processing fails, the system should be able to reprocess that day's data along with the current day's data.
  2. This ensures that no data is lost and all days are processed correctly.

---

### **Summary of the Flow**
1. **Data Generation**: Mock data is generated using `mock_data_generator.py` and stored in CSV files.
2. **Data Ingestion**: The Spark application reads the CSV files from GCS.
3. **Data Validation**: The data is validated to ensure it is clean and consistent.
4. **Data Transformations**: The data is transformed to derive insights (e.g., gender ratio, top diseases, age categories).
5. **Data Loading**: The transformed data is loaded into Cassandra tables.
6. **Archiving**: The processed CSV files are moved to an archive folder.
7. **Target Table Processing**: Data is loaded from stage tables into target tables in upsert mode.
8. **Workflow Automation**: The entire process is automated using Databricks workflows.
9. **Backfilling**: Failed days are reprocessed to ensure data completeness.

---

### **Conclusion**
The **Healthcare Data Analysis** project is a **robust data pipeline** that processes daily healthcare data, performs transformations, and stores the results in **Cassandra**. The flow involves data generation, ingestion, validation, transformation, loading, archiving, and automation, making it a complete solution for healthcare data analysis. The use of **Spark** and **Cassandra** ensures scalability and efficiency, while **Databricks workflows** automate the process and handle dependencies.

<br/>
<br/>

# **`Mock_data_generator.py`**
The **`mock_data_generator.py`** script is a Python script used to generate **mock healthcare data** in the form of CSV files. These CSV files simulate daily healthcare data, which is then used as input for the **Spark application** in the **Healthcare Data Analysis** assignment. Below is a detailed explanation of the script:

---

### **Purpose**
The purpose of the `mock_data_generator.py` script is to:
1. Generate **mock healthcare data** for multiple days.
2. Create CSV files for each day, containing patient records with fields such as `patient_id`, `age`, `gender`, `diagnosis_code`, `diagnosis_description`, and `diagnosis_date`.
3. Simulate real-world healthcare data for testing and development purposes.

---

### **Key Features**
- The script generates **100 patient records per day**.
- It uses the **Faker** library to generate realistic data.
- The data is saved in CSV files named in the format: `health_data_<YYYYMMDD>.csv`, where `<YYYYMMDD>` represents the date.

---

### **Detailed Explanation of the Script**

#### **1. Importing Libraries**
The script imports the following libraries:
- **Pandas**: Used to create and manipulate DataFrames.
- **Random**: Used to generate random values (e.g., age, gender, diagnosis).
- **Faker**: Used to generate realistic fake data (e.g., patient IDs).

```python
import pandas as pd
import random
from faker import Faker
```

---

#### **2. Initializing Faker**
The script initializes the **Faker** library, which is used to generate realistic fake data.

```python
fake = Faker()
```

---

#### **3. Defining Constants**
The script defines several constants that are used to generate the mock data:
- **Days**: A list of dates for which the data will be generated.
- **Diseases**: A list of tuples containing disease codes and descriptions.
- **Genders**: A list of possible gender values (`M` for male, `F` for female).

```python
# Definitions
days = ["2023-08-01", "2023-08-02", "2023-08-03", "2023-08-04", "2023-08-05"]
diseases = [("D123", "Diabetes"), ("H234", "High Blood Pressure"), ("C345", "Cancer")]
genders = ["M", "F"]
```

---

#### **4. Generating Mock Data**
The script generates mock data for each day in the `days` list. For each day, it creates **100 patient records** with the following fields:
- **patient_id**: A unique identifier for each patient, generated using Faker.
- **age**: A random integer between 30 and 70.
- **gender**: A random value from the `genders` list (`M` or `F`).
- **diagnosis_code**: A random disease code from the `diseases` list.
- **diagnosis_description**: The corresponding description for the selected disease code.
- **diagnosis_date**: The date for which the data is being generated.

```python
# For each day
for i, day in enumerate(days):
    # Create a list to hold data
    data = []
    # Create 100 records for each day
    for j in range(1, 101):
        patient_id = f'P{i*100 + j}'
        age = random.randint(30, 70)
        gender = random.choice(genders)
        diagnosis_code, diagnosis_description = random.choice(diseases)
        diagnosis_date = day
        # Append the row to the data list
        data.append([patient_id, age, gender, diagnosis_code, diagnosis_description, diagnosis_date])
```

---

#### **5. Saving Data to CSV Files**
After generating the data for each day, the script creates a **Pandas DataFrame** and saves it as a CSV file. The CSV files are named in the format: `health_data_<YYYYMMDD>.csv`, where `<YYYYMMDD>` is the date in `YYYYMMDD` format (e.g., `health_data_20230801.csv`).

```python
    # Create a DataFrame and write it to CSV
    df = pd.DataFrame(data, columns=["patient_id", "age", "gender", "diagnosis_code", "diagnosis_description", "diagnosis_date"])
    df.to_csv(f'health_data_{day.replace("-", "")}.csv', index=False)
```

---

### **Example Output**
For each day, the script generates a CSV file with 100 rows of data. Below is an example of the data generated for one day:

| patient_id | age | gender | diagnosis_code | diagnosis_description | diagnosis_date |
|------------|-----|--------|----------------|-----------------------|----------------|
| P1         | 45  | M      | H234           | High Blood Pressure   | 2023-08-01     |
| P2         | 32  | F      | D123           | Diabetes              | 2023-08-01     |
| P3         | 39  | F      | H234           | High Blood Pressure   | 2023-08-01     |
| ...        | ... | ...    | ...            | ...                   | ...            |
| P100       | 55  | F      | D123           | Diabetes              | 2023-08-01     |

---

### **How to Use the Script**
1. **Install Required Libraries**:
   - Ensure that **Pandas** and **Faker** are installed. You can install them using pip:
     ```bash
     pip install pandas faker
     ```

2. **Run the Script**:
   - Execute the script using Python:
     ```bash
     python mock_data_generator.py
     ```

3. **Generated Files**:
   - The script will generate CSV files for each day in the `days` list. For example:
     - `health_data_20230801.csv`
     - `health_data_20230802.csv`
     - `health_data_20230803.csv`
     - `health_data_20230804.csv`
     - `health_data_20230805.csv`

4. **Use in Spark Application**:
   - These CSV files can be used as input for the **Spark application** in the **Healthcare Data Analysis** assignment.

---

### **Conclusion**
The **`mock_data_generator.py`** script is a simple yet powerful tool for generating **mock healthcare data**. It simulates real-world patient records, making it ideal for testing and development purposes. The generated CSV files can be used as input for the **Spark application**, allowing you to test the data processing pipeline without needing real data.

<br/>
<br/>

# **stage_healthcare_analysis.ipynb**
The **`stage_healthcare_analysis.ipynb`** file is a **Databricks notebook** written in **Python** using **PySpark**. It is part of the **Healthcare Data Analysis** assignment and is responsible for processing daily healthcare data, performing transformations, and loading the results into **Cassandra** tables. Below is a detailed explanation of the notebook:

---

### **Overview**
The notebook performs the following tasks:
1. **Initializes a Spark session** and configures it to connect to **Google Cloud Storage (GCS)** and **Cassandra**.
2. **Reads daily CSV files** from a GCS bucket.
3. **Performs data validation** to ensure the data is clean and consistent.
4. **Executes transformations** on the data, such as calculating disease gender ratios, identifying top diseases, categorizing patients by age, and flagging senior citizens.
5. **Loads the transformed data** into **Cassandra** tables.
6. **Archives the processed files** to a different folder in GCS.

---

### **Detailed Explanation of the Notebook**

#### **1. Initialization and Configuration**
- The notebook starts by importing necessary libraries, including **PySpark**, **Cassandra**, and **Pandas**.
- A **Spark session** is initialized with configurations for connecting to **GCS** and **Cassandra**.
  - **GCS Configuration**: The notebook uses a service account JSON key to authenticate with GCS.
  - **Cassandra Configuration**: The notebook connects to **DataStax Astra** (a cloud-based Cassandra service) using a secure connect bundle and token.

```python
# Initialize Spark Session
spark = SparkSession.builder \
    .appName("Data Ingestion") \
    .config("fs.gs.impl", "com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem") \
    .config("fs.AbstractFileSystem.gs.impl", "com.google.cloud.hadoop.fs.gcs.GoogleHadoopFS") \
    .config("spark.cassandra.connection.host", "aa93594f-b5ab-4912-994e-6156fb347536-us-east1.db.astra.datastax.com") \
    .config("spark.cassandra.connection.port", "29042") \
    .config("spark.cassandra.connection.ssl.enabled", "true") \
    .config("spark.cassandra.connection.ssl.trustStore.path", "./trustStore.jks") \
    .config("spark.cassandra.connection.ssl.trustStore.password", "0ub5Xk2C3Pzqjhy1m") \
    .config("spark.cassandra.connection.ssl.clientAuth.enabled", "true") \
    .config("spark.cassandra.connection.ssl.keyStore.path", "./identity.jks") \
    .config("spark.cassandra.connection.ssl.keyStore.password", "G1BrN3bZO47AQMI6X") \
    .getOrCreate()
```

- The notebook also sets up the **GCS bucket** and **directories** for input and archive files.

```python
# GCS bucket details
bucket_name = "healthcare_analysis"
data_directory = f"gs://{bucket_name}/input/"
archive_directory = f"gs://{bucket_name}/archive/"
```

---

#### **2. Data Ingestion**
- The notebook reads all CSV files from the specified GCS directory using Spark's `read.csv` function.
- The data is loaded into a **Spark DataFrame** for further processing.

```python
# Read all CSV files from the specified GCS directory
df = spark.read.csv(data_directory, inferSchema=True, header=True)
df.show()
```

---

#### **3. Data Validation**
- The notebook performs **data validation** to ensure the data is clean and consistent.
- It checks for **null values** in each column and verifies the **data types** of each column.

```python
# Check for null values in each column
null_counts = df.agg(
    *[sum(col(column).isNull().cast("int")).alias(f"{column}_null_count") for column in df.columns]
)

# Check for data types
data_type_checks = [col(column).cast("string").alias(f"{column}_type_check") for column in df.columns]
df_check = df.select(data_type_checks)

# Show the results of the checks
print("Null Counts:")
null_counts.show()

print("Data Type Checks:")
df_check.show()
```

---

#### **4. Data Transformations**
The notebook performs several transformations on the data:

##### **a. Disease Gender Ratio**
- The notebook calculates the **gender ratio** for each disease by grouping the data by `diagnosis_code` and `gender`.
- It uses the `pivot` function to create separate columns for male and female counts and then calculates the ratio.

```python
# Group by diagnosis_code and gender, and calculate the count for each group
gender_counts = df.groupBy("diagnosis_code", "diagnosis_description", "gender").agg(count("patient_id").alias("counter"))

# Pivot the data to get Male and Female counts as separate columns
gender_pivoted = gender_counts.groupBy("diagnosis_code", "diagnosis_description").pivot("gender").agg(
    coalesce(sum(when(col("gender") == "M", col("counter"))), lit(0)).alias("Males"),
    coalesce(sum(when(col("gender") == "F", col("counter"))), lit(0)).alias("Females")
)

# Calculate the gender ratio
gender_ratio = gender_pivoted.withColumn("Gender_Ratio", col("M_Males") / col("F_Females"))
gender_ratio.show()
```

##### **b. Most Common Diseases**
- The notebook identifies the **top 3 most common diseases** using Spark SQL.
- It uses the `ROW_NUMBER()` function to rank the diseases based on their occurrence.

```python
# Use SQL to find the top 3 common diseases
top3_query = """
    SELECT
        ROW_NUMBER() OVER (ORDER BY count(*) DESC) AS Rank,
        diagnosis_code,
        diagnosis_description
    FROM
        patient_data
    GROUP BY
        diagnosis_code, diagnosis_description
    ORDER BY
        Rank
    LIMIT 3
"""
top3 = spark.sql(top3_query)
top3.show()
```

##### **c. Age Category**
- The notebook categorizes patients into **age groups** (e.g., '30-40', '41-50', '51-60', '61+') using Spark SQL's `CASE` statements.
- It calculates the number of patients in each age category for each disease.

```python
# Use SQL to create age buckets
distro_query = """
    SELECT
        diagnosis_code,
        diagnosis_description,
        CASE
            WHEN age >= 30 AND age < 40 THEN '30-40'
            WHEN age >= 40 AND age < 50 THEN '41-50'
            WHEN age >= 50 AND age < 60 THEN '51-60'
            WHEN age >= 60 THEN '61+'
        END AS age_category,
        COUNT(patient_id) AS patient_count,
        CONCAT(diagnosis_code, '_', 
            CASE
               WHEN age >= 30 AND age < 40 THEN '30-40'
               WHEN age >= 40 AND age < 50 THEN '41-50'
               WHEN age >= 50 AND age < 60 THEN '51-60'
               WHEN age >= 60 THEN '61+'
            END) AS code_age_category
    FROM
        patient_data
    GROUP BY
        diagnosis_code, diagnosis_description, age_category
"""
distro = spark.sql(distro_query)
distro.show()
```

##### **d. Flag for Senior Patients**
- The notebook flags patients who are **senior citizens** (age >= 60) using Spark SQL.

```python
# Use SQL to flag senior citizens
senior_query = """
    SELECT
        patient_id, age, 
        CASE WHEN age >= 60 THEN 'Y' ELSE 'N' END AS senior_citizen_flag
    FROM
        patient_data
"""
senior = spark.sql(senior_query)
senior.show()
```

---

#### **5. Data Loading into Cassandra**
- The notebook connects to **Cassandra** using the **DataStax Astra** secure connect bundle and token.
- It checks if the table exists in Cassandra. If it does, the table is truncated before loading new data. If it doesn't, a new table is created.
- The transformed data is loaded into Cassandra using **CQL (Cassandra Query Language)**.

```python
# Check if the table exists
existing_table_query = f"SELECT table_name FROM system_schema.tables WHERE keyspace_name = '{keyspace}' AND table_name = '{table}'"
existing_table_result = session.execute(existing_table_query)

if existing_table_result.one():
    # Table exists, truncate (delete all data)
    truncate_query = f"TRUNCATE TABLE {keyspace}.{table}"
    session.execute(truncate_query)
else:
    # Table does not exist, create it
    create_table_query = f"""
    CREATE TABLE IF NOT EXISTS healthcare.stage_disease_ratio (
        diagnosis_code TEXT PRIMARY KEY,
        diagnosis_description TEXT,
        F_Females INT,
        M_Males INT,
        Gender_Ratio DOUBLE
    )
    """
    session.execute(create_table_query)

# Convert Spark DataFrame to Pandas DataFrame
pandas_df = gender_ratio.toPandas()

# Insert data into Cassandra table
for index, row in pandas_df.iterrows():
    insert_query = f"""
    INSERT INTO healthcare.stage_disease_ratio
    (diagnosis_code, diagnosis_description, F_Females, M_Males, Gender_Ratio)
    VALUES ('{row['diagnosis_code']}', '{row['diagnosis_description']}', 
            {row['F_Females']}, {row['M_Males']}, {row['Gender_Ratio']})
    """
    session.execute(insert_query)
```

---

#### **6. Archiving Processed Files**
- After processing, the input CSV file is moved to the **archive folder** in GCS using Databricks' `dbutils.fs.mv` function.

```python
# List and move files individually
file_list = dbutils.fs.ls(data_directory)
for file in file_list:
    if file.name.endswith(".csv"):
        print(f"{file} Moved in archive folder")
        dbutils.fs.mv(file.path, os.path.join(archive_directory, file.name))
```

---

### **Conclusion**
The **`stage_healthcare_analysis.ipynb`** notebook is a **PySpark-based data pipeline** that processes daily healthcare data, performs transformations, and loads the results into **Cassandra** tables. It handles data validation, transformation, and archiving, making it a robust solution for healthcare data analysis. The notebook is designed to run daily as part of a larger workflow, ensuring that the data is processed incrementally and stored efficiently in Cassandra.