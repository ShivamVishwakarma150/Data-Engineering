# **Kafka to MongoDB Pipeline: Detailed Documentation**

## **1. Overview**
This document explains how data is produced from a CSV file, published to a Kafka topic, and then consumed and stored in MongoDB.

- **Producer:** Reads data from a CSV file and publishes it to Kafka.
- **Kafka Broker:** Holds the messages until consumed.
- **Consumer:** Reads messages from Kafka and stores them in MongoDB.

---
## **2. Data Flow Architecture**

```plaintext
+------------------+        +----------------+        +----------------+
| CSV File        | -----> | Kafka Producer | -----> |  Kafka Broker  |
+------------------+        +----------------+        +----------------+
                                                  |
                                                  v
                                          +----------------+
                                          | Kafka Consumer |
                                          +----------------+
                                                  |
                                                  v
                                         +-----------------+
                                         |    MongoDB      |
                                         +-----------------+
```

---
## **3. Kafka Producer Code Explanation**

### **3.1 Import Required Libraries**
```python
import datetime
import threading
from decimal import *
from time import sleep
from uuid import uuid4, UUID
import time

from confluent_kafka import SerializingProducer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
from confluent_kafka.serialization import StringSerializer
import pandas as pd
```
- Imports required libraries for Kafka, Avro serialization, and CSV handling.

### **3.2 Kafka Configuration**
```python
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc'
}
```
- Configures Kafka broker connection and security settings.

### **3.3 Schema Registry Configuration**
```python
schema_registry_client = SchemaRegistryClient({
  'url': 'http://schema-registry:8081',
  'basic.auth.user.info': 'klm:abc123'
})
```
- Connects to Confluent Schema Registry to fetch the latest Avro schema.

### **3.4 Fetch and Define Avro Serializer**
```python
subject_name = 'retail_data_test-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str

key_serializer = StringSerializer('utf_8')
avro_serializer = AvroSerializer(schema_registry_client, schema_str)
```
- Retrieves the Avro schema for the Kafka topic.
- Defines key and value serializers.

### **3.5 Create Kafka Producer**
```python
producer = SerializingProducer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.serializer': key_serializer,
    'value.serializer': avro_serializer
})
```
- Initializes a Kafka producer with the defined configurations.

### **3.6 Load CSV Data**
```python
df = pd.read_csv('retail_data.csv')
df = df.fillna('null')
```
- Reads the CSV file and replaces null values.

### **3.7 Produce Messages to Kafka**
```python
for index, row in df.iterrows():
    value = row.to_dict()
    producer.produce(topic='retail_data_test', key=str(index), value=value, on_delivery=delivery_report)
    producer.flush()
    time.sleep(2)
```
- Iterates over each row of the CSV file, serializes it, and sends it to Kafka.

---
## **4. Kafka Consumer Code Explanation**

### **4.1 Import Required Libraries**
```python
from confluent_kafka import Consumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from pymongo import MongoClient
```
- Imports required Kafka consumer and MongoDB libraries.

### **4.2 Kafka Consumer Configuration**
```python
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'group.id': 'mongo-consumer-group',
    'auto.offset.reset': 'earliest',
    'enable.auto.commit': True,
    'security.protocol': 'SASL_SSL',
    'sasl.mechanisms': 'PLAIN',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc'
}
```
- Configures Kafka consumer settings.

### **4.3 Connect to Schema Registry**
```python
schema_registry_client = SchemaRegistryClient({
    'url': 'http://schema-registry:8081',
    'basic.auth.user.info': 'klm:abc123'
})

avro_deserializer = AvroDeserializer(schema_registry_client)
```
- Connects to Schema Registry and initializes Avro deserializer.

### **4.4 Initialize Consumer and MongoDB Connection**
```python
consumer = Consumer(kafka_config)
consumer.subscribe(['retail_data_test'])

mongo_client = MongoClient("mongodb://username:password@mongo-host:27017/")
db = mongo_client["database_name"]
collection = db["retail_data"]
```
- Subscribes to the Kafka topic.
- Connects to MongoDB.

### **4.5 Consume Messages and Insert into MongoDB**
```python
try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            print("Consumer error: {}".format(msg.error()))
            continue
        
        value = avro_deserializer(msg.value(), None)
        collection.insert_one(value)
        print("Inserted:", value)

except KeyboardInterrupt:
    pass
finally:
    consumer.close()
    mongo_client.close()
```
- Polls Kafka for new messages.
- Deserializes messages and inserts them into MongoDB.

---
## **5. MongoDB Storage Structure**
```json
{
  "_id": ObjectId("65abc123def456"),
  "product_id": "12345",
  "product_name": "Laptop",
  "price": 999.99,
  "quantity": 10,
  "timestamp": "2025-03-12T10:00:00Z"
}
```
- Each Kafka message is stored as a document in MongoDB.

---
## **6. Summary of Data Flow**
| **Step** | **Description** |
|---------|----------------|
| **Producer** | Reads CSV, serializes, and sends data to Kafka. |
| **Kafka Broker** | Stores messages in a topic (`retail_data_test`). |
| **Consumer** | Reads messages, deserializes them. |
| **MongoDB** | Stores the messages as documents. |

---
## **7. Future Improvements**
✅ **Batch Processing** for efficiency.  
✅ **Error Handling & Logging** for debugging.  
✅ **Indexing in MongoDB** to speed up queries.  

<br/>
<br/>

# **Kafka to MongoDB: How Data Moves from Kafka to MongoDB?**  

Once the **Kafka Producer** (your script) publishes data to a Kafka topic (`retail_data_test`), a **Kafka Consumer** retrieves this data and stores it in **MongoDB**. Here’s a detailed breakdown of how this process works:

---

## **1. Data Flow Overview**
1. **Producer** sends data → **Kafka Topic (`retail_data_test`)**
2. **Kafka Consumer (MongoDB Connector or Custom Consumer)** reads data from the topic.
3. The **consumer processes and deserializes** the Avro-encoded data.
4. The **consumer inserts data into MongoDB**.

---

## **2. Kafka Consumer: Fetching Data from Kafka**
The **Kafka Consumer** is responsible for reading the Avro-encoded messages from the Kafka topic and storing them in MongoDB.

### **Using Kafka Connect MongoDB Sink Connector (Easiest Approach)**
Kafka provides an **official MongoDB Sink Connector** to move data from Kafka topics to MongoDB automatically.

### **Kafka Connect MongoDB Sink Configuration**
You can configure Kafka Connect to store messages from the `retail_data_test` topic into MongoDB.

#### **Example Sink Connector Configuration (`mongo-sink.properties`)**
```properties
name=mongo-sink-connector
connector.class=io.confluent.connect.mongodb.MongoDbSinkConnector
tasks.max=1
topics=retail_data_test

connection.uri=mongodb://username:password@mongo-host:27017/database_name
database=database_name
collection=retail_data
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=io.confluent.connect.avro.AvroConverter
value.converter.schema.registry.url=http://schema-registry:8081
```
🔹 **`connection.uri`** → MongoDB connection string.  
🔹 **`topics`** → The Kafka topic to read from (`retail_data_test`).  
🔹 **`value.converter`** → Uses **AvroConverter** to deserialize Avro data.  
🔹 **`collection`** → Specifies the MongoDB collection (`retail_data`).  

### **Steps to Use Kafka Connect for MongoDB**
1. Install **Kafka Connect MongoDB Sink** (`confluentinc-kafka-connect-mongodb`).
2. Configure **Kafka Connect Worker** to use the above `mongo-sink.properties`.
3. Start Kafka Connect → It automatically consumes messages and inserts them into MongoDB.

---

## **3. Custom Kafka Consumer in Python**
If you don’t want to use Kafka Connect, you can write a **custom consumer** in Python using the `confluent_kafka` and `pymongo` libraries.

### **Python Kafka Consumer Code**
```python
from confluent_kafka import Consumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from pymongo import MongoClient

# Kafka Consumer Configuration
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'group.id': 'mongo-consumer-group',
    'auto.offset.reset': 'earliest',
    'enable.auto.commit': True,
    'security.protocol': 'SASL_SSL',
    'sasl.mechanisms': 'PLAIN',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc'
}

# Schema Registry Configuration
schema_registry_client = SchemaRegistryClient({
    'url': 'http://schema-registry:8081',
    'basic.auth.user.info': 'klm:abc123'
})

# Avro Deserializer
avro_deserializer = AvroDeserializer(schema_registry_client)

# Initialize Consumer
consumer = Consumer(kafka_config)
consumer.subscribe(['retail_data_test'])

# MongoDB Connection
mongo_client = MongoClient("mongodb://username:password@mongo-host:27017/")
db = mongo_client["database_name"]
collection = db["retail_data"]

# Poll Kafka and insert into MongoDB
try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            print("Consumer error: {}".format(msg.error()))
            continue
        
        # Deserialize Avro data
        value = avro_deserializer(msg.value(), None)

        # Insert into MongoDB
        collection.insert_one(value)
        print("Inserted:", value)

except KeyboardInterrupt:
    pass
finally:
    consumer.close()
    mongo_client.close()
```

### **Explanation of the Consumer Code**
1. **Kafka Consumer**
   - Connects to Kafka (`bootstrap.servers`).
   - Subscribes to the topic `retail_data_test`.
   - Uses `poll()` to continuously fetch messages.

2. **Schema Registry & Avro Deserialization**
   - Uses **AvroDeserializer** to convert binary Avro messages into JSON-like Python dictionaries.

3. **MongoDB Insertion**
   - Establishes a connection with MongoDB.
   - Inserts each message into the `retail_data` collection.

---

## **4. MongoDB Storage Structure**
Once the messages are processed by the consumer and stored in MongoDB, they will have a structure similar to:

```json
{
  "_id": ObjectId("65abc123def456"),
  "product_id": "12345",
  "product_name": "Laptop",
  "price": 999.99,
  "quantity": 10,
  "timestamp": "2025-03-12T10:00:00Z"
}
```
- Each Kafka message becomes a **document in MongoDB**.
- The `_id` field is autogenerated unless explicitly provided.

---

## **5. Performance & Optimization Considerations**
✅ **Batch Processing:** Instead of inserting messages one by one, process in batches for better performance.  
✅ **Indexing in MongoDB:** Ensure indexes are created on frequently queried fields (`product_id`, `timestamp`).  
✅ **Error Handling & Logging:** Log failed messages for debugging.  
✅ **Consumer Group Scaling:** Use multiple consumers in a consumer group to handle high loads.  

---

## **Final Summary**
| **Step**  | **Details** |
|-----------|------------|
| **1. Producer** | Python script produces Avro data to `retail_data_test`. |
| **2. Kafka Broker** | Stores messages in the Kafka topic. |
| **3. Consumer** | Reads messages from Kafka (`Kafka Connect` or Python Consumer). |
| **4. Avro Deserialization** | Converts Avro-encoded messages into JSON. |
| **5. MongoDB Storage** | Messages are stored in MongoDB collections. |
