# **📌 Cassandra Assignment Requirements**  

### **🔹 Objective:**  
The assignment requires building a **real-time data pipeline** using **Apache Kafka** and **Apache Cassandra** to process and store e-commerce order data.

---

## **📌 Tasks to Complete**  

### **1️⃣ Load and Process Data**
- Read the provided dataset (`olist_orders_dataset.csv`).  
- Perform **basic data cleaning** (handle missing values).  

### **2️⃣ Set Up Kafka**
- Install & configure **Apache Kafka**.  
- Create a **Kafka topic** named `ecommerce-orders`.  

### **3️⃣ Implement Kafka Producer (`producer.py`)**
- Read order data from **CSV**.  
- Serialize it in **JSON format**.  
- Publish data to the **Kafka topic (`ecommerce-orders`)**.  

### **4️⃣ Set Up Cassandra**
- Install & configure **Apache Cassandra**.  
- Create a **keyspace** named `ecommerce`.  
- Define an **orders table** to store processed data.  

### **5️⃣ Implement Kafka Consumer (`consumer.py`)**
- Subscribe to the **Kafka topic (`ecommerce-orders`)**.  
- Deserialize & process incoming messages.  
- Extract additional fields like `OrderHour` and `OrderDayOfWeek`.  
- Insert processed data into the **Cassandra orders table**.  

### **6️⃣ Ensure Consistency**
- Use **`CONSISTENCY QUORUM`** when inserting data into Cassandra.  

### **7️⃣ Test the Pipeline**
- **Run the producer (`producer.py`)** to send data.  
- **Run the consumer (`consumer.py`)** to process & store data.  
- **Verify stored data** using Cassandra queries.  

### **8️⃣ Assignment Submission**
You need to submit the following files:  
✔ **Kafka Producer script** (`producer.py`).  
✔ **Kafka Consumer script** (`consumer.py`).  
✔ **CQL file (`CQL.docx`)** containing Cassandra keyspace & table creation queries.  
✔ **Pipeline explanation report (`Pipeline_explanation.docx`)** describing the workflow.  

---

## **📌 Expected Output**
- Orders **ingested from CSV → Kafka → Cassandra**.  
- Orders stored in Cassandra with **additional fields (`OrderHour`, `OrderDayOfWeek`)**.  
- Queries in Cassandra should return valid order records.  


<br/>
<br/>

# **📌 Cassandra Assignment Explanation**  

## **📌 Overview**  
This assignment focuses on building a **real-time data pipeline** using **Apache Kafka** and **Apache Cassandra**. The objective is to **ingest e-commerce order data**, transform it, and store it in a **Cassandra database** while ensuring **quorum consistency**.

---

## **📌 Problem Statement**  
You are a **data engineer** at a large **e-commerce company** that processes **real-time customer orders**.  
- The data arrives in **CSV format**.  
- It must be **processed in real-time** to extract **valuable insights**.  
- The processed data is **stored in Cassandra** for further analysis.  

---

## **📌 Assignment Steps & Explanation**  

### **1️⃣ Load and Examine the Dataset**
🔹 Load **`olist_orders_dataset.csv`** into a **Pandas DataFrame** and inspect its structure.  

```python
import pandas as pd

df = pd.read_csv('olist_orders_dataset.csv')
print(df.head())  # Display first 5 rows
```
✅ **Why?**  
✔ Ensures data is **correctly formatted** before sending it to Kafka.  

---

### **2️⃣ Apache Kafka Setup**  
🔹 **Install & Configure Kafka** on your system.  
🔹 **Create a Kafka topic named `ecommerce-orders`** to store e-commerce data.  

```sh
# Start Zookeeper (if not running)
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka Server
bin/kafka-server-start.sh config/server.properties

# Create Kafka Topic
bin/kafka-topics.sh --create --topic ecommerce-orders --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```
✅ **Why?**  
✔ Enables **real-time order streaming** into Kafka.  

---

### **3️⃣ Develop Kafka Producer (`producer.py`)**  
🔹 Reads CSV data and **publishes it to Kafka** (`ecommerce-orders` topic).  

```python
from confluent_kafka import Producer
import json

# Kafka Configuration
kafka_config = {
    'bootstrap.servers': 'localhost:9092'
}
producer = Producer(kafka_config)

# Read CSV and Send Data to Kafka
df = pd.read_csv('olist_orders_dataset.csv')
for _, row in df.iterrows():
    order_data = row.to_dict()
    key = f"{row['customer_id']}_{row['order_id']}"  # Use 'customer_id' and 'order_id' as key
    value = json.dumps(order_data)

    producer.produce('ecommerce-orders', key=key, value=value)
    producer.flush()
```
✅ **Why?**  
✔ **Streams data into Kafka** in **real-time**.  
✔ Uses **customer_id + order_id as the Kafka key** for proper partitioning.  

---

### **4️⃣ Install & Set Up Apache Cassandra**  
🔹 Install Cassandra and create a **keyspace** named `ecommerce`.  

```sql
CREATE KEYSPACE ecommerce WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};
```
✅ **Why?**  
✔ Provides **scalability & fault tolerance** in Cassandra.  

---

### **5️⃣ Cassandra Data Model (`orders` Table)**  
🔹 Define a **table named `orders`** to store **processed** order data.  
🔹 Include **additional columns** (`OrderHour`, `OrderDayOfWeek`).  

```sql
CREATE TABLE ecommerce.orders (
    order_id UUID,
    customer_id UUID,
    order_status TEXT,
    order_purchase_timestamp TIMESTAMP,
    order_approved_at TIMESTAMP,
    order_delivered_carrier_date TIMESTAMP,
    order_delivered_customer_date TIMESTAMP,
    order_estimated_delivery_date TIMESTAMP,
    OrderHour INT,
    OrderDayOfWeek TEXT,
    PRIMARY KEY ((customer_id), order_id, order_purchase_timestamp)
);
```
✅ **Why?**  
✔ **`customer_id` as Partition Key** → Groups orders per customer.  
✔ **`order_id, order_purchase_timestamp` as Clustering Keys** → Ensures ordering of orders.  
✔ Stores **order hour & day** for analytics.  

---

### **6️⃣ Kafka Consumer & Data Transformation (`consumer.py`)**  
🔹 **Consumes messages from Kafka**, extracts additional fields, and **stores them in Cassandra**.  

```python
from confluent_kafka import Consumer
from cassandra.cluster import Cluster
import json
from datetime import datetime

# Kafka Consumer Configuration
consumer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'ecommerce-group',
    'auto.offset.reset': 'earliest'
})
consumer.subscribe(['ecommerce-orders'])

# Connect to Cassandra
cluster = Cluster(['127.0.0.1'])
session = cluster.connect()
session.set_keyspace('ecommerce')

# Process Messages
while True:
    msg = consumer.poll(1.0)
    if msg is None:
        continue

    value = json.loads(msg.value().decode('utf-8'))
    order_id = value['order_id']
    customer_id = value['customer_id']
    order_time = datetime.strptime(value['order_purchase_timestamp'], '%Y-%m-%d %H:%M:%S')

    # Extract new columns
    order_hour = order_time.hour
    order_day_of_week = order_time.strftime('%A')

    # Insert Data into Cassandra
    query = """
    INSERT INTO ecommerce.orders 
    (order_id, customer_id, order_status, order_purchase_timestamp, OrderHour, OrderDayOfWeek) 
    VALUES (?, ?, ?, ?, ?, ?)
    """
    session.execute(query, (order_id, customer_id, value['order_status'], order_time, order_hour, order_day_of_week))
```
✅ **Why?**  
✔ **Transforms data** (extracts `OrderHour`, `OrderDayOfWeek`).  
✔ **Inserts processed data** into Cassandra.  

---

### **7️⃣ Ensure Quorum Consistency**
🔹 When inserting data into Cassandra, ensure **quorum consistency**.  

```sql
INSERT INTO ecommerce.orders 
(order_id, customer_id, order_status, order_purchase_timestamp, OrderHour, OrderDayOfWeek) 
VALUES (?, ?, ?, ?, ?, ?) USING CONSISTENCY QUORUM;
```
✅ **Why?**  
✔ Guarantees **data accuracy across replicas** before write is confirmed.  

---

### **8️⃣ Test the Pipeline**
1️⃣ **Run Kafka Producer (`producer.py`)**  
```sh
python producer.py
```
2️⃣ **Run Kafka Consumer (`consumer.py`)**  
```sh
python consumer.py
```
3️⃣ **Verify Data in Cassandra**
```sql
SELECT * FROM ecommerce.orders LIMIT 10;
```
✅ **Why?**  
✔ Confirms **data is processed & stored correctly**.  

---

### **9️⃣ Assignment Submission**
✅ Submit the following:  
✔ **Python scripts** for **Producer (`producer.py`)** and **Consumer (`consumer.py`)**.  
✔ **CQL file (`CQL.docx`)** with keyspace & table creation queries.  
✔ **Detailed report (`Pipeline_explanation.docx`)** explaining the pipeline.  

---

## **📌 Final Summary**
| **Step** | **Task** |
|----------|---------|
| **1️⃣ Load Data** | Read `olist_orders_dataset.csv` into Pandas. |
| **2️⃣ Set Up Kafka** | Install & create `ecommerce-orders` topic. |
| **3️⃣ Kafka Producer** | Reads data & streams it into Kafka. |
| **4️⃣ Set Up Cassandra** | Create `ecommerce` keyspace & `orders` table. |
| **5️⃣ Kafka Consumer** | Reads Kafka messages, extracts fields & stores in Cassandra. |
| **6️⃣ Quorum Consistency** | Ensures data is safely written to Cassandra. |
| **7️⃣ Test Pipeline** | Run end-to-end & verify in Cassandra. |

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add performance optimizations for Kafka & Cassandra?**  
✔ **Implement real-time analytics (Apache Spark or Grafana)?**  
✔ **Provide troubleshooting tips for common errors?**  


<br/>
<br/>

# **📌 Explanation of `producer.py`**  

📄 **[File: `producer.py`]**  

This script **reads order data from a CSV file** and **publishes it to a Kafka topic (`ecommerce-orders`)** using **Avro serialization**.  

---

## **📌 1. Key Functionalities**
✔ **Reads `olist_orders_dataset.csv` (Order Data)**  
✔ **Serializes data using Avro format**  
✔ **Sends data to Kafka topic (`ecommerce-orders`)**  

---

## **📌 2. Step-by-Step Code Explanation**  

### **🔹 Step 1: Import Dependencies**  
The script uses `pandas` for reading CSV data and `confluent_kafka` for Kafka communication.  

```python
import pandas as pd
import json
import time
from confluent_kafka import Producer
```
✅ **Why?**  
- **`pandas`** → Reads order data from CSV.  
- **`json`** → Serializes data before sending to Kafka.  
- **`confluent_kafka.Producer`** → Publishes messages to Kafka.  

---

### **🔹 Step 2: Load Order Data from CSV**  
Reads the `olist_orders_dataset.csv` file into a Pandas DataFrame.  

```python
df = pd.read_csv('olist_orders_dataset.csv')
df.fillna("", inplace=True)  # Replaces NaN values with empty strings
```
✅ **Why?**  
- **Ensures missing values (`NaN`) don’t break message serialization.**  

---

### **🔹 Step 3: Kafka Producer Configuration**  
Configures connection to the **Kafka cluster (Confluent Cloud)** with **SASL authentication**.  

```python
kafka_config = {
    'bootstrap.servers': 'ind-5fg89.australia-southeast2.gcp.confluent.cloud:9092',
    'security.protocol': 'SASL_SSL',
    'sasl.mechanisms': 'PLAIN',
    'sasl.username': '<username>',
    'sasl.password': '<password>'
}
producer = Producer(kafka_config)
```
✅ **Why?**  
- **`bootstrap.servers`** → Kafka broker URL.  
- **`security.protocol`** → Ensures **secure connection** via SSL.  
- **`sasl.username/password`** → Authentication credentials.  

---

### **🔹 Step 4: Define Kafka Delivery Callback**  
This function **logs success or failure** when a message is delivered to Kafka.  

```python
def delivery_report(err, msg):
    if err is not None:
        print(f"Message delivery failed: {err}")
    else:
        print(f"Message delivered to {msg.topic()} [{msg.partition()}]")
```
✅ **Why?**  
- Logs **errors** for debugging.  
- Confirms **successful delivery** of messages.  

---

### **🔹 Step 5: Publish Messages to Kafka**  
Loops over **each order**, serializes it as JSON, and **sends it to Kafka**.  

```python
for index, row in df.iterrows():
    order_data = {
        'order_id': row['order_id'],
        'customer_id': row['customer_id'],
        'order_status': row['order_status'],
        'order_purchase_timestamp': row['order_purchase_timestamp'],
        'order_approved_at': row['order_approved_at']
    }
    
    key = str(row['order_id'])  
    value = json.dumps(order_data)  

    producer.produce(
        topic='ecommerce-orders', 
        key=key, 
        value=value, 
        on_delivery=delivery_report
    )
    time.sleep(0.1)  # To simulate real-time data flow
```
✅ **Why?**  
✔ **Reads each order** from CSV.  
✔ **Serializes it to JSON (`json.dumps`)**.  
✔ **Uses `order_id` as the Kafka key** (Ensures correct partitioning).  
✔ **Calls `producer.produce()`** to publish messages.  
✔ **Adds `time.sleep(0.1)`** to **simulate real-time streaming**.  

---

### **🔹 Step 6: Flush & Close Kafka Producer**
Ensures all messages are **successfully sent** before exiting.  

```python
producer.flush()
```
✅ **Why?**  
- **Waits until all messages are sent** before the script exits.  

---

## **📌 3. Kafka Topic (`ecommerce-orders`)**
📌 Kafka acts as a **message queue** between the producer and consumer.  
✔ Uses **Confluent Schema Registry** to validate Avro messages.  
✔ Messages are stored **temporarily** before the consumer processes them.  

---

## **📌 4. Example JSON Message Sent to Kafka**
```json
{
    "order_id": "abc123",
    "customer_id": "cust789",
    "order_status": "DELIVERED",
    "order_purchase_timestamp": "2024-07-10 12:00:00",
    "order_approved_at": "2024-07-10 12:05:00"
}
```
✅ **Why JSON?**  
- **Lightweight** & easy to **deserialize** in Python (for the consumer).  

---

## **📌 5. Summary**
| **Step** | **Task** |
|----------|---------|
| **1️⃣ Load CSV** | Reads order data from `olist_orders_dataset.csv`. |
| **2️⃣ Kafka Config** | Sets up Kafka producer with authentication. |
| **3️⃣ Define Callback** | Logs success/failure of message delivery. |
| **4️⃣ Loop Over Data** | Converts each order to JSON & sends it to Kafka. |
| **5️⃣ Flush Messages** | Ensures all messages are sent before exit. |

---

## **📌 6. Next Steps**
Would you like me to:  
✔ **Optimize batch processing (send multiple messages at once)?**  
✔ **Implement retry logic for failed deliveries?**  
✔ **Integrate this with Apache Spark for real-time analytics?**  

<br/>
<br/>

# **📌 Explanation of `producer.py`**  

📄 **[File: `producer.py`]**  

This script **reads order data from a CSV file** and **publishes it to a Kafka topic (`ecommerce-orders`)** using **JSON serialization**.  

---

## **📌 1. Key Functionalities**
✔ **Reads `olist_orders_dataset.csv` (Order Data).**  
✔ **Serializes data using JSON format.**  
✔ **Sends data to Kafka topic (`ecommerce-orders`).**  
✔ **Uses a `delivery_report` callback to confirm message delivery.**  

---

## **📌 2. Step-by-Step Code Explanation**  

### **🔹 Step 1: Import Dependencies**  
The script uses `pandas` for reading CSV data and `confluent_kafka` for Kafka communication.  

```python
import pandas as pd
import json
import time
from confluent_kafka import Producer
```
✅ **Why?**  
- **`pandas`** → Reads order data from CSV.  
- **`json`** → Serializes data before sending to Kafka.  
- **`confluent_kafka.Producer`** → Publishes messages to Kafka.  

---

### **🔹 Step 2: Load Order Data from CSV**  
Reads the `olist_orders_dataset.csv` file into a Pandas DataFrame.  

```python
df = pd.read_csv('olist_orders_dataset.csv')
df.fillna("", inplace=True)  # Replaces NaN values with empty strings
```
✅ **Why?**  
- Ensures missing values (`NaN`) don’t break message serialization.  

---

### **🔹 Step 3: Kafka Producer Configuration**  
Configures connection to the **Kafka cluster (Confluent Cloud)** with **SASL authentication**.  

```python
kafka_config = {
    'bootstrap.servers': 'ind-5fg89.australia-southeast2.gcp.confluent.cloud:9092',
    'security.protocol': 'SASL_SSL',
    'sasl.mechanisms': 'PLAIN',
    'sasl.username': '<username>',
    'sasl.password': '<password>'
}
producer = Producer(kafka_config)
```
✅ **Why?**  
- **`bootstrap.servers`** → Kafka broker URL.  
- **`security.protocol`** → Ensures **secure connection** via SSL.  
- **`sasl.username/password`** → Authentication credentials.  

---

### **🔹 Step 4: Define Kafka Delivery Callback**  
This function **logs success or failure** when a message is delivered to Kafka.  

```python
def delivery_report(err, msg):
    if err is not None:
        print(f"Message delivery failed: {err}")
    else:
        print(f"Message delivered to {msg.topic()} [{msg.partition()}]")
```
✅ **Why?**  
- Logs **errors** for debugging.  
- Confirms **successful delivery** of messages.  

---

### **🔹 Step 5: Publish Messages to Kafka**  
Loops over **each order**, serializes it as JSON, and **sends it to Kafka**.  

```python
for index, row in df.iterrows():
    order_data = {
        'order_id': row['order_id'],
        'customer_id': row['customer_id'],
        'order_status': row['order_status'],
        'order_purchase_timestamp': row['order_purchase_timestamp'],
        'order_approved_at': row['order_approved_at']
    }
    
    key = str(row['order_id'])  
    value = json.dumps(order_data)  

    producer.produce(
        topic='ecommerce-orders', 
        key=key, 
        value=value, 
        on_delivery=delivery_report
    )
    time.sleep(0.1)  # To simulate real-time data flow
```
✅ **Why?**  
✔ **Reads each order** from CSV.  
✔ **Serializes it to JSON (`json.dumps`)**.  
✔ **Uses `order_id` as the Kafka key** (Ensures correct partitioning).  
✔ **Calls `producer.produce()`** to publish messages.  
✔ **Adds `time.sleep(0.1)`** to **simulate real-time streaming**.  

---

### **🔹 Step 6: Flush & Close Kafka Producer**
Ensures all messages are **successfully sent** before exiting.  

```python
producer.flush()
```
✅ **Why?**  
- **Waits until all messages are sent** before the script exits.  

---

## **📌 3. Kafka Topic (`ecommerce-orders`)**
📌 Kafka acts as a **message queue** between the producer and consumer.  
✔ Uses **Confluent Schema Registry** to validate JSON messages.  
✔ Messages are stored **temporarily** before the consumer processes them.  

---

## **📌 4. Example JSON Message Sent to Kafka**
```json
{
    "order_id": "abc123",
    "customer_id": "cust789",
    "order_status": "DELIVERED",
    "order_purchase_timestamp": "2024-07-10 12:00:00",
    "order_approved_at": "2024-07-10 12:05:00"
}
```
✅ **Why JSON?**  
- **Lightweight** & easy to **deserialize** in Python (for the consumer).  

---

## **📌 5. Summary**
| **Step** | **Task** |
|----------|---------|
| **1️⃣ Load CSV** | Reads order data from `olist_orders_dataset.csv`. |
| **2️⃣ Kafka Config** | Sets up Kafka producer with authentication. |
| **3️⃣ Define Callback** | Logs success/failure of message delivery. |
| **4️⃣ Loop Over Data** | Converts each order to JSON & sends it to Kafka. |
| **5️⃣ Flush Messages** | Ensures all messages are sent before exit. |

---

## **📌 6. Next Steps**
Would you like me to:  
✔ **Optimize batch processing (send multiple messages at once)?**  
✔ **Implement retry logic for failed deliveries?**  
✔ **Integrate this with Apache Spark for real-time analytics?**  

<br/>
<br/>

# **📌 Explanation of `cassandra_table_create.ipynb` (Jupyter Notebook)**  

This Jupyter Notebook sets up **Apache Cassandra**, connects to a database, and creates tables for an **e-commerce system**.  

---

## **📌 1. Key Functionalities**  
✔ **Installs the Cassandra Python Driver (`cassandra-driver`).**  
✔ **Connects to an AstraDB (Managed Cassandra) instance.**  
✔ **Creates and selects a keyspace (`ecommerce`).**  
✔ **Creates tables for storing e-commerce data.**  
✔ **Executes CQL queries for inserting and retrieving data.**  

---

## **📌 2. Step-by-Step Code Explanation**  

### **🔹 Step 1: Print Hello World**
```python
print("Hello World !!")
```
✅ **Basic test to confirm Jupyter Notebook execution.**  

---

### **🔹 Step 2: Install Cassandra Python Driver**  
```python
pip install cassandra-driver
```
✅ **Why?**  
- The `cassandra-driver` library is required to connect Python to **Apache Cassandra**.  

---

### **🔹 Step 3: Check Cassandra Driver Version**  
```python
import cassandra
print(cassandra.__version__)
```
✅ **Why?**  
- Ensures that the **Cassandra driver is installed and available**.  

---

### **🔹 Step 4: Connect to Cassandra (AstraDB)**
```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

cloud_config = {
  'secure_connect_bundle': 'secure-connect-cassandra-demo.zip'
}

auth_provider = PlainTextAuthProvider('YcFBXWbhJbGWbvvqFdIUmpvs', 'iAOq5Z_Z+DxYAurWtvTpdfGBSTf6IDUGHigG2oQsxUspbTY,pLkP5ADfsdfsAdjB87rewk-LP-a.OkOz+xOIr0PbDSDSdsfDSASFw3SWaUagQZcy6FulrKMvzD.dsD6eNET')
cluster = Cluster(cloud=cloud_config, auth_provider=auth_provider)
session = cluster.connect()

row = session.execute("select release_version from system.local").one()
if row:
    print(row[0])
else:
    print("An error occurred.")
```
✅ **Why?**  
✔ Connects to **AstraDB (Managed Cassandra)** using a secure connection.  
✔ Verifies **Cassandra connection** by retrieving the `release_version`.  

---

### **🔹 Step 5: Select the Keyspace (`ecommerce`)**
```python
try:
    query = "use ecommerce"
    session.execute(query)
    print("Inside the ecommerce keyspace")
except Exception as err:
    print("Exception Occurred while using Keyspace:", err)
```
✅ **Why?**  
✔ **Switches to the `ecommerce` keyspace** where tables will be created.  

---

### **🔹 Step 6: Create an Orders Table**
```python
query = """
CREATE TABLE IF NOT EXISTS ecommerce.orders (
    order_id UUID PRIMARY KEY,
    customer_id UUID,
    order_status TEXT,
    order_purchase_timestamp TIMESTAMP,
    order_approved_at TIMESTAMP,
    order_delivered_carrier_date TIMESTAMP,
    order_delivered_customer_date TIMESTAMP,
    order_estimated_delivery_date TIMESTAMP
)
"""
session.execute(query)
print("Orders table created successfully!")
```
✅ **Why?**  
✔ Defines an **`orders` table** to store **order details**.  
✔ Uses **UUID as `PRIMARY KEY`** for efficient retrieval.  

---

### **🔹 Step 7: Insert Data into Orders Table**  
```python
import uuid
from datetime import datetime

query = """
INSERT INTO ecommerce.orders (
    order_id, customer_id, order_status, order_purchase_timestamp, order_approved_at, order_delivered_carrier_date,
    order_delivered_customer_date, order_estimated_delivery_date
) VALUES (?, ?, ?, ?, ?, ?, ?, ?)
"""

session.execute(query, (uuid.uuid4(), uuid.uuid4(), 'delivered', datetime.now(), datetime.now(), datetime.now(), datetime.now(), datetime.now()))
print("Data inserted into orders table!")
```
✅ **Why?**  
✔ Inserts **order details** into the Cassandra table.  
✔ Uses `uuid.uuid4()` to generate **unique order IDs**.  
✔ Uses `datetime.now()` to insert **current timestamps**.  

---

### **🔹 Step 8: Retrieve Data from Orders Table**
```python
query = "SELECT * FROM ecommerce.orders LIMIT 5"
rows = session.execute(query)
for row in rows:
    print(row)
```
✅ **Why?**  
✔ **Fetches the first 5 orders** from the database.  
✔ **Verifies that the data was inserted successfully**.  

---

## **📌 3. Summary of Jupyter Notebook**
| **Step** | **Task** |
|----------|---------|
| **1️⃣ Install Driver** | Installs `cassandra-driver`. |
| **2️⃣ Connect to DB** | Connects to AstraDB (Managed Cassandra). |
| **3️⃣ Select Keyspace** | Uses `ecommerce` keyspace. |
| **4️⃣ Create Table** | Defines `orders` table. |
| **5️⃣ Insert Data** | Adds sample orders to the database. |
| **6️⃣ Retrieve Data** | Fetches orders for validation. |

---

## **📌 4. Next Steps**
Would you like me to:  
✔ **Optimize Cassandra queries for analytics?**  
✔ **Add indexing for faster retrieval?**  
✔ **Connect this to a real-time order processing pipeline?**  

<br/>
<br/>

# **📌 Expected Output for Producer & Consumer in Cassandra Data Pipeline**  

This section describes the expected **console output** when running the **Kafka Producer (`producer.py`)** and **Kafka Consumer (`consumer.py`)**.

---

## **📌 1. Kafka Producer Output (`producer.py`)**
When the producer reads the CSV and sends messages to Kafka, the output should look like this:

```sh
Message delivered to ecommerce-orders [0]
Message delivered to ecommerce-orders [1]
Message delivered to ecommerce-orders [2]
...
```
✅ **Explanation:**  
✔ Confirms **messages are successfully sent** to Kafka.  
✔ The `[0], [1], [2]` values indicate **Kafka partitions** where data is stored.  

### **📌 Example Message Sent to Kafka**
Each message sent to Kafka is **a JSON object** like this:

```json
{
    "order_id": "abc123",
    "customer_id": "cust789",
    "order_status": "DELIVERED",
    "order_purchase_timestamp": "2024-07-10 12:00:00",
    "order_approved_at": "2024-07-10 12:05:00"
}
```
---

## **📌 2. Kafka Consumer Output (`consumer.py`)**
When the consumer reads from Kafka, processes the message, and inserts it into Cassandra, the output should look like:

```sh
Received Order: abc123 from Customer: cust789
Processed Order Time: 2024-07-10 12:00:00
Extracted Order Hour: 12
Extracted Order Day: Wednesday
Inserted into Cassandra: Order ID abc123, Customer ID cust789
...
```
✅ **Explanation:**  
✔ **Confirms data is received from Kafka.**  
✔ **Extracts additional fields** (`OrderHour`, `OrderDayOfWeek`).  
✔ **Inserts processed order into Cassandra.**  

---

## **📌 3. Cassandra Query Output (`SELECT * FROM ecommerce.orders LIMIT 5;`)**
After processing, querying Cassandra should return:

| order_id | customer_id | order_status | order_purchase_timestamp | order_hour | order_day_of_week |
|----------|------------|--------------|---------------------------|------------|-------------------|
| abc123   | cust789    | DELIVERED    | 2024-07-10 12:00:00       | 12         | Wednesday        |
| xyz456   | cust321    | SHIPPED      | 2024-07-11 15:30:00       | 15         | Thursday         |

✅ **Explanation:**  
✔ Data from Kafka **successfully inserted into Cassandra**.  
✔ Additional fields **OrderHour & OrderDayOfWeek extracted** for analytics.  

---

## **📌 4. Error Scenarios (If Something Goes Wrong)**  
### 🔴 **Producer Error: Kafka Not Running**
```sh
Message delivery failed: Broker not available
```
**🔹 Fix:** Start Kafka server using:  
```sh
bin/kafka-server-start.sh config/server.properties
```

### 🔴 **Consumer Error: Cassandra Not Running**
```sh
cassandra.cluster.NoHostAvailable: Unable to connect to any servers
```
**🔹 Fix:** Start Cassandra using:  
```sh
cassandra -f
```

### 🔴 **Consumer Error: JSON Decode Error**
```sh
json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
```
**🔹 Fix:** Ensure producer **sends valid JSON messages**.

---

## **📌 Summary of Outputs**
| **Component** | **Expected Output** |
|--------------|-------------------|
| **Producer (`producer.py`)** | Confirms messages sent to Kafka (`Message delivered to ecommerce-orders [0]`). |
| **Consumer (`consumer.py`)** | Reads messages, extracts fields, inserts into Cassandra (`Inserted into Cassandra: Order ID abc123`). |
| **Cassandra Query (`SELECT * FROM ecommerce.orders;`)** | Returns stored orders with additional fields (`OrderHour`, `OrderDayOfWeek`). |

---

<br/>
<br/>

# **Explanation of Kafka Producer and Consumer in Context of the Given CSV Data**  

Kafka is a distributed event streaming platform used for real-time data streaming. In the context of your CSV file, which contains order data (Order ID, Delivery Status, and Timestamps), Kafka's **Producer** and **Consumer** can be used to process and analyze this data efficiently.

---

### **1. Kafka Producer (producer.py)**
The **Producer** is responsible for publishing messages (order data) to a Kafka topic. Based on your CSV file, the **Producer** would:
- Read each row from the CSV file.
- Convert the row into a JSON or structured message format.
- Publish (send) the message to a Kafka topic (e.g., `order_status_topic`).

**Example of how the Producer sends data:**
```json
{
  "order_id": "32f4f0673547f9e7bef5c7d24d8f3a87",
  "customer_id": "7c21b22cb0e33c875265ecc2c0b772fb",
  "status": "delivered",
  "order_time": "08-06-2018 16:26",
  "processing_time": "12-06-2018 05:36",
  "shipment_time": "13-06-2018 08:10",
  "delivery_time": "22-06-2018 18:32",
  "completion_time": "20-07-2018 00:00"
}
```
- The **Producer** will continuously read rows from the CSV file and send them to the Kafka topic.
- Other systems (such as analytics dashboards or databases) can consume this data for further processing.

---

### **2. Kafka Consumer (consumer.py)**
The **Consumer** listens to the Kafka topic (`order_status_topic`) and processes messages in real time. The Consumer could:
- Read incoming messages from the Kafka topic.
- Parse the order details.
- Store the order data in a database (such as Cassandra) for historical tracking.
- Trigger alerts or actions based on the order status (e.g., notify if an order is delayed).

**Example of how the Consumer processes data:**
1. The **Consumer** receives the above JSON message.
2. It extracts details such as:
   - `order_id`
   - `status`
   - `timestamps`
3. The Consumer can store this information in Cassandra for further querying.

**Cassandra Table Example for Storing Order Data:**
```sql
CREATE TABLE orders (
    order_id UUID PRIMARY KEY,
    customer_id UUID,
    status TEXT,
    order_time TIMESTAMP,
    processing_time TIMESTAMP,
    shipment_time TIMESTAMP,
    delivery_time TIMESTAMP,
    completion_time TIMESTAMP
);
```
- The **Consumer** continuously listens for new messages and inserts them into Cassandra.

---

### **3. Real-Time Use Cases**
By using Kafka, you can:
- Track orders in **real-time**.
- Perform **analytics** on order delays, shipment trends, and delivery efficiency.
- Notify customers of **order status updates** (e.g., "Your order is out for delivery").
- Detect **fraudulent activities** (e.g., too many failed deliveries for a customer).

---

### **Conclusion**
- **Producer** reads data from the CSV file and publishes it to Kafka.
- **Consumer** reads from Kafka and stores it in Cassandra for analysis.
- Kafka ensures **scalability**, **real-time processing**, and **fault tolerance**.
