# **Detailed Explanation of Hive Assignment 2: Data Analysis with Apache Hive on Telecom Data**

This assignment involves analyzing a telecom dataset to understand customer churn and other related metrics. The dataset contains information about customers, such as their demographics, services subscribed, contract details, and whether they have churned (left the service). The assignment is divided into several problems, each focusing on different aspects of Hive, such as **data loading**, **data exploration**, **data analysis**, **partitioning**, **bucketing**, **join optimizations**, and **advanced analysis**.

Below is a detailed explanation of each problem, along with the corresponding HiveQL queries and their results.

---

### **Problem 1: Data Loading**

#### **1.1 Create a Hive Table and Load Data**
- **Objective**: Create a Hive table and load the telecom dataset into it.
- **Query**:
  ```sql
  CREATE TABLE telecom_data (
      customerID STRING,
      gender STRING,
      SeniorCitizen INT,
      Partner STRING,
      Dependents STRING,
      tenure INT,
      PhoneService STRING,
      MultipleLines STRING,
      InternetService STRING,
      OnlineSecurity STRING,
      OnlineBackup STRING,
      DeviceProtection STRING,
      TechSupport STRING,
      StreamingTV STRING,
      StreamingMovies STRING,
      Contract STRING,
      PaperlessBilling STRING,
      PaymentMethod STRING,
      MonthlyCharges FLOAT,
      TotalCharges FLOAT,
      Churn STRING
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE;
  ```
- **Explanation**: This query creates a table named `telecom_data` with the given schema. The data is stored as a text file, and the fields are delimited by commas.

#### **1.2 Display Top 10 Rows**
- **Objective**: Display the top 10 rows of the table.
- **Query**:
  ```sql
  SELECT * FROM telecom_data LIMIT 10;
  ```
- **Explanation**: This query retrieves the first 10 rows from the `telecom_data` table.

---

### **Problem 2: Data Exploration**

#### **2.1 Total Number of Customers**
- **Objective**: Find the total number of customers in the dataset.
- **Query**:
  ```sql
  SELECT COUNT(*) FROM telecom_data;
  ```
- **Explanation**: This query counts all rows in the `telecom_data` table.

#### **2.2 Number of Customers Who Have Churned**
- **Objective**: Find the number of customers who have churned.
- **Query**:
  ```sql
  SELECT COUNT(*) FROM telecom_data WHERE Churn = 'Yes';
  ```
- **Explanation**: This query counts the number of customers where the `Churn` column is `'Yes'`.

#### **2.3 Distribution of Customers by Gender and SeniorCitizen Status**
- **Objective**: Analyze the distribution of customers based on gender and SeniorCitizen status.
- **Query**:
  ```sql
  SELECT gender, SeniorCitizen, COUNT(*)
  FROM telecom_data
  GROUP BY gender, SeniorCitizen;
  ```
- **Explanation**: This query groups customers by `gender` and `SeniorCitizen` status and counts the number of customers in each group.

#### **2.4 Total Charge Due to Churned Customers**
- **Objective**: Determine the total charge to the company due to churned customers.
- **Query**:
  ```sql
  SELECT SUM(TotalCharges)
  FROM telecom_data
  WHERE Churn = 'Yes';
  ```
- **Explanation**: This query calculates the total charges for customers who have churned.

---

### **Problem 3: Data Analysis**

#### **3.1 Number of Churned Customers by Contract Type**
- **Objective**: Find the number of customers who have churned, grouped by their contract type.
- **Query**:
  ```sql
  SELECT Contract, COUNT(*)
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY Contract;
  ```
- **Explanation**: This query groups churned customers by their contract type and counts the number of customers in each group.

#### **3.2 Average Monthly Charges for Churned vs Non-Churned Customers**
- **Objective**: Find the average monthly charges for customers who have churned vs those who have not.
- **Query**:
  ```sql
  SELECT Churn, AVG(MonthlyCharges)
  FROM telecom_data
  GROUP BY Churn;
  ```
- **Explanation**: This query calculates the average monthly charges for customers who have churned and those who have not.

#### **3.3 Maximum, Minimum, and Average Tenure**
- **Objective**: Determine the maximum, minimum, and average tenure of the customers.
- **Query**:
  ```sql
  SELECT MAX(tenure), MIN(tenure), AVG(tenure)
  FROM telecom_data;
  ```
- **Explanation**: This query calculates the maximum, minimum, and average tenure of customers.

#### **3.4 Most Popular Payment Method**
- **Objective**: Find out which payment method is most popular among customers.
- **Query**:
  ```sql
  SELECT PaymentMethod, COUNT(*)
  FROM telecom_data
  GROUP BY PaymentMethod
  ORDER BY COUNT(*) DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the most popular payment method by counting the number of customers using each payment method.

#### **3.5 Relationship Between PaperlessBilling and Churn Rate**
- **Objective**: Analyze the relationship between paperless billing and churn rate.
- **Query**:
  ```sql
  SELECT PaperlessBilling, COUNT(*)
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY PaperlessBilling;
  ```
- **Explanation**: This query counts the number of churned customers who have paperless billing enabled or disabled.

---

### **Problem 4: Partitioning**

#### **4.1 Create a Partitioned Table by Contract**
- **Objective**: Create a partitioned table by `Contract` and load data from the original table.
- **Query**:
  ```sql
  CREATE TABLE telecom_data_partitioned (
      customerID STRING,
      gender STRING,
      SeniorCitizen INT,
      Partner STRING,
      Dependents STRING,
      tenure INT,
      PhoneService STRING,
      MultipleLines STRING,
      InternetService STRING,
      OnlineSecurity STRING,
      OnlineBackup STRING,
      DeviceProtection STRING,
      TechSupport STRING,
      StreamingTV STRING,
      StreamingMovies STRING,
      PaperlessBilling STRING,
      PaymentMethod STRING,
      MonthlyCharges FLOAT,
      TotalCharges FLOAT,
      Churn STRING
  )
  PARTITIONED BY (Contract STRING);

  INSERT OVERWRITE TABLE telecom_data_partitioned
  PARTITION(Contract)
  SELECT * FROM telecom_data;
  ```
- **Explanation**: This query creates a partitioned table by `Contract` and loads data from the original table.

#### **4.2 Number of Churned Customers by Contract Type Using Partitioned Table**
- **Objective**: Find the number of customers who have churned in each contract type using the partitioned table.
- **Query**:
  ```sql
  SELECT Contract, COUNT(*)
  FROM telecom_data_partitioned
  WHERE Churn = 'Yes'
  GROUP BY Contract;
  ```
- **Explanation**: This query counts the number of churned customers in each contract type using the partitioned table.

#### **4.3 Average Monthly Charges by Contract Type Using Partitioned Table**
- **Objective**: Find the average monthly charges for each type of contract using the partitioned table.
- **Query**:
  ```sql
  SELECT Contract, AVG(MonthlyCharges)
  FROM telecom_data_partitioned
  GROUP BY Contract;
  ```
- **Explanation**: This query calculates the average monthly charges for each contract type using the partitioned table.

#### **4.4 Maximum Tenure by Contract Type Using Partitioned Table**
- **Objective**: Determine the maximum tenure in each contract type partition.
- **Query**:
  ```sql
  SELECT Contract, MAX(tenure)
  FROM telecom_data_partitioned
  GROUP BY Contract;
  ```
- **Explanation**: This query finds the maximum tenure for each contract type using the partitioned table.

---

### **Problem 5: Bucketing**

#### **5.1 Create a Bucketed Table by Tenure**
- **Objective**: Create a bucketed table by `tenure` into 6 buckets.
- **Query**:
  ```sql
  CREATE TABLE telecom_data_bucketed (
      customerID STRING,
      gender STRING,
      SeniorCitizen INT,
      Partner STRING,
      Dependents STRING,
      PhoneService STRING,
      MultipleLines STRING,
      InternetService STRING,
      OnlineSecurity STRING,
      OnlineBackup STRING,
      DeviceProtection STRING,
      TechSupport STRING,
      StreamingTV STRING,
      StreamingMovies STRING,
      Contract STRING,
      PaperlessBilling STRING,
      PaymentMethod STRING,
      MonthlyCharges FLOAT,
      TotalCharges FLOAT,
      Churn STRING
  )
  CLUSTERED BY (tenure) INTO 6 BUCKETS;

  INSERT OVERWRITE TABLE telecom_data_bucketed
  SELECT * FROM telecom_data;
  ```
- **Explanation**: This query creates a bucketed table by `tenure` into 6 buckets and loads data from the original table.

#### **5.2 Average Monthly Charges by Tenure Bucket**
- **Objective**: Find the average monthly charges for customers in each tenure bucket.
- **Query**:
  ```sql
  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 1 AND 10;

  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 11 AND 20;

  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 21 AND 30;

  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 31 AND 40;

  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 41 AND 50;

  SELECT AVG(MonthlyCharges)
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 51 AND 60;
  ```
- **Explanation**: These queries calculate the average monthly charges for customers in each tenure bucket.

#### **5.3 Highest Total Charges by Tenure Bucket**
- **Objective**: Find the highest total charges in each tenure bucket.
- **Query**:
  ```sql
  SELECT 'Bucket 1' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 1 AND 10
  UNION ALL
  SELECT 'Bucket 2' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 11 AND 20
  UNION ALL
  SELECT 'Bucket 3' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 21 AND 30
  UNION ALL
  SELECT 'Bucket 4' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 31 AND 40
  UNION ALL
  SELECT 'Bucket 5' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 41 AND 50
  UNION ALL
  SELECT 'Bucket 6' AS Bucket, MAX(TotalCharges) AS MaxCharge
  FROM telecom_data_bucketed
  WHERE tenure BETWEEN 51 AND 60;
  ```
- **Explanation**: This query finds the highest total charges in each tenure bucket.

---

### **Problem 6: Performance Optimization with Joins**

#### **6.1 Load Demographics Dataset**
- **Objective**: Load the demographics dataset into another Hive table.
- **Query**:
  ```sql
  CREATE TABLE demographic_data (
      customerID STRING,
      demographic_field1 STRING,
      demographic_field2 STRING,
      ...
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE;

  LOAD DATA INPATH '/path/to/demographic_data.csv' INTO TABLE demographic_data;
  ```
- **Explanation**: This query creates a table for the demographics dataset and loads data into it.

#### **6.2 Join Telecom Data with Demographics Data**
- **Objective**: Join the telecom data table with the demographics table using different types of joins (common join, map join, bucket map join, and sorted merge bucket join).
- **Query**:
  ```sql
  -- Common Join
  SELECT t.customerID, t.Churn, d.demographic_field1
  FROM telecom_data t
  JOIN demographic_data d ON t.customerID = d.customerID;

  -- Map Join
  SELECT /*+ MAPJOIN(d) */ t.customerID, t.Churn, d.demographic_field1
  FROM telecom_data t
  JOIN demographic_data d ON t.customerID = d.customerID;

  -- Bucket Map Join
  SET hive.optimize.bucketmapjoin = true;
  SELECT t.customerID, t.Churn, d.demographic_field1
  FROM telecom_data_bucketed t
  JOIN demographic_data d ON t.customerID = d.customerID;

  -- Sorted Merge Bucket Join
  SET hive.auto.convert.sortmerge.join = true;
  SELECT t.customerID, t.Churn, d.demographic_field1
  FROM telecom_data_bucketed t
  JOIN demographic_data d ON t.customerID = d.customerID;
  ```
- **Explanation**: These queries demonstrate different types of joins between the telecom data and demographics tables.

---

### **Problem 7: Advanced Analysis**

#### **7.1 Distribution of Payment Method Among Churned Customers**
- **Objective**: Find the distribution of payment methods among churned customers.
- **Query**:
  ```sql
  SELECT PaymentMethod, COUNT(*)
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY PaymentMethod;
  ```
- **Explanation**: This query counts the number of churned customers for each payment method.

#### **7.2 Churn Rate by Internet Service Category**
- **Objective**: Calculate the churn rate (percentage of customers who left) for each internet service category.
- **Query**:
  ```sql
  SELECT InternetService, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE Churn = 'Yes') AS churn_rate
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY InternetService;
  ```
- **Explanation**: This query calculates the churn rate for each internet service category.

#### **7.3 Churned Customers with No Dependents by Contract Type**
- **Objective**: Find the number of customers who have no dependents and have churned, grouped by contract type.
- **Query**:
  ```sql
  SELECT Contract, COUNT(*)
  FROM telecom_data
  WHERE Churn = 'Yes' AND Dependents = 'No'
  GROUP BY Contract;
  ```
- **Explanation**: This query counts the number of churned customers with no dependents for each contract type.

#### **7.4 Top 5 Tenure Lengths with Highest Churn Rates**
- **Objective**: Find the top 5 tenure lengths that have the highest churn rates.
- **Query**:
  ```sql
  SELECT tenure, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE Churn = 'Yes') AS churn_rate
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY tenure
  ORDER BY churn_rate DESC
  LIMIT 5;
  ```
- **Explanation**: This query calculates the churn rate for each tenure length and returns the top 5 tenure lengths with the highest churn rates.

#### **7.5 Average Monthly Charges for Churned Customers with Phone Service**
- **Objective**: Calculate the average monthly charges for customers who have phone service and have churned, grouped by contract type.
- **Query**:
  ```sql
  SELECT Contract, AVG(MonthlyCharges)
  FROM telecom_data
  WHERE PhoneService = 'Yes' AND Churn = 'Yes'
  GROUP BY Contract;
  ```
- **Explanation**: This query calculates the average monthly charges for churned customers with phone service, grouped by contract type.

#### **7.6 Internet Service Type Most Associated with Churned Customers**
- **Objective**: Identify which internet service type is most associated with churned customers.
- **Query**:
  ```sql
  SELECT InternetService, COUNT(*)
  FROM telecom_data
  WHERE Churn = 'Yes'
  GROUP BY InternetService
  ORDER BY COUNT(*) DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the internet service type most associated with churned customers.

#### **7.7 Churn Rate for Customers with and Without Partners**
- **Objective**: Determine if customers with a partner have a lower churn rate compared to those without.
- **Query**:
  ```sql
  SELECT Partner, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE Partner = 'Yes') AS churn_rate_with_partner
  FROM telecom_data
  WHERE Churn = 'Yes' AND Partner = 'Yes'
  UNION ALL
  SELECT Partner, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE Partner = 'No') AS churn_rate_without_partner
  FROM telecom_data
  WHERE Churn = 'Yes' AND Partner = 'No';
  ```
- **Explanation**: This query calculates the churn rate for customers with and without partners.

#### **7.8 Relationship Between MultipleLines and Churn Rate**
- **Objective**: Analyze the relationship between multiple lines and churn rate.
- **Query**:
  ```sql
  SELECT MultipleLines, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE MultipleLines = 'Yes') AS churn_rate_with_multiplelines
  FROM telecom_data
  WHERE Churn = 'Yes' AND MultipleLines = 'Yes'
  UNION ALL
  SELECT MultipleLines, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM telecom_data WHERE MultipleLines = 'No') AS churn_rate_without_multiplelines
  FROM telecom_data
  WHERE Churn = 'Yes' AND MultipleLines = 'No';
  ```
- **Explanation**: This query calculates the churn rate for customers with and without multiple lines.

---

### **Summary**

- **Problem 1**: Focused on loading data into Hive and displaying the top rows.
- **Problem 2**: Explored basic data metrics like total customers, churned customers, and distribution by gender and