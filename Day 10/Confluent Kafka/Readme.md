# Confluent Kafka

### **1. `confluent_avro_data_consumer.py`**
This script is a **Kafka consumer** that reads Avro-serialized data from a Kafka topic.

#### **Key Components**:
- **Kafka Configuration**:
  - Connects to a Kafka broker with SASL/SSL authentication.
  - Specifies the consumer group (`group.id`) and offset reset policy (`auto.offset.reset`).
- **Schema Registry**:
  - Connects to a Schema Registry to fetch the latest Avro schema for the topic.
  - Uses the schema to deserialize the Avro-encoded messages.
- **Consumer**:
  - Subscribes to the `retail_data_test` topic.
  - Polls for messages and deserializes them using the Avro schema.
  - Prints the consumed records.

#### **Workflow**:
1. Fetches the Avro schema from the Schema Registry.
2. Subscribes to the Kafka topic.
3. Polls for messages and deserializes them using the schema.
4. Prints the consumed records.

---

### **2. `confluent_avro_data_producer.py`**
This script is a **Kafka producer** that sends Avro-serialized data to a Kafka topic.

#### **Key Components**:
- **Kafka Configuration**:
  - Connects to a Kafka broker with SASL/SSL authentication.
- **Schema Registry**:
  - Connects to a Schema Registry to fetch the latest Avro schema for the topic.
  - Uses the schema to serialize the data before sending it to Kafka.
- **Producer**:
  - Reads data from a CSV file (`retail_data.csv`).
  - Serializes each row of data using the Avro schema.
  - Sends the serialized data to the `retail_data_test` topic.
  - Includes a delivery report callback to handle success/failure of message delivery.

#### **Workflow**:
1. Fetches the Avro schema from the Schema Registry.
2. Reads data from the CSV file.
3. Serializes each row of data using the Avro schema.
4. Sends the serialized data to the Kafka topic.

---

### **3. `docker_compose_confluent_kafka.yml`**
This file defines a **Docker Compose setup** for running a Confluent Kafka cluster, including Zookeeper, Kafka Broker, Schema Registry, Control Center, and Kafka Connect.

#### **Key Services**:
- **Zookeeper**:
  - Manages Kafka broker coordination and metadata.
  - Exposes port `2181` for client connections.
- **Kafka Broker**:
  - The core Kafka server that handles message storage and delivery.
  - Exposes port `9092` for client connections and `9101` for JMX monitoring.
  - Depends on Zookeeper for coordination.
- **Schema Registry**:
  - Manages Avro schemas for Kafka topics.
  - Exposes port `8081` for client connections.
  - Depends on the Kafka broker.
- **Control Center**:
  - Provides a web-based UI for monitoring and managing the Kafka cluster.
  - Exposes port `9021` for the web interface.
  - Depends on the Kafka broker and Schema Registry.
- **Kafka Connect**:
  - A tool for integrating Kafka with external systems (e.g., databases, cloud services).
  - Exposes port `8083` for REST API connections.
  - Depends on the Kafka broker and Schema Registry.

#### **Key Features**:
- **Networking**: All services are part of the `gds` network, allowing them to communicate with each other.
- **Volumes**: Persistent storage is configured for Zookeeper and Kafka data.
- **Environment Variables**: Configure Kafka and Schema Registry settings, such as replication factors and listener addresses.

---

### **4. `reatil_data_avro_schema.json`**
This file defines the **Avro schema** for the data being produced and consumed in the Kafka topic.

#### **Key Components**:
- **Namespace**: `com.kaggle.onlineretail` (logical grouping for the schema).
- **Name**: `Purchase` (name of the schema).
- **Type**: `record` (defines a structured record with multiple fields).
- **Fields**:
  - `Invoice`: A string representing the invoice number.
  - `StockCode`: A string representing the stock code.
  - `Description`: A nullable string representing the product description.
  - `Quantity`: An integer representing the quantity purchased.
  - `InvoiceDate`: A string representing the invoice date.
  - `Price`: A double representing the price.
  - `CustomerID`: A nullable string representing the customer ID.
  - `Country`: A string representing the country.

#### **Purpose**:
- Defines the structure of the data being serialized and deserialized in Kafka.
- Ensures that producers and consumers agree on the format of the data.

---

### **5. How It All Works Together**

#### **Data Flow**:
1. **Producer**:
   - Reads data from the CSV file.
   - Serializes the data using the Avro schema.
   - Sends the serialized data to the `retail_data_test` topic.
2. **Consumer**:
   - Subscribes to the `retail_data_test` topic.
   - Deserializes the data using the Avro schema.
   - Prints the consumed records.

#### **Infrastructure**:
- The **Docker Compose setup** provides the necessary infrastructure (Zookeeper, Kafka Broker, Schema Registry) for the producer and consumer to operate.
- The **Schema Registry** ensures that the producer and consumer use the same schema for serialization and deserialization.

---

### **6. Example Use Case**

Imagine a real-time retail analytics system:
- **Producer**:
  - Reads sales data from a CSV file.
  - Serializes the data using the Avro schema and sends it to Kafka.
- **Consumer**:
  - Reads the sales data from Kafka.
  - Deserializes the data and processes it for real-time analytics (e.g., calculating total sales, generating reports).
- **Infrastructure**:
  - The Docker Compose setup provides a scalable and fault-tolerant Kafka cluster.
  - The Schema Registry ensures that the data format is consistent across producers and consumers.

---

### **Summary**
- **Producer**: Reads data from a CSV file, serializes it using Avro, and sends it to Kafka.
- **Consumer**: Reads data from Kafka, deserializes it using Avro, and processes it.
- **Docker Compose**: Provides the infrastructure (Zookeeper, Kafka Broker, Schema Registry) for the system.
- **Avro Schema**: Defines the structure of the data being exchanged in Kafka.

This setup is a robust and scalable solution for real-time data processing using Confluent Kafka.

<br/>
<br/>

## Let’s break down and explain the **code** from the provided files (`confluent_avro_data_consumer.py`, `confluent_avro_data_producer.py`) and the **Docker Compose configuration** (`docker_compose_confluent_kafka.yml`). We'll also explain the **Avro schema** (`reatil_data_avro_schema.json`).

---

### **1. `confluent_avro_data_consumer.py`**

#### **Code**:
```python
import threading
from confluent_kafka import DeserializingConsumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from confluent_kafka.serialization import StringDeserializer

# Define Kafka configuration
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc',
    'group.id': 'group1',
    'auto.offset.reset': 'earliest'
}

# Create a Schema Registry client
schema_registry_client = SchemaRegistryClient({
  'url': 'hots',
  'basic.auth.user.info': '{}:{}'.format('klm', 'abc123')
})

# Fetch the latest Avro schema for the value
subject_name = 'retail_data_test-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str

# Create Avro Deserializer for the value
key_deserializer = StringDeserializer('utf_8')
avro_deserializer = AvroDeserializer(schema_registry_client, schema_str)

# Define the DeserializingConsumer
consumer = DeserializingConsumer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.deserializer': key_deserializer,
    'value.deserializer': avro_deserializer,
    'group.id': kafka_config['group.id'],
    'auto.offset.reset': kafka_config['auto.offset.reset']
})

# Subscribe to the 'retail_data' topic
consumer.subscribe(['retail_data_test'])

# Continually read messages from Kafka
try:
    while True:
        msg = consumer.poll(1.0)  # Wait for message (1 second timeout)

        if msg is None:
            continue
        if msg.error():
            print('Consumer error: {}'.format(msg.error()))
            continue

        print('Successfully consumed record with key {} and value {}'.format(msg.key(), msg.value()))

except KeyboardInterrupt:
    pass
finally:
    consumer.close()
```

#### **Explanation**:
- **Kafka Configuration**:
  - The consumer connects to a Kafka broker using SASL/SSL authentication.
  - The `group.id` specifies the consumer group, and `auto.offset.reset` ensures the consumer starts reading from the earliest available message if no offset is committed.
- **Schema Registry**:
  - The consumer fetches the latest Avro schema for the topic `retail_data_test` from the Schema Registry.
  - The schema is used to deserialize the Avro-encoded messages.
- **DeserializingConsumer**:
  - The consumer subscribes to the `retail_data_test` topic.
  - It polls for messages, deserializes them using the Avro schema, and prints the key and value of each message.
- **Error Handling**:
  - If an error occurs (e.g., network issues), the consumer logs the error and continues polling.
- **Graceful Shutdown**:
  - The consumer closes gracefully when the script is interrupted (e.g., via `Ctrl+C`).

---

### **2. `confluent_avro_data_producer.py`**

#### **Code**:
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

def delivery_report(err, msg):
    """
    Reports the failure or success of a message delivery.
    """
    if err is not None:
        print("Delivery failed for User record {}: {}".format(msg.key(), err))
        return
    print('User record {} successfully produced to {} [{}] at offset {}'.format(
        msg.key(), msg.topic(), msg.partition(), msg.offset()))

# Define Kafka configuration
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc'
}

# Create a Schema Registry client
schema_registry_client = SchemaRegistryClient({
  'url': 'hots',
  'basic.auth.user.info': '{}:{}'.format('klm', 'abc123')
})

# Fetch the latest Avro schema for the value
subject_name = 'retail_data_test-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str

# Create Avro Serializer for the value
key_serializer = StringSerializer('utf_8')
avro_serializer = AvroSerializer(schema_registry_client, schema_str)

# Define the SerializingProducer
producer = SerializingProducer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.serializer': key_serializer,  # Key will be serialized as a string
    'value.serializer': avro_serializer  # Value will be serialized as Avro
})

# Load the CSV data into a pandas DataFrame
df = pd.read_csv('retail_data.csv')
df = df.fillna('null')

# Iterate over DataFrame rows and produce to Kafka
for index, row in df.iterrows():
    # Create a dictionary from the row values
    value = row.to_dict()
    # Produce to Kafka
    producer.produce(topic='retail_data_test', key=str(index), value=value, on_delivery=delivery_report)
    producer.flush()
    time.sleep(2)

print("All Data successfully published to Kafka")
```

#### **Explanation**:
- **Kafka Configuration**:
  - The producer connects to a Kafka broker using SASL/SSL authentication.
- **Schema Registry**:
  - The producer fetches the latest Avro schema for the topic `retail_data_test` from the Schema Registry.
  - The schema is used to serialize the data before sending it to Kafka.
- **SerializingProducer**:
  - The producer reads data from a CSV file (`retail_data.csv`).
  - Each row of data is serialized using the Avro schema and sent to the `retail_data_test` topic.
  - The `delivery_report` callback logs the success or failure of each message delivery.
- **Error Handling**:
  - If a message fails to be delivered, the producer logs the error.
- **Graceful Shutdown**:
  - The producer flushes its buffer to ensure all messages are sent before exiting.

---

### **3. `docker_compose_confluent_kafka.yml`**

#### **Code**:
```yaml
version: '3.9'

networks:
  gds:
    name: gds

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    networks:
      - gds
    volumes:
      - /Users/shivamvishwakarma/Desktop/temp_kafka/zk-data:/var/lib/zookeeper/data
      - /Users/shivamvishwakarma/Desktop/temp_kafka/zk-txn-logs:/var/lib/zookeeper/log
    tmpfs: "/datalog"

  broker:
    image: confluentinc/cp-server:latest
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9101:9101"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://host.docker.internal:9092
      KAFKA_METRIC_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: host.docker.internal
      KAFKA_CONFLUENT_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: broker:29092
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
      CONFLUENT_METRICS_ENABLE: 'true'
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'
    networks:
      - gds
    restart: on-failure
    volumes:
      - /Users/shivamvishwakarma/Desktop/temp_kafka/kafka-data:/var/lib/kafka/data

  schema-registry:
    image: confluentinc/cp-schema-registry:latest
    hostname: schema-registry
    container_name: schema-registry
    depends_on:
      - broker
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'broker:29092'
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8081
    networks:
      - gds
    restart: on-failure

  control-center:
    image: confluentinc/cp-enterprise-control-center:latest
    hostname: control-center
    container_name: control-center
    depends_on:
      - broker
      - schema-registry
    ports:
      - "9021:9021"
    environment:
      CONTROL_CENTER_BOOTSTRAP_SERVERS: 'broker:29092'
      CONTROL_CENTER_CONNECT_CONNECT-DEFAULT_CLUSTER: 'connect:8083'
      CONTROL_CENTER_SCHEMA_REGISTRY_URL: "http://schema-registry:8081"
      CONTROL_CENTER_REPLICATION_FACTOR: 1
      CONTROL_CENTER_INTERNAL_TOPICS_PARTITIONS: 1
      CONTROL_CENTER_MONITORING_INTERCEPTOR_TOPIC_PARTITIONS: 1
      CONFLUENT_METRICS_TOPIC_REPLICATION: 1
      PORT: 9021
    networks:
      - gds
    restart: on-failure

  connect:
    image: cnfldemos/cp-server-connect-datagen:0.5.3-7.1.0
    hostname: connect
    container_name: connect
    volumes:
      - /Users/shivamvishwakarma/Desktop/temp_kafka/connect-jars:/usr/share/confluent-hub-components
    depends_on:
      - broker
      - schema-registry
    ports:
      - "8083:8083"
    environment:
      CONNECT_BOOTSTRAP_SERVERS: 'broker:29092'
      CONNECT_REST_ADVERTISED_HOST_NAME: connect
      CONNECT_GROUP_ID: compose-connect-group
      CONNECT_CONFIG_STORAGE_TOPIC: docker-connect-configs
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_FLUSH_INTERVAL_MS: 10000
      CONNECT_OFFSET_STORAGE_TOPIC: docker-connect-offsets
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_TOPIC: docker-connect-status
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.storage.StringConverter
      CONNECT_VALUE_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONNECT_CONNECTOR_CLIENT_CONFIG_OVERRIDE_POLICY: 'All'
      CLASSPATH: /usr/share/java/monitoring-interceptors/monitoring-interceptors-7.0.1.jar
      CONNECT_PRODUCER_INTERCEPTOR_CLASSES: "io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor"
      CONNECT_CONSUMER_INTERCEPTOR_CLASSES: "io.confluent.monitoring.clients.interceptor.MonitoringConsumerInterceptor"
      CONNECT_PLUGIN_PATH: "/usr/share/java,/usr/share/confluent-hub-components"
      CONNECT_LOG4J_LOGGERS: org.apache.zookeeper=ERROR,org.I0Itec.zkclient=ERROR,org.reflections=ERROR
    networks:
      - gds
```

#### **Explanation**:
- **Zookeeper**:
  - Manages Kafka broker coordination and metadata.
  - Exposes port `2181` for client connections.
- **Kafka Broker**:
  - Handles message storage and delivery.
  - Exposes port `9092` for client connections and `9101` for JMX monitoring.
- **Schema Registry**:
  - Manages Avro schemas for Kafka topics.
  - Exposes port `8081` for client connections.
- **Control Center**:
  - Provides a web-based UI for monitoring and managing the Kafka cluster.
  - Exposes port `9021` for the web interface.
- **Kafka Connect**:
  - Integrates Kafka with external systems (e.g., databases, cloud services).
  - Exposes port `8083` for REST API connections.

---

### **4. `reatil_data_avro_schema.json`**

#### **Code**:
```json
{
  "namespace": "com.kaggle.onlineretail",
  "name": "Purchase",
  "type": "record",
  "fields": [
    {
      "name": "Invoice",
      "type": "string"
    },
    {
      "name": "StockCode",
      "type": "string"
    },
    {
      "name": "Description",
      "type": ["null", "string"],
      "default": null
    },
    {
      "name": "Quantity",
      "type": "int"
    },
    {
      "name": "InvoiceDate",
      "type": "string"
    },
    {
      "name": "Price",
      "type": "double"
    },
    {
      "name": "CustomerID",
      "type": ["null", "string"],
      "default": null
    },
    {
      "name": "Country",
      "type": "string"
    }
  ]
}
```

#### **Explanation**:
- **Namespace**: `com.kaggle.onlineretail` (logical grouping for the schema).
- **Name**: `Purchase` (name of the schema).
- **Type**: `record` (defines a structured record with multiple fields).
- **Fields**:
  - `Invoice`: A string representing the invoice number.
  - `StockCode`: A string representing the stock code.
  - `Description`: A nullable string representing the product description.
  - `Quantity`: An integer representing the quantity purchased.
  - `InvoiceDate`: A string representing the invoice date.
  - `Price`: A double representing the price.
  - `CustomerID`: A nullable string representing the customer ID.
  - `Country`: A string representing the country.

---

### **Summary**
- **Consumer**: Reads Avro-serialized data from Kafka and deserializes it using the schema.
- **Producer**: Reads data from a CSV file, serializes it using Avro, and sends it to Kafka.
- **Docker Compose**: Provides the infrastructure (Zookeeper, Kafka Broker, Schema Registry) for the system.
- **Avro Schema**: Defines the structure of the data being exchanged in Kafka.

This setup is a robust and scalable solution for real-time data processing using Confluent Kafka.


<br/>
<br/>

## **`confluent_avro_data_producer.py`**
Let’s dive into the **`confluent_avro_data_producer.py`** script in detail. This script is responsible for **producing Avro-serialized data** to a Kafka topic. It reads data from a CSV file, serializes it using an Avro schema, and sends it to a Kafka topic. Below is a step-by-step explanation of the code:

---

### **1. Imports**
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

#### **Explanation**:
- **`confluent_kafka`**: This library is used to interact with Kafka and the Schema Registry.
  - `SerializingProducer`: A Kafka producer that serializes data before sending it to Kafka.
  - `SchemaRegistryClient`: Connects to the Schema Registry to fetch Avro schemas.
  - `AvroSerializer`: Serializes data into Avro format using the schema.
  - `StringSerializer`: Serializes keys into strings.
- **`pandas`**: Used to read and process the CSV file containing the data to be sent to Kafka.

---

### **2. Delivery Report Callback**
```python
def delivery_report(err, msg):
    """
    Reports the failure or success of a message delivery.
    """
    if err is not None:
        print("Delivery failed for User record {}: {}".format(msg.key(), err))
        return
    print('User record {} successfully produced to {} [{}] at offset {}'.format(
        msg.key(), msg.topic(), msg.partition(), msg.offset()))
```

#### **Explanation**:
- This function is called when a message is successfully delivered to Kafka or if delivery fails.
- **Parameters**:
  - `err`: Contains the error if delivery fails, or `None` if successful.
  - `msg`: The Kafka message object, containing details like the topic, partition, and offset.
- **Behavior**:
  - If delivery fails, the error is logged.
  - If delivery succeeds, the success message is logged, including the topic, partition, and offset.

---

### **3. Kafka Configuration**
```python
# Define Kafka configuration
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc'
}
```

#### **Explanation**:
- **`bootstrap.servers`**: The address of the Kafka broker(s).
- **`sasl.mechanisms`**: Specifies the SASL mechanism for authentication (e.g., `PLAIN`).
- **`security.protocol`**: Specifies the security protocol (e.g., `SASL_SSL` for secure communication).
- **`sasl.username` and `sasl.password`**: Credentials for SASL authentication.

---

### **4. Schema Registry Client**
```python
# Create a Schema Registry client
schema_registry_client = SchemaRegistryClient({
  'url': 'hots',
  'basic.auth.user.info': '{}:{}'.format('klm', 'abc123')
})
```

#### **Explanation**:
- The `SchemaRegistryClient` connects to the Schema Registry to fetch Avro schemas.
- **`url`**: The URL of the Schema Registry.
- **`basic.auth.user.info`**: Credentials for authenticating with the Schema Registry.

---

### **5. Fetch Avro Schema**
```python
# Fetch the latest Avro schema for the value
subject_name = 'retail_data_test-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str
```

#### **Explanation**:
- **`subject_name`**: The name of the schema subject (typically `<topic-name>-value`).
- **`get_latest_version`**: Fetches the latest version of the schema for the specified subject.
- **`schema_str`**: The schema in string format, which will be used to serialize the data.

---

### **6. Create Serializers**
```python
# Create Avro Serializer for the value
key_serializer = StringSerializer('utf_8')
avro_serializer = AvroSerializer(schema_registry_client, schema_str)
```

#### **Explanation**:
- **`key_serializer`**: Serializes the message key into a string.
- **`avro_serializer`**: Serializes the message value into Avro format using the fetched schema.

---

### **7. Define the SerializingProducer**
```python
# Define the SerializingProducer
producer = SerializingProducer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.serializer': key_serializer,  # Key will be serialized as a string
    'value.serializer': avro_serializer  # Value will be serialized as Avro
})
```

#### **Explanation**:
- The `SerializingProducer` is configured with:
  - Kafka broker details.
  - Security settings.
  - Serializers for the key and value.

---

### **8. Load CSV Data**
```python
# Load the CSV data into a pandas DataFrame
df = pd.read_csv('retail_data.csv')
df = df.fillna('null')
```

#### **Explanation**:
- **`pd.read_csv`**: Reads the CSV file into a pandas DataFrame.
- **`fillna('null')`**: Replaces missing values with `'null'` to ensure compatibility with the Avro schema.

---

### **9. Produce Data to Kafka**
```python
# Iterate over DataFrame rows and produce to Kafka
for index, row in df.iterrows():
    # Create a dictionary from the row values
    value = row.to_dict()
    # Produce to Kafka
    producer.produce(topic='retail_data_test', key=str(index), value=value, on_delivery=delivery_report)
    producer.flush()
    time.sleep(2)

print("All Data successfully published to Kafka")
```

#### **Explanation**:
- **`df.iterrows()`**: Iterates over each row in the DataFrame.
- **`row.to_dict()`**: Converts the row into a dictionary, which is serialized using the Avro schema.
- **`producer.produce`**:
  - Sends the message to the `retail_data_test` topic.
  - The key is the row index (serialized as a string).
  - The value is the row data (serialized as Avro).
  - The `delivery_report` callback is invoked after each message is delivered.
- **`producer.flush()`**: Ensures that all messages are sent to Kafka before proceeding.
- **`time.sleep(2)`**: Introduces a delay between messages (optional, for demonstration purposes).

---

### **10. Summary of Workflow**
1. **Setup**:
   - Configure Kafka and Schema Registry clients.
   - Fetch the Avro schema for the topic.
2. **Data Preparation**:
   - Read data from a CSV file into a pandas DataFrame.
   - Replace missing values with `'null'`.
3. **Produce Messages**:
   - Iterate over each row in the DataFrame.
   - Serialize the row data using the Avro schema.
   - Send the serialized data to the Kafka topic.
4. **Delivery Reporting**:
   - Log the success or failure of each message delivery.

---

### **11. Example Use Case**
Imagine a retail analytics system:
- **CSV File**: Contains sales data (e.g., invoice number, stock code, quantity, price).
- **Kafka Topic**: `retail_data_test` is used to stream the sales data.
- **Producer**:
  - Reads the CSV file.
  - Serializes each row of data using the Avro schema.
  - Sends the data to the Kafka topic for real-time processing.

---

### **12. Key Points**
- **Avro Serialization**: Ensures that the data is structured and consistent.
- **Schema Registry**: Manages the Avro schema and ensures compatibility between producers and consumers.
- **Delivery Reporting**: Provides feedback on message delivery success or failure.
- **Scalability**: The producer can handle large datasets and high-throughput systems.

This script is a robust solution for producing structured, schema-compliant data to Kafka, making it ideal for real-time 
data pipelines and analytics systems.

<br/>
<br/>

# **Flow of Producer**

When you run the **`confluent_avro_data_producer.py`** script, it interacts with **Confluent Kafka** and the **Schema Registry** to produce Avro-serialized data to a Kafka topic. Here’s a step-by-step breakdown of what happens internally when you execute the code:

---

### **1. Initialization**
- **Kafka Configuration**:
  - The script initializes the Kafka producer with the provided configuration (e.g., broker address, security settings).
  - It connects to the Kafka broker using the specified credentials (`sasl.username`, `sasl.password`).
- **Schema Registry**:
  - The script connects to the **Schema Registry** using the provided URL and credentials.
  - It fetches the latest Avro schema for the topic `retail_data_test-value`.

---

### **2. Data Preparation**
- **CSV File**:
  - The script reads the CSV file (`retail_data.csv`) into a pandas DataFrame.
  - Missing values in the DataFrame are replaced with `'null'` to ensure compatibility with the Avro schema.
- **Row Iteration**:
  - The script iterates over each row in the DataFrame, converting each row into a dictionary.

---

### **3. Serialization**
- **Key Serialization**:
  - The row index is serialized as a string using the `StringSerializer`.
- **Value Serialization**:
  - The row data (dictionary) is serialized into Avro format using the `AvroSerializer` and the fetched schema.
  - The serialized data includes a **schema ID**, which references the schema stored in the Schema Registry.

---

### **4. Producing Messages**
- **Kafka Topic**:
  - The script sends the serialized data to the Kafka topic `retail_data_test`.
  - Each message includes:
    - A **key** (row index, serialized as a string).
    - A **value** (row data, serialized as Avro).
- **Delivery Report**:
  - After each message is sent, the `delivery_report` callback is invoked.
    - If the message is successfully delivered, the callback logs the topic, partition, and offset.
    - If delivery fails, the callback logs the error.

---

### **5. Flushing and Completion**
- **Flush**:
  - The script calls `producer.flush()` after producing each message to ensure that the message is sent to Kafka immediately.
- **Completion**:
  - Once all rows in the CSV file have been processed, the script prints a success message: `"All Data successfully published to Kafka"`.

---

### **6. Internal Workflow of Confluent Kafka**
When the script interacts with Confluent Kafka, the following internal processes occur:

#### **a. Kafka Broker**
- The Kafka broker receives the serialized messages from the producer.
- It stores the messages in the specified topic (`retail_data_test`).
- The broker ensures that the messages are replicated across multiple partitions (if configured) for fault tolerance.

#### **b. Schema Registry**
- The Schema Registry stores the Avro schema for the topic.
- When the producer serializes the data, it embeds the **schema ID** in the message.
- Consumers can fetch the schema from the Schema Registry using the schema ID to deserialize the data.

#### **c. Serialization**
- The producer serializes the data into a compact binary format (Avro) before sending it to Kafka.
- This reduces the size of the messages and ensures that the data is structured and consistent.

#### **d. Delivery Guarantees**
- The producer ensures that messages are delivered to Kafka with the specified delivery guarantees (e.g., at least once, exactly once).
- The `delivery_report` callback provides feedback on the success or failure of each message delivery.

---

### **7. Example Output**
When you run the script, you will see output similar to the following:

```
User record 0 successfully produced to retail_data_test [0] at offset 42
User record 1 successfully produced to retail_data_test [1] at offset 43
User record 2 successfully produced to retail_data_test [0] at offset 44
...
All Data successfully published to Kafka
```

- Each line indicates that a message was successfully produced to the Kafka topic.
- The key (row index), topic, partition, and offset are logged for each message.

---

### **8. What Happens Behind the Scenes**
- **Data Flow**:
  1. The producer reads data from the CSV file.
  2. It serializes the data using the Avro schema fetched from the Schema Registry.
  3. It sends the serialized data to the Kafka topic.
  4. The Kafka broker stores the data in the topic.
- **Schema Management**:
  - The Schema Registry ensures that the producer and consumers use the same schema for serialization and deserialization.
  - If the schema evolves (e.g., new fields are added), the Schema Registry ensures backward and forward compatibility.

---

### **9. Key Benefits of Using Confluent Kafka**
- **Schema Enforcement**: Ensures that all messages adhere to a defined schema, improving data quality.
- **Scalability**: Kafka can handle high volumes of data and scale horizontally.
- **Fault Tolerance**: Kafka replicates data across multiple brokers, ensuring no data is lost during failures.
- **Real-Time Processing**: Kafka enables low-latency data processing, making it ideal for real-time use cases.

---

### **10. Example Use Case**
Imagine a retail analytics system:
- **CSV File**: Contains sales data (e.g., invoice number, stock code, quantity, price).
- **Kafka Topic**: `retail_data_test` is used to stream the sales data.
- **Producer**:
  - Reads the CSV file.
  - Serializes each row of data using the Avro schema.
  - Sends the data to the Kafka topic for real-time processing.
- **Consumer**:
  - Reads the data from the Kafka topic.
  - Deserializes the data using the Avro schema.
  - Processes the data for real-time analytics (e.g., calculating total sales, generating reports).

---

### **Summary**
When you run the **`confluent_avro_data_producer.py`** script:
1. It connects to Kafka and the Schema Registry.
2. It reads data from a CSV file and serializes it using the Avro schema.
3. It sends the serialized data to the Kafka topic.
4. It logs the success or failure of each message delivery.

This script is a robust solution for producing structured, schema-compliant data to Kafka, making it ideal for real-time data pipelines and analytics systems.


<br/>
<br/>

## **`confluent_avro_data_consumer.py`**
Let’s dive into the **`confluent_avro_data_consumer.py`** script in detail. This script is responsible for **consuming Avro-serialized data** from a Kafka topic. It connects to Kafka, fetches the Avro schema from the Schema Registry, deserializes the data, and processes it. Below is a step-by-step explanation of the code:

---

### **1. Imports**
```python
import threading
from confluent_kafka import DeserializingConsumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from confluent_kafka.serialization import StringDeserializer
```

#### **Explanation**:
- **`confluent_kafka`**: This library is used to interact with Kafka and the Schema Registry.
  - `DeserializingConsumer`: A Kafka consumer that deserializes data after consuming it from Kafka.
  - `SchemaRegistryClient`: Connects to the Schema Registry to fetch Avro schemas.
  - `AvroDeserializer`: Deserializes Avro-encoded data using the schema.
  - `StringDeserializer`: Deserializes keys into strings.
- **`threading`**: Used for multi-threading (though not explicitly used in this script).

---

### **2. Kafka Configuration**
```python
# Define Kafka configuration
kafka_config = {
    'bootstrap.servers': 'host:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': 'XYZ',
    'sasl.password': 'abc',
    'group.id': 'group1',
    'auto.offset.reset': 'earliest'
}
```

#### **Explanation**:
- **`bootstrap.servers`**: The address of the Kafka broker(s).
- **`sasl.mechanisms`**: Specifies the SASL mechanism for authentication (e.g., `PLAIN`).
- **`security.protocol`**: Specifies the security protocol (e.g., `SASL_SSL` for secure communication).
- **`sasl.username` and `sasl.password`**: Credentials for SASL authentication.
- **`group.id`**: Specifies the consumer group ID. All consumers in the same group share the workload of consuming messages.
- **`auto.offset.reset`**: Specifies what to do if there is no initial offset in Kafka or if the current offset is invalid. Setting it to `earliest` means the consumer will start reading from the earliest available message.

---

### **3. Schema Registry Client**
```python
# Create a Schema Registry client
schema_registry_client = SchemaRegistryClient({
  'url': 'hots',
  'basic.auth.user.info': '{}:{}'.format('klm', 'abc123')
})
```

#### **Explanation**:
- The `SchemaRegistryClient` connects to the Schema Registry to fetch Avro schemas.
- **`url`**: The URL of the Schema Registry.
- **`basic.auth.user.info`**: Credentials for authenticating with the Schema Registry.

---

### **4. Fetch Avro Schema**
```python
# Fetch the latest Avro schema for the value
subject_name = 'retail_data_test-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str
```

#### **Explanation**:
- **`subject_name`**: The name of the schema subject (typically `<topic-name>-value`).
- **`get_latest_version`**: Fetches the latest version of the schema for the specified subject.
- **`schema_str`**: The schema in string format, which will be used to deserialize the data.

---

### **5. Create Deserializers**
```python
# Create Avro Deserializer for the value
key_deserializer = StringDeserializer('utf_8')
avro_deserializer = AvroDeserializer(schema_registry_client, schema_str)
```

#### **Explanation**:
- **`key_deserializer`**: Deserializes the message key into a string.
- **`avro_deserializer`**: Deserializes the message value from Avro format using the fetched schema.

---

### **6. Define the DeserializingConsumer**
```python
# Define the DeserializingConsumer
consumer = DeserializingConsumer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.deserializer': key_deserializer,
    'value.deserializer': avro_deserializer,
    'group.id': kafka_config['group.id'],
    'auto.offset.reset': kafka_config['auto.offset.reset']
})
```

#### **Explanation**:
- The `DeserializingConsumer` is configured with:
  - Kafka broker details.
  - Security settings.
  - Deserializers for the key and value.
  - Consumer group ID and offset reset policy.

---

### **7. Subscribe to Kafka Topic**
```python
# Subscribe to the 'retail_data' topic
consumer.subscribe(['retail_data_test'])
```

#### **Explanation**:
- The consumer subscribes to the `retail_data_test` topic to start consuming messages.

---

### **8. Poll for Messages**
```python
# Continually read messages from Kafka
try:
    while True:
        msg = consumer.poll(1.0)  # Wait for message (1 second timeout)

        if msg is None:
            continue
        if msg.error():
            print('Consumer error: {}'.format(msg.error()))
            continue

        print('Successfully consumed record with key {} and value {}'.format(msg.key(), msg.value()))

except KeyboardInterrupt:
    pass
finally:
    consumer.close()
```

#### **Explanation**:
- **Polling**:
  - The consumer polls for messages from the Kafka topic with a timeout of 1 second.
  - If no message is received within the timeout, it continues polling.
- **Error Handling**:
  - If an error occurs (e.g., network issues), the error is logged, and the consumer continues polling.
- **Message Processing**:
  - If a message is successfully consumed, the key and value are deserialized and printed.
- **Graceful Shutdown**:
  - If the script is interrupted (e.g., via `Ctrl+C`), the consumer closes gracefully.

---

### **9. Internal Workflow of Confluent Kafka**
When the script interacts with Confluent Kafka, the following internal processes occur:

#### **a. Kafka Broker**
- The Kafka broker delivers messages from the `retail_data_test` topic to the consumer.
- The consumer fetches messages from the assigned partitions.

#### **b. Schema Registry**
- The consumer fetches the Avro schema from the Schema Registry using the schema ID embedded in the message.
- The schema is used to deserialize the message value.

#### **c. Deserialization**
- The consumer deserializes the message key into a string.
- The consumer deserializes the message value from Avro format into a Python dictionary using the fetched schema.

#### **d. Offset Management**
- The consumer tracks its position (offset) in each partition.
- If the consumer restarts, it resumes reading from the last committed offset.

---

### **10. Example Output**
When you run the script, you will see output similar to the following:

```
Successfully consumed record with key 0 and value {'Invoice': '123', 'StockCode': 'ABC', 'Description': 'Product A', 'Quantity': 2, 'InvoiceDate': '2023-10-01', 'Price': 10.5, 'CustomerID': 'CUST001', 'Country': 'USA'}
Successfully consumed record with key 1 and value {'Invoice': '124', 'StockCode': 'XYZ', 'Description': 'Product B', 'Quantity': 1, 'InvoiceDate': '2023-10-02', 'Price': 15.0, 'CustomerID': 'CUST002', 'Country': 'UK'}
...
```

- Each line indicates that a message was successfully consumed from the Kafka topic.
- The key (row index) and value (deserialized data) are printed.

---

### **11. Key Benefits of Using Confluent Kafka**
- **Schema Enforcement**: Ensures that all messages adhere to a defined schema, improving data quality.
- **Scalability**: Kafka can handle high volumes of data and scale horizontally.
- **Fault Tolerance**: Kafka replicates data across multiple brokers, ensuring no data is lost during failures.
- **Real-Time Processing**: Kafka enables low-latency data processing, making it ideal for real-time use cases.

---

### **12. Example Use Case**
Imagine a retail analytics system:
- **Kafka Topic**: `retail_data_test` contains sales data (e.g., invoice number, stock code, quantity, price).
- **Consumer**:
  - Reads the data from the Kafka topic.
  - Deserializes the data using the Avro schema.
  - Processes the data for real-time analytics (e.g., calculating total sales, generating reports).

---

### **Summary**
When you run the **`confluent_avro_data_consumer.py`** script:
1. It connects to Kafka and the Schema Registry.
2. It subscribes to the `retail_data_test` topic.
3. It polls for messages, deserializes them using the Avro schema, and prints the key and value.
4. It handles errors gracefully and closes the consumer on interruption.

This script is a robust solution for consuming structured, schema-compliant data from Kafka, making it ideal for real-time data pipelines and analytics systems.


<br/>
<br/>

# **Flow of Consumer**

When you run the **`confluent_avro_data_consumer.py`** script, it interacts with **Confluent Kafka** and the **Schema Registry** to consume Avro-serialized data from a Kafka topic. Here’s a step-by-step breakdown of what happens internally when you execute the code:

---

### **1. Initialization**
- **Kafka Configuration**:
  - The script initializes the Kafka consumer with the provided configuration (e.g., broker address, security settings).
  - It connects to the Kafka broker using the specified credentials (`sasl.username`, `sasl.password`).
- **Schema Registry**:
  - The script connects to the **Schema Registry** using the provided URL and credentials.
  - It fetches the latest Avro schema for the topic `retail_data_test-value`.

---

### **2. Consumer Setup**
- **Deserializers**:
  - The script sets up deserializers for the message key (`StringDeserializer`) and value (`AvroDeserializer`).
  - The `AvroDeserializer` uses the fetched schema to deserialize the Avro-encoded messages.
- **Consumer Group**:
  - The consumer is part of a consumer group (`group.id = 'group1'`), which allows multiple consumers to share the workload of consuming messages.
- **Offset Reset**:
  - The `auto.offset.reset` setting is set to `earliest`, meaning the consumer will start reading from the earliest available message if no offset is committed.

---

### **3. Subscription**
- The consumer subscribes to the Kafka topic `retail_data_test` to start consuming messages.

---

### **4. Polling for Messages**
- The consumer enters a loop where it continually polls for messages from the Kafka topic.
- **Polling Behavior**:
  - The consumer waits for messages with a timeout of 1 second (`poll(1.0)`).
  - If no message is received within the timeout, it continues polling.
- **Message Processing**:
  - When a message is received, the consumer deserializes the key and value using the configured deserializers.
  - The deserialized key and value are printed to the console.

---

### **5. Error Handling**
- If an error occurs (e.g., network issues, deserialization errors), the consumer logs the error and continues polling.
- If the consumer is interrupted (e.g., via `Ctrl+C`), it closes gracefully.

---

### **6. Internal Workflow of Confluent Kafka**
When the script interacts with Confluent Kafka, the following internal processes occur:

#### **a. Kafka Broker**
- The Kafka broker delivers messages from the `retail_data_test` topic to the consumer.
- The consumer fetches messages from the assigned partitions.

#### **b. Schema Registry**
- The consumer fetches the Avro schema from the Schema Registry using the schema ID embedded in the message.
- The schema is used to deserialize the message value.

#### **c. Deserialization**
- The consumer deserializes the message key into a string.
- The consumer deserializes the message value from Avro format into a Python dictionary using the fetched schema.

#### **d. Offset Management**
- The consumer tracks its position (offset) in each partition.
- If the consumer restarts, it resumes reading from the last committed offset.

---

### **7. Example Output**
When you run the script, you will see output similar to the following:

```
Successfully consumed record with key 0 and value {'Invoice': '123', 'StockCode': 'ABC', 'Description': 'Product A', 'Quantity': 2, 'InvoiceDate': '2023-10-01', 'Price': 10.5, 'CustomerID': 'CUST001', 'Country': 'USA'}
Successfully consumed record with key 1 and value {'Invoice': '124', 'StockCode': 'XYZ', 'Description': 'Product B', 'Quantity': 1, 'InvoiceDate': '2023-10-02', 'Price': 15.0, 'CustomerID': 'CUST002', 'Country': 'UK'}
...
```

- Each line indicates that a message was successfully consumed from the Kafka topic.
- The key (row index) and value (deserialized data) are printed.

---

### **8. Key Benefits of Using Confluent Kafka**
- **Schema Enforcement**: Ensures that all messages adhere to a defined schema, improving data quality.
- **Scalability**: Kafka can handle high volumes of data and scale horizontally.
- **Fault Tolerance**: Kafka replicates data across multiple brokers, ensuring no data is lost during failures.
- **Real-Time Processing**: Kafka enables low-latency data processing, making it ideal for real-time use cases.

---

### **9. Example Use Case**
Imagine a retail analytics system:
- **Kafka Topic**: `retail_data_test` contains sales data (e.g., invoice number, stock code, quantity, price).
- **Consumer**:
  - Reads the data from the Kafka topic.
  - Deserializes the data using the Avro schema.
  - Processes the data for real-time analytics (e.g., calculating total sales, generating reports).

---

### **Summary**
When you run the **`confluent_avro_data_consumer.py`** script:
1. It connects to Kafka and the Schema Registry.
2. It subscribes to the `retail_data_test` topic.
3. It polls for messages, deserializes them using the Avro schema, and prints the key and value.
4. It handles errors gracefully and closes the consumer on interruption.

This script is a robust solution for consuming structured, schema-compliant data from Kafka, making it ideal for real-time data pipelines and analytics systems.

<br/>
<br/>


# **Scenario: Running Producer Once and Consumer Code 3 Times with Different Consumer Groups**

Your setup consists of:
1. **Producer (`confluent_avro_data_producer.py`)**  
   - Produces messages to **`retail_data_test`** topic.  
   - Uses **Avro serialization** with Schema Registry.  
   - Produces records from a CSV file.  

2. **Consumers (`confluent_avro_data_consumer.py`)**  
   - Three instances of the consumer will run, each with a different `group.id`:  
     - `group1`
     - `group2`
     - `group3`  
   - Uses **Avro deserialization** to read messages.

---

## **What Happens When You Run the Producer Once?**
- The producer will publish all messages **once** to **`retail_data_test`** topic.
- These messages are stored in Kafka’s **partitions** (assuming 2 partitions by default).

---

## **What Happens When You Run the Consumer 3 Times?**

Since each consumer instance has a **different consumer group (`group.id`)**, the behavior will be:

1. **Each consumer group gets an independent copy of the messages.**
   - Kafka **does not share partitions across different consumer groups**.
   - So, each consumer group (`group1`, `group2`, `group3`) will **consume the entire topic separately**.

2. **Messages are delivered three times (once per group).**
   - Each consumer group will consume messages **independently** from the beginning (`auto.offset.reset = 'earliest'`).
   - All three consumer groups will receive the **same** messages from Kafka.

3. **Even if partitions are limited, each group gets a full copy.**
   - The number of partitions affects **parallelism** within a group, **not across groups**.
   - Since your setup has 2 partitions:
     - If a group has more than 2 consumers, some will stay idle.
     - But across groups, each will still get a full message set.

---

## **Expected Behavior**
| Consumer Group | Messages Received |
|---------------|------------------|
| `group1`      | All messages from topic |
| `group2`      | All messages from topic |
| `group3`      | All messages from topic |

- Each consumer group gets **all** messages independently.
- This is useful when **multiple applications need to process the same data differently**.

---

## **Conclusion**
✅ **Kafka ensures that each consumer group gets its own copy of data.**  
✅ **Running 3 consumer instances with different `group.id`s means each will receive all messages.**  
✅ **This is useful for independent processing workflows (e.g., analytics, monitoring, logging).**  

<br/>
<br/>

# **Scenario: Two Partitions, Two Consumer Groups, and One Group with Three Consumers**  

## **Given Setup:**
- **Topic:** `retail_data_test`
- **Partitions:** 2 (`P0`, `P1`)
- **Consumer Groups:**
  - **Group1 (`group1`)** → Has **1 consumer** (`C1`)
  - **Group2 (`group2`)** → Has **3 consumers** (`C2.1`, `C2.2`, `C2.3`)

---

## **How Kafka Distributes Messages?**
1. **Partitions distribute messages among consumers within a group.**
2. **Each consumer group consumes messages independently.**
3. **If a group has more consumers than partitions, extra consumers remain idle.**

---

## **Message Flow & Consumer Assignment**
### **Consumer Group 1 (`group1` → 1 Consumer)**
| Partition | Assigned Consumer |
|-----------|------------------|
| `P0`      | `C1` |
| `P1`      | `C1` |

✅ **Since `group1` has only one consumer (`C1`), it will consume messages from both partitions (`P0` & `P1`).**

---

### **Consumer Group 2 (`group2` → 3 Consumers)**
| Partition | Assigned Consumer |
|-----------|------------------|
| `P0`      | `C2.1` |
| `P1`      | `C2.2` |
| -         | `C2.3` (Idle) |

❌ **One consumer (`C2.3`) will remain idle** because Kafka can only assign **2 partitions to 2 active consumers**.

---

## **Expected Behavior**
| Consumer Group | Consumers | Partition Assignment | Messages Received |
|---------------|----------|----------------------|-------------------|
| `group1` | `C1` | `P0`, `P1` | All messages |
| `group2` | `C2.1`, `C2.2` (Active), `C2.3` (Idle) | `P0` → `C2.1`, `P1` → `C2.2` | All messages distributed across two active consumers |

### **Key Observations**
1. **Each consumer group receives all messages independently.**
2. **Within `group2`, only two consumers (`C2.1`, `C2.2`) get messages, while `C2.3` remains idle.**
3. **Consumers in `group2` get messages from only one partition each, enabling parallel processing.**
4. **Consumer in `group1` gets messages from both partitions, leading to sequential processing.**

---

## **Best Practices for Optimizing This Setup**
- **Keep the number of partitions at least equal to the number of consumers within a group** to ensure all consumers are utilized.
- **If more consumers than partitions exist, some will stay idle.**
- **Use `--describe` on the consumer group to check partition assignment:**
  ```bash
  kafka-consumer-groups --bootstrap-server localhost:9092 --group group2 --describe
  ```
- **Consider adding more partitions if you want to fully utilize all consumers.**
