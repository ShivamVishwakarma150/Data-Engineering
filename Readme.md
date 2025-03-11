# Database Fundamentals

## What is a Database?
A database is an organized collection of data stored and accessed electronically. It allows for efficient data management, retrieval, and manipulation.

### Transactional Databases vs NoSQL Databases
- **Transactional Databases**: These are relational databases that support ACID properties (Atomicity, Consistency, Isolation, Durability). They are ideal for structured data and complex queries.
- **NoSQL Databases**: These are non-relational databases designed for unstructured or semi-structured data. They offer high scalability and flexibility but may not support ACID properties.

### DBMS & RDBMS
- **DBMS (Database Management System)**: Software that interacts with the database, applications, and users to capture and analyze data.
- **RDBMS (Relational Database Management System)**: A type of DBMS that stores data in tables (relations) and supports relationships between tables.

### Transactions & ACID Properties
- **Transactions**: A sequence of operations performed as a single logical unit of work.
- **ACID Properties**:
  - **Atomicity**: Ensures that all operations within a transaction are completed successfully; otherwise, the transaction is aborted.
  - **Consistency**: Ensures that the database remains in a valid state before and after the transaction.
  - **Isolation**: Ensures that concurrent transactions do not interfere with each other.
  - **Durability**: Ensures that once a transaction is committed, it remains permanent even in the event of a system failure.

### Setup MySQL Workbench
MySQL Workbench is a unified visual tool for database architects, developers, and DBAs. It provides data modeling, SQL development, and comprehensive administration tools.

### Setup MySQL Using Docker
Docker allows you to run MySQL in a containerized environment. You can pull the MySQL image from Docker Hub and run it with the necessary configurations.

### DDL, DML, DQL, DCL
- **DDL (Data Definition Language)**: Commands like `CREATE`, `ALTER`, `DROP` used to define and modify database structures.
- **DML (Data Manipulation Language)**: Commands like `INSERT`, `UPDATE`, `DELETE` used to manipulate data within tables.
- **DQL (Data Query Language)**: Commands like `SELECT` used to query data from the database.
- **DCL (Data Control Language)**: Commands like `GRANT`, `REVOKE` used to control access to data.

### CREATE Command
The `CREATE` command is used to create new databases, tables, or other database objects.

### INSERT Command
The `INSERT` command is used to add new records (rows) to a table.

### Integrity Constraints
Integrity constraints ensure the accuracy and consistency of data in a database. Examples include:
- **Primary Key**: Uniquely identifies each record in a table.
- **Foreign Key**: Ensures referential integrity by linking two tables.
- **Unique**: Ensures all values in a column are unique.
- **Not Null**: Ensures a column cannot have a NULL value.
- **Check**: Ensures that all values in a column satisfy a specific condition.

---

# SQL Commands and Operations

## ALTER Command
The `ALTER` command is used to modify the structure of an existing table, such as adding, deleting, or modifying columns.

## Drop, Truncate, and Delete
- **DROP**: Removes the entire table or database.
- **TRUNCATE**: Removes all rows from a table but retains the table structure.
- **DELETE**: Removes specific rows from a table based on a condition.

## Primary Key vs Foreign Key
- **Primary Key**: Uniquely identifies each record in a table.
- **Foreign Key**: A column or set of columns in one table that refers to the primary key in another table, ensuring referential integrity.

## Referential Integrity
Referential integrity ensures that relationships between tables remain consistent. It is enforced using foreign keys.

## SELECT Query, In-Built Functions, Aliases
- **SELECT**: Retrieves data from one or more tables.
- **In-Built Functions**: Functions like `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` used to perform calculations on data.
- **Aliases**: Temporary names given to tables or columns for the purpose of a query.

## UPDATE Command
The `UPDATE` command is used to modify existing records in a table.

## Auto Increment in CREATE TABLE
Auto Increment allows a unique number to be generated automatically when a new record is inserted into a table.

## LIMIT
The `LIMIT` clause is used to specify the number of records to return in a query.

## ORDER BY Clause
The `ORDER BY` clause is used to sort the result set in ascending or descending order.

## Conditional Operators
Operators like `=`, `!=`, `>`, `<`, `>=`, `<=` used to filter data based on conditions.

## Logical Operators
Operators like `AND`, `OR`, `NOT` used to combine multiple conditions in a query.

## LIKE Operation
The `LIKE` operator is used in a `WHERE` clause to search for a specified pattern in a column.

## User Defined Functions (UDFs)
UDFs are custom functions created by users to perform specific operations that are not available as built-in functions.

---

# Advanced SQL Concepts

## IS NULL, IS NOT NULL
- **IS NULL**: Checks if a column has a NULL value.
- **IS NOT NULL**: Checks if a column does not have a NULL value.

## GROUP BY, HAVING Clause
- **GROUP BY**: Groups rows that have the same values into summary rows.
- **HAVING**: Filters groups based on a condition.

## GROUP_CONCAT, GROUP_ROLLUP
- **GROUP_CONCAT**: Concatenates values from multiple rows into a single string.
- **GROUP_ROLLUP**: Generates subtotals and grand totals in the result set.

## Subqueries, IN and NOT IN
- **Subqueries**: Queries nested inside another query.
- **IN**: Checks if a value is within a set of values.
- **NOT IN**: Checks if a value is not within a set of values.

## CASE-When
The `CASE` statement is used to implement conditional logic in SQL.

## SQL Joins
- **INNER JOIN**: Returns records that have matching values in both tables.
- **LEFT JOIN**: Returns all records from the left table and matched records from the right table.
- **RIGHT JOIN**: Returns all records from the right table and matched records from the left table.
- **FULL JOIN**: Returns all records when there is a match in either left or right table.
- **CROSS JOIN**: Returns the Cartesian product of the two tables.

---

# Big Data Fundamentals & Hadoop

## Big Data Fundamentals
Big Data refers to extremely large datasets that cannot be processed using traditional data processing techniques.

### 5 V’s of Big Data
- **Volume**: The sheer amount of data.
- **Velocity**: The speed at which data is generated and processed.
- **Variety**: The different types of data (structured, semi-structured, unstructured).
- **Veracity**: The uncertainty or reliability of data.
- **Value**: The usefulness of the data in deriving insights.

### Distributed Computation
Distributed computation involves processing data across multiple machines in a cluster to handle large datasets efficiently.

### Distributed Storage
Distributed storage systems store data across multiple nodes to ensure scalability and fault tolerance.

### Cluster, Commodity Hardware
- **Cluster**: A group of interconnected computers that work together as a single system.
- **Commodity Hardware**: Inexpensive, readily available hardware used in clusters.

### File Formats
Common file formats in Big Data include CSV, JSON, Parquet, Avro, and ORC.

### Types of Data
- **Structured Data**: Data that is organized in a fixed format (e.g., relational databases).
- **Semi-Structured Data**: Data that does not conform to a fixed schema but has some structure (e.g., JSON, XML).
- **Unstructured Data**: Data that does not have a predefined structure (e.g., text, images, videos).

### History of Hadoop
Hadoop was created by Doug Cutting and Mike Cafarella in 2005 to handle large-scale data processing.

### Hadoop Architecture & Components
- **HDFS (Hadoop Distributed File System)**: A distributed file system that stores data across multiple machines.
- **MapReduce**: A programming model for processing large datasets in parallel.
- **YARN (Yet Another Resource Negotiator)**: Manages resources in a Hadoop cluster.

---

# Apache Hive

## Hive Complete Architecture
Hive is a data warehouse infrastructure built on top of Hadoop for providing data summarization, query, and analysis.

### Hadoop Cluster Setup on GCP (Dataproc)
Google Cloud Dataproc is a managed Hadoop and Spark service that allows you to easily set up and manage Hadoop clusters.

---

# Confluent Kafka

## Kafka Cluster Architecture
Kafka is a distributed streaming platform that allows you to publish, subscribe to, store, and process streams of records in real-time.

### Brokers
Brokers are Kafka servers that store data and serve client requests.

### Topics
Topics are categories or feeds to which records are published.

### Partitions
Partitions allow topics to be split across multiple brokers for scalability.

### Producer-Consumer, Consumer Group
- **Producer**: Publishes records to Kafka topics.
- **Consumer**: Subscribes to topics and processes the records.
- **Consumer Group**: A group of consumers that share the same group ID and divide the task of consuming records.

### Offset Management
Offsets are used to track the position of a consumer in a partition.

### Replicas
Replicas are copies of partitions stored on multiple brokers for fault tolerance.

### Commits
Commits are used to save the current offset of a consumer.

### Sync & Async Commits
- **Sync Commits**: Block until the commit is confirmed.
- **Async Commits**: Do not block and allow the consumer to continue processing.

---

# MongoDB (NoSQL Database)

## CAP Theorem
The CAP theorem states that a distributed system can only provide two out of the following three guarantees:
- **Consistency**: Every read receives the most recent write.
- **Availability**: Every request receives a response.
- **Partition Tolerance**: The system continues to operate despite network partitions.

### What is MongoDB and MongoDB Atlas?
- **MongoDB**: A NoSQL document database that stores data in JSON-like documents.
- **MongoDB Atlas**: A fully managed cloud database service for MongoDB.

### MongoDB vs Relational Database
- **MongoDB**: Schema-less, flexible, and scalable.
- **Relational Database**: Schema-based, rigid, and less scalable.

### MongoDB Features
- Document-oriented storage.
- Full index support.
- Replication and high availability.
- Auto-sharding for horizontal scaling.

### MongoDB Use Cases and Applications
- Real-time analytics.
- Content management systems.
- Mobile applications.
- Internet of Things (IoT).

### MongoDB Architecture
- **Node**: A single instance of MongoDB.
- **Data Center**: A collection of nodes.
- **Cluster**: A group of data centers.
- **Data Replication**: Copies of data stored on multiple nodes.
- **Write Operation**: The process of inserting or updating data.
- **Read Operation**: The process of retrieving data.
- **Indexing**: Improves the speed of data retrieval.

---

# Apache Spark (PySpark)

## Problems with Hadoop Map-Reduce
- High latency due to disk I/O.
- Limited to batch processing.
- Complex programming model.

### What is Apache Spark?
Apache Spark is a fast, in-memory data processing engine with elegant and expressive development APIs.

### Features of Spark
- In-memory processing.
- Supports batch and real-time processing.
- Easy-to-use APIs in Java, Scala, Python, and R.

### Spark Ecosystem
- **Spark Core**: The foundation of the Spark platform.
- **Spark SQL**: For structured data processing.
- **Spark Streaming**: For real-time data processing.
- **MLlib**: For machine learning.
- **GraphX**: For graph processing.

### RDD in Spark
- **RDD (Resilient Distributed Dataset)**: The fundamental data structure of Spark, representing an immutable, partitioned collection of elements.

### Properties of RDD
- Immutable.
- Partitioned.
- Fault-tolerant.

### How Spark Performs Data Partitioning?
Spark partitions data across the cluster to enable parallel processing.

### Transformation in Spark
Operations like `map`, `filter`, `reduceByKey` that transform an RDD into another RDD.

### Narrow Transformation vs Wide Transformation
- **Narrow Transformation**: Does not require data shuffling (e.g., `map`, `filter`).
- **Wide Transformation**: Requires data shuffling (e.g., `reduceByKey`, `join`).

### Action in Spark
Operations like `count`, `collect`, `saveAsTextFile` that trigger the execution of transformations and return results to the driver program.

### Lazy Evaluation in Spark
Spark delays the execution of transformations until an action is called.

### Lineage Graph or DAG in Spark
A Directed Acyclic Graph (DAG) represents the sequence of transformations applied to an RDD.

### Job, Stage, and Task in Spark
- **Job**: A sequence of transformations and actions triggered by an action.
- **Stage**: A set of tasks that can be executed together.
- **Task**: The smallest unit of work sent to an executor.

### Spark In-Depth Architecture and Its Components
- **Driver Program**: The main program that runs the Spark application.
- **Cluster Manager**: Manages resources in the cluster (e.g., YARN, Mesos).
- **Executor**: Runs tasks on worker nodes.

### Spark with Standalone Cluster Manager Type
A simple cluster manager included with Spark that allows you to set up a cluster without additional software.

### Spark with YARN Cluster Manager Type
YARN (Yet Another Resource Negotiator) is a resource manager in Hadoop that can be used to manage Spark applications.

### Deployment Modes of Spark Application
- **Cluster Mode**: The driver runs on a worker node.
- **Client Mode**: The driver runs on the client machine.

---

# Databricks

## What is Databricks?
Databricks is a unified data analytics platform that provides an interactive workspace for data scientists, engineers, and analysts.

### Unity Catalog
A centralized metadata management system for Databricks.

### Delta Lake & Delta Tables
- **Delta Lake**: An open-source storage layer that brings ACID transactions to Apache Spark.
- **Delta Tables**: Tables stored in Delta Lake format.

### Databricks Account Setup on GCP
Setting up a Databricks account on Google Cloud Platform (GCP).

### Workspace Setup
Configuring the Databricks workspace for collaborative data analysis.

### Metastore Setup
Setting up a metastore to manage metadata for tables and databases.

### Managed & External Catalog Setup
Configuring managed and external catalogs for data organization.

### Volumes in Databricks
Volumes are used to store and manage large datasets in Databricks.

### Databricks Cluster Setup
Setting up a cluster in Databricks for distributed data processing.

### PySpark Notebook Setup
Configuring a PySpark notebook for data analysis and processing.

### Read/Write from Databricks Volume in PySpark Notebook
Reading and writing data from Databricks volumes using PySpark.

### Create Delta Table and Write Data in PySpark Using DeltaTable Python API
Creating Delta tables and writing data using the DeltaTable API.

### Write Partitioned Data in Delta Table
Writing data into Delta tables with partitions for efficient querying.

### Read from Delta Table in PySpark
Reading data from Delta tables using PySpark.

### Time Travel in Delta Table
Querying historical data in Delta tables using time travel.

---

# Apache Airflow

## What is Orchestration in Big Data?
Orchestration involves managing and coordinating complex workflows and dependencies in data pipelines.

### Need for Dependency Management in Data Pipeline Design
Dependency management ensures that tasks in a data pipeline are executed in the correct order.

### What is Airflow?
Apache Airflow is an open-source platform to programmatically author, schedule, and monitor workflows.

### Architecture & Different Components of Airflow
- **Web Server**: Provides a user interface for managing workflows.
- **Scheduler**: Triggers tasks based on dependencies and schedules.
- **Executor**: Runs the tasks.
- **Metadata Database**: Stores the state of workflows and tasks.

### Operators in Airflow
Operators define the individual tasks in a workflow. Examples include `BashOperator`, `PythonOperator`, and `DummyOperator`.

### How to Write Airflow DAG Scripts?
DAG (Directed Acyclic Graph) scripts define the workflow in Airflow.

### Attribute Description
Attributes like `start_date`, `schedule_interval`, and `retries` are used to configure DAGs.

### How to Execute Parallel Tasks?
Parallel tasks can be executed by defining multiple tasks with no dependencies between them.

---

# Data Warehousing

## OLAP vs OLTP
- **OLAP (Online Analytical Processing)**: Optimized for complex queries and data analysis.
- **OLTP (Online Transaction Processing)**: Optimized for fast transaction processing.

### What is a Data Warehouse?
A data warehouse is a centralized repository for storing and analyzing large volumes of data from multiple sources.

### Difference Between Data Warehouse, Data Lake, and Data Mart
- **Data Warehouse**: Structured data storage optimized for analysis.
- **Data Lake**: Stores raw data in its native format.
- **Data Mart**: A subset of a data warehouse focused on a specific business line.

### Fact Tables
Fact tables store quantitative data for analysis, such as sales or transactions.

### Dimension Tables
Dimension tables store descriptive attributes related to fact tables, such as customer or product information.

### Slowly Changing Dimensions (SCD)
SCD are used to manage changes in dimension data over time.

### Types of SCDs
- **Type 1**: Overwrites old data with new data.
- **Type 2**: Adds a new row for changes, preserving history.
- **Type 3**: Adds a new column to track changes.

### Star Schema Design
A star schema consists of a central fact table surrounded by dimension tables.

### Snowflake Schema Design
A snowflake schema is a normalized version of the star schema, with dimension tables split into multiple related tables.

### Galaxy Schema Design
A galaxy schema (or fact constellation) consists of multiple fact tables sharing dimension tables.

---

# Snowflake

## Snowflake Free Tier Account Setup
Setting up a free tier account on Snowflake for data warehousing.

### Snowflake UI Walkthrough
Navigating the Snowflake user interface for data management and analysis.

### Load Data from UI and Create Snowflake Table
Loading data into Snowflake and creating tables for analysis.

### Event-Driven Data Ingestion in Snowflake Table Using SnowPipe
Using SnowPipe for real-time data ingestion into Snowflake.

### How to Create and Schedule Task in Snowflake
Creating and scheduling tasks in Snowflake for automated data processing.

---

# BigQuery

## BigQuery Overview
BigQuery is a fully managed, serverless data warehouse that enables super-fast SQL queries using the processing power of Google's infrastructure.

### BigQuery Architecture
- **Capacitor**: Columnar storage format used by BigQuery.
- **Colossus**: Google's distributed file system.
- **Dremel**: Execution engine for running SQL queries.
- **Borg**: Compute resource management system.
- **Jupiter**: High-speed network for data transfer.

---

# AWS Cloud

## AWS Services Covered
- **Event Bridge Scheduler**: Schedule AWS Lambda functions or other events.
- **Event Bridge Pipe**: Connect event sources to targets.
- **Kinesis**: Real-time data streaming.
- **Kinesis Firehose**: Load streaming data into data stores.
- **DynamoDB**: NoSQL database service.
- **SNS (Simple Notification Service)**: Send notifications.
- **SQS (Simple Queue Service)**: Message queuing service.
- **S3 (Simple Storage Service)**: Object storage.
- **Lambda**: Serverless compute service.
- **IAM (Identity and Access Management)**: Manage access to AWS services.
- **CloudWatch**: Monitor AWS resources.
- **EC2 (Elastic Compute Cloud)**: Virtual servers in the cloud.
- **Step Function**: Coordinate distributed applications.
- **EMR (Elastic MapReduce)**: Big data processing.
- **Glue**: ETL (Extract, Transform, Load) service.
- **RDS (Relational Database Service)**: Managed relational database.
- **Athena**: Query data in S3 using SQL.
- **Redshift**: Data warehousing service.

---

# Industrial Projects

## Project 1: Order Tracking Event-Driven Data Ingestion
**Tech Stack**: Google Storage, PySpark, Databricks, Delta Lake, Databricks Workflows, GitHub

## Project 2: UPI Transactions Real-Time CDC Feed Processing
**Tech Stack**: Databricks, Spark Structured Streaming, Delta Lake

## Project 3: Travel Bookings Data Ingestion Pipeline with SCD2 Merge
**Tech Stack**: Databricks, PySpark, Delta Lake, Delta Live Table Job

## Project 4: Healthcare Delta Live Table Pipeline with Medallion Architecture
**Tech Stack**: Databricks, PySpark, Delta Lake, Delta Live Table Job

## Project 5: Flight Booking Data Pipeline with Airflow & CICD
**Tech Stack**: GitHub, GitHub Actions, Google Storage, PySpark, Dataproc Serverless, Airflow, BigQuery

## Project 6: News Data Analysis with Event-Driven Incremental Load in Snowflake Table
**Tech Stack**: Airflow, Google Cloud Storage, Python, Snowflake

## Project 7: Movie Booking CDC Data Real-Time Aggregation in Snowflake Dynamic Table
**Tech Stack**: Python, Snowflake Dynamic Table, Snowflake Stream, Snowflake Tasks, Streamlit

## Project 8: Car Rental Data Batch Ingestion with SCD2 Merge in Snowflake Table
**Tech Stack**: Python, PySpark, GCP Dataproc, Airflow, Snowflake

## Project 9: IRCTC Streaming Data Ingestion into BigQuery
**Tech Stack**: Python, GCP Storage, GCP Pub-Sub, BigQuery, Dataflow

## Project 10: Walmart Data Ingestion into BigQuery
**Tech Stack**: Python, Airflow, GCP Storage, BigQuery

## Project 11: Quality Movie Data Analysis
**Tech Stack**: S3, Glue Crawler, Glue Catalog, Glue Catalog Data Quality, Glue Low Code ETL (With PySpark), Redshift, Event Bridge, SNS

## Project 12: Gadget Sales Data Projection
**Tech Stack**: Python, DynamoDB, DynamoDB Streams, Kinesis Streams, Event Bridge Pipe, Kinesis Firehose, S3, Lambda, Athena

## Project 13: Airline Data Ingestion
**Tech Stack**: S3, S3 Cloudtrail Notification, Event Bridge Pattern Rule, Glue Crawler, Glue Visual ETL (With PySpark), SNS, Redshift, Step Function

## Project 14: Logistics Data Warehouse Management
**Tech Stack**: GCP Storage, Airflow (GCP Composer), Hive Operators, PySpark With GCP Dataproc, Hive

## Project 15: Sales Order & Payment Data Real-Time Ingestion
**Tech Stack**: GCP Pub-Sub, Python, Docker, Cassandra