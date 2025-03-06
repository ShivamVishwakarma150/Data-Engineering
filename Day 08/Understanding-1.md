### **Detailed Explanation of Hive Assignment 1: Car Insurance Cold Calls Data Analysis**

This assignment involves analyzing a dataset related to car insurance cold calls. The dataset contains information about customers, such as their age, job, marital status, education, balance, insurance status, and call details. The assignment is divided into several problems, each focusing on different aspects of Hive, such as **data loading**, **data exploration**, **aggregations**, **partitioning**, **bucketing**, **join optimizations**, **window functions**, and **performance tuning**.

Below is a detailed explanation of each problem, along with the corresponding HiveQL queries and their results.

---

### **Problem 1: Data Loading**

#### **1.1 Create an External Table and Load Data**
- **Objective**: Create an external table in Hive and load data from a text file stored in HDFS.
- **Query**:
  ```sql
  CREATE EXTERNAL TABLE car_insurance_data (
      id INT,
      Age INT,
      Job STRING,
      Marital STRING,
      Education STRING,
      Default INT,
      Balance INT,
      HHInsurance INT,
      CarLoan INT,
      Communication STRING,
      LastContactDay INT,
      LastContactMonth INT,
      NoOfContacts INT,
      DaysPassed INT,
      PrevAttempts INT,
      Outcome STRING,
      CallStart STRING,
      CallEnd STRING,
      CarInsurance INT
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/hive/data/car_insurance_data/';
  ```
- **Explanation**:
  - An external table named `car_insurance_data` is created with the given schema.
  - The data is stored as a text file, and the fields are delimited by commas.
  - The table points to the HDFS location `/hive/data/car_insurance_data/`.

---

### **Problem 2: Data Exploration**

#### **2.1 Count the Number of Records**
- **Objective**: Find the total number of records in the dataset.
- **Query**:
  ```sql
  SELECT COUNT(*) FROM car_insurance_data;
  ```
- **Explanation**: This query counts all rows in the `car_insurance_data` table.

#### **2.2 Count Unique Job Categories**
- **Objective**: Find the number of unique job categories.
- **Query**:
  ```sql
  SELECT COUNT(DISTINCT Job) FROM car_insurance_data;
  ```
- **Explanation**: This query counts the distinct values in the `Job` column.

#### **2.3 Age Distribution of Customers**
- **Objective**: Break down the age distribution into groups: 18-30, 31-45, 46-60, and 61+.
- **Query**:
  ```sql
  SELECT 
      CASE
          WHEN Age BETWEEN 18 AND 30 THEN '18-30'
          WHEN Age BETWEEN 31 AND 45 THEN '31-45'
          WHEN Age BETWEEN 46 AND 60 THEN '46-60'
          ELSE '61+'
      END AS age_group,
      COUNT(*) AS count
  FROM car_insurance_data
  GROUP BY 
      CASE
          WHEN Age BETWEEN 18 AND 30 THEN '18-30'
          WHEN Age BETWEEN 31 AND 45 THEN '31-45'
          WHEN Age BETWEEN 46 AND 60 THEN '46-60'
          ELSE '61+'
      END;
  ```
- **Explanation**: This query groups customers by age and counts the number of customers in each group.

#### **2.4 Count Records with Missing Values**
- **Objective**: Count the number of records with missing values in any field.
- **Query**:
  ```sql
  SELECT COUNT(*)
  FROM car_insurance_data
  WHERE Id IS NULL
     OR Age IS NULL
     OR Job IS NULL
     OR Marital IS NULL
     OR Education IS NULL
     OR Default IS NULL
     OR Balance IS NULL
     OR HHInsurance IS NULL
     OR CarLoan IS NULL
     OR Communication IS NULL
     OR LastContactDay IS NULL
     OR LastContactMonth IS NULL
     OR NoOfContacts IS NULL
     OR DaysPassed IS NULL
     OR PrevAttempts IS NULL
     OR Outcome IS NULL
     OR CallStart IS NULL
     OR CallEnd IS NULL
     OR CarInsurance IS NULL;
  ```
- **Explanation**: This query counts records where any column has a `NULL` value.

#### **2.5 Count Unique 'Outcome' Values**
- **Objective**: Find the number of unique 'Outcome' values and their counts.
- **Query**:
  ```sql
  SELECT Outcome, COUNT(*)
  FROM car_insurance_data
  GROUP BY Outcome;
  ```
- **Explanation**: This query groups records by the `Outcome` column and counts the number of records for each outcome.

#### **2.6 Count Customers with Both Car Loan and Home Insurance**
- **Objective**: Find the number of customers who have both a car loan and home insurance.
- **Query**:
  ```sql
  SELECT COUNT(*)
  FROM car_insurance_data
  WHERE CarLoan = 1 AND HHInsurance = 1;
  ```
- **Explanation**: This query counts records where both `CarLoan` and `HHInsurance` are `1`.

---

### **Problem 3: Aggregations**

#### **3.1 Average, Minimum, and Maximum Balance by Job**
- **Objective**: Calculate the average, minimum, and maximum balance for each job category.
- **Query**:
  ```sql
  SELECT Job, AVG(Balance) AS average_balance, MIN(Balance) AS min_balance, MAX(Balance) AS max_balance
  FROM car_insurance_data
  GROUP BY Job;
  ```
- **Explanation**: This query calculates the average, minimum, and maximum balance for each job category.

#### **3.2 Count Customers with and Without Car Insurance**
- **Objective**: Find the total number of customers with and without car insurance.
- **Query**:
  ```sql
  SELECT CarInsurance, COUNT(*)
  FROM car_insurance_data
  GROUP BY CarInsurance;
  ```
- **Explanation**: This query groups records by `CarInsurance` and counts the number of records for each group.

#### **3.3 Count Customers by Communication Type**
- **Objective**: Count the number of customers for each communication type.
- **Query**:
  ```sql
  SELECT Communication, COUNT(*)
  FROM car_insurance_data
  GROUP BY Communication;
  ```
- **Explanation**: This query groups records by `Communication` and counts the number of records for each communication type.

#### **3.4 Sum of Balance by Communication Type**
- **Objective**: Calculate the sum of `Balance` for each communication type.
- **Query**:
  ```sql
  SELECT Communication, SUM(Balance)
  FROM car_insurance_data
  GROUP BY Communication;
  ```
- **Explanation**: This query calculates the total balance for each communication type.

#### **3.5 Count 'PrevAttempts' by Outcome**
- **Objective**: Count the number of `PrevAttempts` for each `Outcome`.
- **Query**:
  ```sql
  SELECT Outcome, SUM(PrevAttempts)
  FROM car_insurance_data
  GROUP BY Outcome;
  ```
- **Explanation**: This query calculates the total number of previous attempts for each outcome.

#### **3.6 Average 'NoOfContacts' by Car Insurance Status**
- **Objective**: Calculate the average number of contacts for customers with and without car insurance.
- **Query**:
  ```sql
  SELECT CarInsurance, AVG(NoOfContacts)
  FROM car_insurance_data
  GROUP BY CarInsurance;
  ```
- **Explanation**: This query calculates the average number of contacts for customers with and without car insurance.

---

### **Problem 4: Partitioning and Bucketing**

#### **4.1 Create a Partitioned Table**
- **Objective**: Create a table partitioned by `Education` and `Marital` status.
- **Query**:
  ```sql
  CREATE TABLE car_insurance_data_partitioned (
      Id INT,
      Age INT,
      Job STRING,
      Default INT,
      Balance INT,
      HHInsurance INT,
      CarLoan INT,
      Communication STRING,
      LastContactDay INT,
      LastContactMonth INT,
      NoOfContacts INT,
      DaysPassed INT,
      PrevAttempts INT,
      Outcome STRING,
      CallStart STRING,
      CallEnd STRING,
      CarInsurance INT
  )
  PARTITIONED BY (Education STRING, Marital STRING)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE;

  INSERT OVERWRITE TABLE car_insurance_data_partitioned PARTITION(Education, Marital)
  SELECT Id, Age, Job, Default, Balance, HHInsurance, CarLoan, Communication, LastContactDay, LastContactMonth, NoOfContacts, DaysPassed, PrevAttempts, Outcome, CallStart, CallEnd, CarInsurance, Education, Marital
  FROM car_insurance_data;
  ```
- **Explanation**: This query creates a partitioned table and loads data from the original table.

#### **4.2 Create a Bucketed Table**
- **Objective**: Create a table bucketed by `Age` into 4 buckets.
- **Query**:
  ```sql
  CREATE TABLE car_insurance_data_bucketed (
      Id INT,
      Age INT,
      Job STRING,
      Marital STRING,
      Education STRING,
      Default INT,
      Balance INT,
      HHInsurance INT,
      CarLoan INT,
      Communication STRING,
      LastContactDay INT,
      LastContactMonth INT,
      NoOfContacts INT,
      DaysPassed INT,
      PrevAttempts INT,
      Outcome STRING,
      CallStart STRING,
      CallEnd STRING,
      CarInsurance INT
  )
  CLUSTERED BY (Age) INTO 4 BUCKETS
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE;

  INSERT OVERWRITE TABLE car_insurance_data_bucketed
  SELECT * FROM car_insurance_data;
  ```
- **Explanation**: This query creates a bucketed table and loads data from the original table.

---

### **Problem 5: Optimized Joins**

#### **5.1 Join Original Table with Partitioned Table**
- **Objective**: Find the average `Balance` for each `Job` and `Education` level.
- **Query**:
  ```sql
  SELECT o.Job, p.Education, AVG(o.Balance) AS average_balance
  FROM car_insurance_data o
  JOIN car_insurance_data_partitioned p ON o.Id = p.Id
  GROUP BY o.Job, p.Education;
  ```
- **Explanation**: This query joins the original table with the partitioned table and calculates the average balance for each job and education level.

#### **5.2 Join Original Table with Bucketed Table**
- **Objective**: Calculate the total `NoOfContacts` for each `Age` group.
- **Query**:
  ```sql
  SELECT o.Age, SUM(o.NoOfContacts) AS total_contacts
  FROM car_insurance_data o
  JOIN car_insurance_data_bucketed b ON o.Id = b.Id
  GROUP BY o.Age;
  ```
- **Explanation**: This query joins the original table with the bucketed table and calculates the total number of contacts for each age group.

#### **5.3 Join Partitioned Table with Bucketed Table**
- **Objective**: Find the total balance for each education level and marital status for each age group.
- **Query**:
  ```sql
  SELECT p.Age, p.Education, p.Marital, SUM(b.Balance) AS total_balance
  FROM car_insurance_data_partitioned p
  JOIN car_insurance_data_bucketed b ON p.Id = b.Id
  GROUP BY p.Age, p.Education, p.Marital;
  ```
- **Explanation**: This query joins the partitioned table with the bucketed table and calculates the total balance for each education level, marital status, and age group.

---

### **Problem 6: Window Functions**

#### **6.1 Cumulative Sum of 'NoOfContacts' by Job**
- **Objective**: Calculate the cumulative sum of `NoOfContacts` for each `Job` category, ordered by `Age`.
- **Query**:
  ```sql
  SELECT Age, Job, NoOfContacts, SUM(NoOfContacts) OVER (PARTITION BY Job ORDER BY Age) AS cumulative_sum
  FROM car_insurance_data
  ORDER BY Age, Job;
  ```
- **Explanation**: This query calculates the cumulative sum of contacts for each job category, ordered by age.

#### **6.2 Running Average of 'Balance' by Job**
- **Objective**: Calculate the running average of `Balance` for each `Job` category, ordered by `Age`.
- **Query**:
  ```sql
  SELECT Age, Job, Balance, AVG(Balance) OVER (PARTITION BY Job ORDER BY Age) AS running_average
  FROM car_insurance_data
  ORDER BY Age, Job;
  ```
- **Explanation**: This query calculates the running average of balance for each job category, ordered by age.

#### **6.3 Maximum 'Balance' by Job and Age**
- **Objective**: Find the maximum `Balance` for each `Job` and `Age` group.
- **Query**:
  ```sql
  SELECT Age, Job, Balance
  FROM (
      SELECT Age, Job, Balance, ROW_NUMBER() OVER (PARTITION BY Job, Age ORDER BY Balance DESC) AS rn
      FROM car_insurance_data
  ) t
  WHERE rn = 1
  ORDER BY Job, Age;
  ```
- **Explanation**: This query finds the maximum balance for each job and age group using window functions.

#### **6.4 Rank of 'Balance' by Job**
- **Objective**: Calculate the rank of `Balance` within each `Job` category, ordered by `Balance` descending.
- **Query**:
  ```sql
  SELECT Age, Job, Balance, RANK() OVER (PARTITION BY Job ORDER BY Balance DESC) AS balance_rank
  FROM car_insurance_data
  ORDER BY Job, Balance DESC;
  ```
- **Explanation**: This query ranks the balance within each job category, ordered by balance in descending order.

---

### **Problem 7: Advanced Aggregations**

#### **7.1 Job Category with Highest Car Insurances**
- **Objective**: Find the job category with the highest number of car insurances.
- **Query**:
  ```sql
  SELECT Job
  FROM (
      SELECT Job, COUNT(*) AS car_insurance_count
      FROM car_insurance_data
      WHERE CarInsurance = 1
      GROUP BY Job
  ) t
  ORDER BY car_insurance_count DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the job category with the highest number of car insurances.

#### **7.2 Month with Highest Last Contacts**
- **Objective**: Find the month with the highest number of last contacts.
- **Query**:
  ```sql
  SELECT LastContactMonth, COUNT(*) AS contact_count
  FROM car_insurance_data
  GROUP BY LastContactMonth
  ORDER BY contact_count DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the month with the highest number of last contacts.

#### **7.3 Ratio of Customers with and Without Car Insurance**
- **Objective**: Calculate the ratio of customers with car insurance to those without car insurance for each job category.
- **Query**:
  ```sql
  SELECT t1.Job, t1.car_insurance_count / t2.no_car_insurance_count AS car_insurance_ratio
  FROM (
      SELECT Job, COUNT(*) AS car_insurance_count
      FROM car_insurance_data
      WHERE CarInsurance = 1
      GROUP BY Job
  ) t1
  JOIN (
      SELECT Job, COUNT(*) AS no_car_insurance_count
      FROM car_insurance_data
      WHERE CarInsurance = 0
      GROUP BY Job
  ) t2
  ON t1.Job = t2.Job;
  ```
- **Explanation**: This query calculates the ratio of customers with car insurance to those without car insurance for each job category.

#### **7.4 Job and Education with Highest Car Insurances**
- **Objective**: Find the job and education level combination with the highest number of car insurances.
- **Query**:
  ```sql
  SELECT Job, Education
  FROM (
      SELECT Job, Education, COUNT(*) AS car_insurance_count
      FROM car_insurance_data
      WHERE CarInsurance = 1
      GROUP BY Job, Education
  ) t
  ORDER BY car_insurance_count DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the job and education level combination with the highest number of car insurances.

#### **7.5 Average 'NoOfContacts' by Outcome and Job**
- **Objective**: Calculate the average number of contacts for each outcome and job combination.
- **Query**:
  ```sql
  SELECT Outcome, Job, AVG(NoOfContacts) AS average_contacts
  FROM car_insurance_data
  GROUP BY Outcome, Job;
  ```
- **Explanation**: This query calculates the average number of contacts for each outcome and job combination.

#### **7.6 Month with Highest Total Balance**
- **Objective**: Determine the month with the highest total balance of customers.
- **Query**:
  ```sql
  SELECT LastContactMonth, SUM(Balance) AS total_balance
  FROM car_insurance_data
  GROUP BY LastContactMonth
  ORDER BY total_balance DESC
  LIMIT 1;
  ```
- **Explanation**: This query finds the month with the highest total balance of customers.

---

### **Problem 8: Complex Joins and Aggregations**

#### **8.1 Average Balance for Customers with Car Loan and Home Insurance**
- **Objective**: For customers who have both a car loan and home insurance, find the average balance for each education level.
- **Query**:
  ```sql
  SELECT Education, AVG(Balance) AS average_balance
  FROM car_insurance_data
  WHERE CarLoan = 1 AND HHInsurance = 1
  GROUP BY Education;
  ```
- **Explanation**: This query calculates the average balance for customers with both a car loan and home insurance, grouped by education level.

#### **8.2 Top 3 Communication Types for Customers with Car Insurance**
- **Objective**: Identify the top 3 communication types for customers with car insurance and display their average number of contacts.
- **Query**:
  ```sql
  SELECT Communication, AVG(NoOfContacts) AS average_contacts
  FROM car_insurance_data
  WHERE CarInsurance = 1
  GROUP BY Communication
  ORDER BY average_contacts DESC
  LIMIT 3;
  ```
- **Explanation**: This query finds the top 3 communication types for customers with car insurance and calculates their average number of contacts

### **Problem 8: Complex Joins and Aggregations (Continued)**

#### **8.3 Average Balance for Customers with Car Loan**
- **Objective**: For customers who have a car loan, calculate the average balance for each job category.
- **Query**:
  ```sql
  SELECT Job, AVG(Balance) AS average_balance
  FROM car_insurance_data
  WHERE CarLoan = 1
  GROUP BY Job;
  ```
- **Explanation**: This query calculates the average balance for customers with a car loan, grouped by job category.

#### **8.4 Top 5 Job Categories with Most Customers Having a Default**
- **Objective**: Identify the top 5 job categories that have the most customers with a default and show their average balance.
- **Query**:
  ```sql
  SELECT Job, AVG(Balance) AS average_balance
  FROM car_insurance_data
  WHERE Default = 1
  GROUP BY Job
  ORDER BY COUNT(*) DESC
  LIMIT 5;
  ```
- **Explanation**: This query finds the top 5 job categories with the most customers having a default and calculates their average balance.

---

### **Problem 9: Advanced Window Functions**

#### **9.1 Difference in 'NoOfContacts' Between Customers**
- **Objective**: Calculate the difference in the number of contacts (`NoOfContacts`) between each customer and the customer with the next highest number of contacts in the same job category.
- **Query**:
  ```sql
  SELECT c1.Id, c1.Job, c1.NoOfContacts,
         c1.NoOfContacts - c2.NextHighestContacts AS ContactDifference
  FROM car_insurance_data c1
  JOIN (
      SELECT c1.Job, c1.NoOfContacts,
             MIN(c2.NoOfContacts) AS NextHighestContacts
      FROM car_insurance_data c1
      LEFT JOIN car_insurance_data c2 ON c1.Job = c2.Job AND c1.NoOfContacts < c2.NoOfContacts
      GROUP BY c1.Job, c1.NoOfContacts
  ) c2 ON c1.Job = c2.Job AND c1.NoOfContacts = c2.NoOfContacts;
  ```
- **Explanation**: This query calculates the difference in the number of contacts between each customer and the next customer with a higher number of contacts in the same job category.

#### **9.2 Difference Between Customer Balance and Job Average Balance**
- **Objective**: For each customer, calculate the difference between their balance and the average balance of their job category.
- **Query**:
  ```sql
  SELECT c.Id, c.Job, c.Balance,
         c.Balance - j.AvgBalance AS BalanceDifference
  FROM car_insurance_data c
  JOIN (
      SELECT Job, AVG(Balance) AS AvgBalance
      FROM car_insurance_data
      GROUP BY Job
  ) j ON c.Job = j.Job;
  ```
- **Explanation**: This query calculates the difference between each customer's balance and the average balance of their job category.

#### **9.3 Customer with Longest Call Duration in Each Job Category**
- **Objective**: For each job category, find the customer who had the longest call duration.
- **Query**:
  ```sql
  SELECT Job, Id, CallDuration
  FROM (
      SELECT Job, Id, CallDuration,
             ROW_NUMBER() OVER (PARTITION BY Job ORDER BY CallDuration DESC) AS rn
      FROM car_insurance_data
  ) t
  WHERE rn = 1;
  ```
- **Explanation**: This query identifies the customer with the longest call duration in each job category using window functions.

#### **9.4 Moving Average of 'NoOfContacts'**
- **Objective**: Calculate the moving average of the number of contacts (`NoOfContacts`) within each job category, using a window frame of the current row and the two preceding rows.
- **Query**:
  ```sql
  SELECT Id, Job, NoOfContacts,
         AVG(NoOfContacts) OVER (PARTITION BY Job ORDER BY Id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_average
  FROM car_insurance_data;
  ```
- **Explanation**: This query calculates the moving average of the number of contacts for each job category, considering the current row and the two preceding rows.

---

### **Problem 10: Performance Tuning**

#### **10.1 Experiment with Different File Formats**
- **Objective**: Experiment with different file formats (e.g., ORC, Parquet) and measure their impact on the performance of Hive queries.
- **Steps**:
  1. Create tables using ORC and Parquet file formats.
  2. Load data into these tables.
  3. Run the same queries on these tables and compare execution times.
- **Example**:
  ```sql
  -- Create ORC table
  CREATE TABLE car_insurance_data_orc STORED AS ORC AS SELECT * FROM car_insurance_data;

  -- Create Parquet table
  CREATE TABLE car_insurance_data_parquet STORED AS PARQUET AS SELECT * FROM car_insurance_data;
  ```
- **Explanation**: ORC and Parquet are columnar file formats that improve query performance by reducing I/O and storage requirements.

#### **10.2 Use Different Levels of Compression**
- **Objective**: Use different levels of compression (e.g., Snappy, GZIP) and observe their effects on storage and query performance.
- **Steps**:
  1. Set the compression codec in Hive.
  2. Load data into tables with different compression settings.
  3. Compare storage size and query performance.
- **Example**:
  ```sql
  -- Set compression to Snappy
  SET hive.exec.compress.output=true;
  SET mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;

  -- Set compression to GZIP
  SET hive.exec.compress.output=true;
  SET mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec;
  ```
- **Explanation**: Compression reduces storage size but may increase CPU usage during query execution.

#### **10.3 Compare Join Queries with and Without Bucketing**
- **Objective**: Compare the execution time of join queries with and without bucketing.
- **Steps**:
  1. Create bucketed and non-bucketed tables.
  2. Run join queries on both tables.
  3. Compare execution times.
- **Example**:
  ```sql
  -- Join without bucketing
  SELECT * FROM car_insurance_data c1 JOIN car_insurance_data c2 ON c1.Id = c2.Id;

  -- Join with bucketing
  SELECT * FROM car_insurance_data_bucketed c1 JOIN car_insurance_data_bucketed c2 ON c1.Id = c2.Id;
  ```
- **Explanation**: Bucketing improves join performance by reducing the amount of data scanned.

#### **10.4 Optimize Queries Using Hive Optimization Techniques**
- **Objective**: Optimize Hive queries using techniques like predicate pushdown, map-side joins, etc., and discuss the difference in performance.
- **Steps**:
  1. Enable predicate pushdown and map-side joins.
  2. Run queries with and without optimizations.
  3. Compare execution times.
- **Example**:
  ```sql
  -- Enable predicate pushdown
  SET hive.optimize.ppd=true;

  -- Enable map-side join
  SET hive.auto.convert.join=true;
  ```
- **Explanation**: Predicate pushdown reduces the amount of data read, and map-side joins eliminate the need for a reduce phase, improving query performance.

---

### **Summary**

- **Problem 8**: Focused on complex joins and aggregations, such as finding average balances for specific customer segments and identifying top job categories with defaults.
- **Problem 9**: Used advanced window functions to calculate differences, ranks, and moving averages within job categories.
- **Problem 10**: Explored performance tuning techniques, including file formats, compression, bucketing, and query optimizations.

By completing this assignment, you gain hands-on experience with Hive's powerful features for data analysis, optimization, and performance tuning.