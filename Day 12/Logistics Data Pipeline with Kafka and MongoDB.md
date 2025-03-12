# **Logistics Data Pipeline with Kafka and MongoDB**

This project is a comprehensive data pipeline that integrates **Kafka**, **MongoDB**, and **Flask** to handle logistics data. The goal is to ingest, process, store, and query logistics-related data (e.g., delivery trip truck data) efficiently. Below is a detailed explanation of the project:

---

### **1. Tools and Technologies Used**
- **Python3**: The primary programming language used for scripting.
- **Confluent Kafka**: A distributed streaming platform used for data ingestion and processing.
- **Jupyter Notebook**: Used for prototyping and testing scripts.
- **Postman**: Used for testing API endpoints.
- **MongoDB Atlas**: A cloud-based MongoDB service used for storing the logistics data.
- **MongoDB Compass**: A GUI tool for MongoDB to visualize and query data.
- **Flask**: A lightweight web framework used to create RESTful APIs for querying the MongoDB database.
- **Docker**: Used for containerization and scaling the consumer application.

---

### **2. Project Files**
The project consists of several files, each serving a specific purpose:

1. **delivery_trip_truck_data.csv**: The raw CSV file containing logistics data (e.g., vehicle details, GPS provider, trip details, etc.).
2. **logistics_data_producer.py**: A Python script that reads data from the CSV file, serializes it into Avro format, and produces it to a Kafka topic.
3. **logistics_data_consumer.py**: A Python script that consumes data from the Kafka topic, deserializes it, performs data validation, and stores it in MongoDB.
4. **logistics_data_api1.py**: A Flask API script that allows querying MongoDB for documents based on a specific `vehicle_no`.
5. **logistics_data_api2.py**: A Flask API script that aggregates and counts the number of vehicles grouped by `GpsProvider`.
6. **Dockerfile.txt**: A Dockerfile used to build a Docker image for the consumer application.
7. **docker-compose.yml**: A YAML file used to define and run multiple Docker containers for scaling the consumer application.
8. **Solution_Explanation.docx**: A document explaining the project, tools, and steps.

---

### **3. Project Workflow**

#### **Step 1: Kafka Topic Creation**
- A Kafka topic named `logistics_data` was created with 6 partitions.
- The schema for the data was defined in Confluent Schema Registry to ensure proper serialization and deserialization of data.
- The CSV data (`delivery_trip_truck_data.csv`) was prepared by handling `NaN` values, replacing them with `'unknown value'` for string fields.

#### **Step 2: Data Ingestion with Kafka Producer**
- The **logistics_data_producer.py** script reads data from the CSV file and serializes it into **Avro format**.
- The producer sends the data to the Kafka topic (`logistics_data`) using the `GpsProvider` as the key.
- The script ensures that each record is successfully produced to the Kafka topic, and a delivery report is printed.

#### **Step 3: MongoDB Database Setup**
- A MongoDB database named `gds_db` was created in MongoDB Atlas.
- An empty collection named `logistics_data` was created to store the ingested data.

#### **Step 4: Data Consumption and Storage with Kafka Consumer**
- The **logistics_data_consumer.py** script consumes data from the Kafka topic.
- The data is deserialized from Avro format back into Python objects.
- Data validation checks are performed to ensure:
  - The `bookingID` field is present and not null.
  - The `bookingID` is of the correct data type (string).
- The script checks for duplicate records in MongoDB before inserting new data.
- Validated data is stored in the `logistics_data` collection in MongoDB.

#### **Step 5: Scaling with Docker**
- A **Dockerfile** was created to define the dependencies and environment for the consumer application.
- The **docker-compose.yml** file was used to scale the consumer application by running multiple consumer instances in parallel (3 consumers in this case).
- This ensures that the data processing and storage can handle large volumes of data efficiently.

#### **Step 6: Querying Data with Flask APIs**
- Two Flask APIs were created to query the MongoDB database:
  1. **logistics_data_api1.py**:
     - Provides an endpoint (`/api/filter`) to filter documents based on a specific `vehicle_no`.
     - The API returns the matching documents in JSON format.
  2. **logistics_data_api2.py**:
     - Provides an endpoint (`/api/count`) to aggregate and count the number of vehicles grouped by `GpsProvider`.
     - The API returns the aggregated results in JSON format.
- The APIs were tested using **Postman** and directly via browser URLs.

---

### **4. Key Features**
- **Data Validation**: The consumer script ensures that only valid and non-duplicate data is stored in MongoDB.
- **Scalability**: Docker and Docker Compose are used to scale the consumer application, allowing multiple consumers to process data in parallel.
- **RESTful APIs**: Flask APIs provide easy access to query and analyze the stored data.
- **Schema Management**: Avro schemas are used to ensure data consistency and compatibility between producers and consumers.
- **Cloud Integration**: MongoDB Atlas and Confluent Cloud (Kafka) are used for cloud-based data storage and streaming.

---

### **5. Screenshots and Visualizations**
- The **Solution_Explanation.docx** file includes screenshots of:
  - Kafka producer successfully producing messages.
  - MongoDB Compass showing the stored data.
  - Docker Compose running multiple consumers.
  - Flask API outputs in Postman and browser.

---

### **6. Benefits of the Project**
- **Efficient Data Pipeline**: The project demonstrates a robust pipeline for ingesting, processing, and storing logistics data.
- **Scalability**: The use of Docker and Kafka allows the system to handle large volumes of data.
- **Flexibility**: The Flask APIs provide flexible querying options for end-users.
- **Data Integrity**: Avro schemas and data validation ensure data consistency and accuracy.

---

### **7. Future Enhancements**
- **Real-time Analytics**: Integrate real-time analytics tools (e.g., Apache Spark) to analyze the data as it is ingested.
- **Error Handling**: Enhance error handling and retry mechanisms in the consumer script.
- **Authentication**: Add authentication and authorization to the Flask APIs for secure access.
- **Dashboard**: Create a dashboard (e.g., using React or Angular) to visualize the logistics data.

---

This project is a complete end-to-end solution for handling logistics data, showcasing the integration of modern data engineering tools and techniques.

<br/>
<br/>

# Producer Code 

The **Producer Code** (`logistics_data_producer.py`) is responsible for reading logistics data from a CSV file (`delivery_trip_truck_data.csv`), serializing it into **Avro format**, and producing it to a Kafka topic (`logistics_data`). Below is a detailed explanation of the code, along with sample outputs.

---

### **1. Key Components of the Producer Code**

#### **1.1. Libraries Used**
- **confluent_kafka**: For Kafka producer functionality.
- **pandas**: For reading and processing the CSV file.
- **confluent_kafka.schema_registry**: For Avro serialization using Confluent Schema Registry.
- **uuid, datetime, time**: For generating unique IDs and handling timestamps.

#### **1.2. Kafka Configuration**
The Kafka configuration is defined in the `kafka_config` dictionary:
```python
kafka_config = {
    'bootstrap.servers': 'pkc-921jm.us-east-2.aws.confluent.cloud:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': '6XKBXWERKDEGFDUB',
    'sasl.password': 'Px0Bvj8IhlYWQNSChmL7e6o8BrG5IQrZvEQ0HWx9R0FSJDZi4wYotXoa0q6Na+aj'
}
```
- **bootstrap.servers**: The Kafka broker address.
- **sasl.mechanisms**: The authentication mechanism (PLAIN).
- **security.protocol**: The security protocol (SASL_SSL).
- **sasl.username** and **sasl.password**: Credentials for connecting to the Kafka broker.

#### **1.3. Schema Registry Configuration**
The Schema Registry is used to serialize data into Avro format:
```python
schema_registry_client = SchemaRegistryClient({
    'url': 'https://URL',
    'basic.auth.user.info': '{}:{}'.format('JSFZ3A3FPdsfHTHTJUO', 'NmDeqPb+5DkKIc+GCiuaazHtDHwBOHZrf2wYuE12VlF58HfodcJbkbiDc8GDXV')
})
```
- **url**: The Schema Registry URL.
- **basic.auth.user.info**: Credentials for accessing the Schema Registry.

#### **1.4. Avro Serializer**
The Avro serializer is created using the schema fetched from the Schema Registry:
```python
subject_name = 'logistics_data-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str
avro_serializer = AvroSerializer(schema_registry_client, schema_str)
```
- **subject_name**: The name of the schema subject (e.g., `logistics_data-value`).
- **schema_str**: The schema definition in string format.

#### **1.5. Kafka Producer**
The Kafka producer is configured with the Avro serializer:
```python
producer = SerializingProducer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.serializer': StringSerializer('utf_8'),
    'value.serializer': avro_serializer
})
```
- **key.serializer**: Serializes the key (e.g., `GpsProvider`) as a string.
- **value.serializer**: Serializes the value (logistics data) using Avro.

#### **1.6. Data Preparation**
The CSV file is read into a Pandas DataFrame:
```python
data = pd.read_csv('delivery_trip_truck_data.csv')
```
- Missing values in string columns are replaced with `'unknown value'`:
```python
object_columns = data.select_dtypes(include=['object']).columns
data[object_columns] = data[object_columns].fillna('unknown value')
```

#### **1.7. Data Production**
The `fetch_and_produce_data` function iterates through the DataFrame and produces each row to the Kafka topic:
```python
def fetch_and_produce_data(producer, data):
    for index, row in data.iterrows():
        logistics_data = {
            "GpsProvider": row["GpsProvider"],
            "BookingID": row["BookingID"],
            "Market/Regular ": row["Market/Regular "],
            "BookingID_Date": row["BookingID_Date"],
            "vehicle_no": row["vehicle_no"],
            "Origin_Location": row["Origin_Location"],
            "Destination_Location": row["Destination_Location"],
            "Org_lat_lon": row["Org_lat_lon"],
            "Des_lat_lon": row["Des_lat_lon"],
            "Data_Ping_time": row["Data_Ping_time"],
            "Planned_ETA": row["Planned_ETA"],
            "Current_Location": row["Current_Location"],
            "DestinationLocation": row["DestinationLocation"],
            "actual_eta": row["actual_eta"],
            "Curr_lat": row["Curr_lat"],
            "Curr_lon": row["Curr_lon"],
            "ontime": row["ontime"],
            "delay": row["delay"],
            "OriginLocation_Code": row["OriginLocation_Code"],
            "DestinationLocation_Code": row["DestinationLocation_Code"],
            "trip_start_date": row["trip_start_date"],
            "trip_end_date": row["trip_end_date"],
            "TRANSPORTATION_DISTANCE_IN_KM": row["TRANSPORTATION_DISTANCE_IN_KM"],
            "vehicleType": row["vehicleType"],
            "Minimum_kms_to_be_covered_in_a_day": row["Minimum_kms_to_be_covered_in_a_day"],
            "Driver_Name": row["Driver_Name"],
            "Driver_MobileNo": row["Driver_MobileNo"],
            "customerID": row["customerID"],
            "customerNameCode": row["customerNameCode"],
            "supplierID": row["supplierID"],
            "supplierNameCode": row["supplierNameCode"],
            "Material Shipped": row["Material Shipped"]
        }

        producer.produce(
            topic='logistics_data',
            key=str(row["GpsProvider"]),
            value=logistics_data,
            on_delivery=delivery_report
        )
        print("Produced message:", logistics_data)
```
- **key**: The `GpsProvider` field is used as the key.
- **value**: The entire row is serialized as the value.
- **on_delivery**: A callback function (`delivery_report`) is used to log the delivery status.

#### **1.8. Delivery Report**
The `delivery_report` function logs the success or failure of message production:
```python
def delivery_report(err, msg):
    if err is not None:
        print("Delivery failed for User record {}: {}".format(msg.key(), err))
        return
    print('User record {} successfully produced to {} [{}] at offset {}'.format(
        msg.key(), msg.topic(), msg.partition(), msg.offset()))
```

---

### **2. Sample Outputs**

#### **2.1. Successful Message Production**
When a message is successfully produced, the output looks like this:
```
User record GPSProvider1 successfully produced to logistics_data [0] at offset 123
Produced message: {
    "GpsProvider": "GPSProvider1",
    "BookingID": "12345",
    "Market/Regular ": "Market",
    "vehicle_no": "HR68B5696",
    ...
}
```

#### **2.2. Delivery Failure**
If a message fails to be produced, the output looks like this:
```
Delivery failed for User record GPSProvider2: Broker not available
```

#### **2.3. Produced Message**
The produced message (logistics data) is printed in JSON format:
```json
{
    "GpsProvider": "GPSProvider1",
    "BookingID": "12345",
    "Market/Regular ": "Market",
    "vehicle_no": "HR68B5696",
    "Origin_Location": "Delhi",
    "Destination_Location": "Mumbai",
    "Org_lat_lon": "28.7041,77.1025",
    "Des_lat_lon": "19.0760,72.8777",
    "Data_Ping_time": "2023-10-01 12:00:00",
    "Planned_ETA": "2023-10-02 12:00:00",
    "Current_Location": "Jaipur",
    "DestinationLocation": "Mumbai",
    "actual_eta": "2023-10-02 14:00:00",
    "Curr_lat": "26.9124",
    "Curr_lon": "75.7873",
    "ontime": "No",
    "delay": "2 hours",
    ...
}
```

---

### **3. Summary**
The **Producer Code**:
- Reads logistics data from a CSV file.
- Serializes the data into Avro format using Confluent Schema Registry.
- Produces the data to a Kafka topic (`logistics_data`) with `GpsProvider` as the key.
- Logs the delivery status of each message.

This code is a critical part of the data pipeline, ensuring that logistics data is ingested into Kafka for further processing by consumers.


<br/>
<br/>

# **Project Documentation: Logistics Data Pipeline with Kafka, MongoDB, and Flask APIs**


## **1. Project Overview**

This project is a **data pipeline** that integrates **Kafka**, **MongoDB**, and **Flask** to handle logistics data. The pipeline ingests logistics data from a CSV file, processes it using Kafka, stores it in MongoDB, and provides RESTful APIs for querying and analyzing the data. The project is designed to be scalable, efficient, and easy to use.

---

## **2. Tools and Technologies Used**

- **Python3**: Primary programming language for scripting.
- **Confluent Kafka**: Distributed streaming platform for data ingestion and processing.
- **MongoDB Atlas**: Cloud-based MongoDB service for data storage.
- **Flask**: Lightweight web framework for creating RESTful APIs.
- **Docker**: Containerization tool for scaling the consumer application.
- **Postman**: Tool for testing API endpoints.
- **MongoDB Compass**: GUI tool for MongoDB to visualize and query data.
- **Jupyter Notebook**: Used for prototyping and testing scripts.

---

## **3. Project Files**

1. **delivery_trip_truck_data.csv**: Raw CSV file containing logistics data.
2. **logistics_data_producer.py**: Python script to produce data to Kafka.
3. **logistics_data_consumer.py**: Python script to consume data from Kafka and store it in MongoDB.
4. **logistics_data_api1.py**: Flask API to filter documents by `vehicle_no`.
5. **logistics_data_api2.py**: Flask API to count vehicles by `GpsProvider`.
6. **Dockerfile.txt**: Dockerfile to build a Docker image for the consumer.
7. **docker-compose.yml**: Docker Compose file to scale the consumer application.
8. **Solution_Explanation.docx**: Detailed explanation of the project.

---

## **4. Workflow**

### **4.1. Data Ingestion with Kafka Producer**
- The **logistics_data_producer.py** script reads data from the CSV file (`delivery_trip_truck_data.csv`).
- It serializes the data into **Avro format** using Confluent Schema Registry.
- The producer sends the data to the Kafka topic (`logistics_data`) with `GpsProvider` as the key.

### **4.2. Data Processing with Kafka Consumer**
- The **logistics_data_consumer.py** script consumes data from the Kafka topic.
- It deserializes the data from Avro format and performs **data validation**:
  - Checks for missing or null `bookingID`.
  - Ensures `bookingID` is of type `string`.
- The consumer checks for duplicate records in MongoDB before inserting new data.
- Validated data is stored in the `logistics_data` collection in MongoDB.

### **4.3. Scaling with Docker**
- A **Dockerfile** is used to define the environment and dependencies for the consumer application.
- The **docker-compose.yml** file is used to scale the consumer application by running multiple consumer instances in parallel (e.g., 3 consumers).

### **4.4. Querying Data with Flask APIs**
- **logistics_data_api1.py**:
  - Provides an endpoint (`/api/filter`) to filter documents by `vehicle_no`.
  - Returns matching documents in JSON format.
- **logistics_data_api2.py**:
  - Provides an endpoint (`/api/count`) to count vehicles grouped by `GpsProvider`.
  - Returns aggregated results in JSON format.

---

## **5. Detailed Explanation of Scripts**

### **5.1. logistics_data_producer.py**
- **Purpose**: Produces logistics data to Kafka.
- **Key Features**:
  - Reads data from a CSV file.
  - Serializes data into Avro format.
  - Produces data to the `logistics_data` Kafka topic.
- **Sample Output**:
  ```
  Produced message: {"GpsProvider": "GPSProvider1", "BookingID": "12345", ...}
  ```

### **5.2. logistics_data_consumer.py**
- **Purpose**: Consumes logistics data from Kafka and stores it in MongoDB.
- **Key Features**:
  - Deserializes data from Avro format.
  - Performs data validation.
  - Checks for duplicate records.
  - Inserts valid data into MongoDB.
- **Sample Output**:
  ```
  Inserted message into MongoDB: {"GpsProvider": "GPSProvider1", "BookingID": "12345", ...}
  ```

### **5.3. logistics_data_api1.py**
- **Purpose**: Filters documents by `vehicle_no`.
- **Endpoint**: `/api/filter`
- **Sample Output**:
  ```json
  {
      "result": [
          {"_id": "64f8a1b2e4b0a1a2b3c4d5e6", "vehicle_no": "HR68B5696"}
      ]
  }
  ```

### **5.4. logistics_data_api2.py**
- **Purpose**: Counts vehicles by `GpsProvider`.
- **Endpoint**: `/api/count`
- **Sample Output**:
  ```json
  {
      "result": [
          {"_id": "GPSProvider1", "count": 10}
      ]
  }
  ```

---

## **6. Setup and Execution**

### **6.1. Prerequisites**
- Python 3.x
- Kafka and Schema Registry (Confluent Cloud or local setup)
- MongoDB Atlas or local MongoDB instance
- Docker (optional, for scaling consumers)

### **6.2. Steps to Run the Project**

1. **Set Up Kafka and Schema Registry**:
   - Create a Kafka topic (`logistics_data`).
   - Set up Confluent Schema Registry and define the Avro schema.

2. **Run the Producer**:
   - Execute the **logistics_data_producer.py** script to produce data to Kafka.

3. **Run the Consumer**:
   - Execute the **logistics_data_consumer.py** script to consume data from Kafka and store it in MongoDB.
   - Alternatively, use Docker Compose to scale the consumer:
     ```bash
     docker-compose up --scale consumer=3
     ```

4. **Run the Flask APIs**:
   - Start **logistics_data_api1.py**:
     ```bash
     python logistics_data_api1.py
     ```
   - Start **logistics_data_api2.py**:
     ```bash
     python logistics_data_api2.py
     ```

5. **Test the APIs**:
   - Use Postman or a browser to test the API endpoints:
     - Filter documents by `vehicle_no`: `http://localhost:5000/api/filter?vehicle_no=HR68B5696`
     - Count vehicles by `GpsProvider`: `http://localhost:5001/api/count`

---

## **7. Error Handling**

- **Producer**: Logs delivery failures.
- **Consumer**: Skips invalid or duplicate records.
- **APIs**: Return meaningful error messages for missing parameters, no data, or unexpected errors.

---

## **8. Future Enhancements**

- **Real-time Analytics**: Integrate Apache Spark for real-time data analysis.
- **Authentication**: Add authentication and authorization to the Flask APIs.
- **Dashboard**: Create a dashboard (e.g., using React or Angular) to visualize logistics data.
- **Error Recovery**: Implement retry mechanisms for Kafka consumers.

---

## **9. Conclusion**

This project demonstrates a robust **end-to-end data pipeline** for handling logistics data. It integrates Kafka for data ingestion, MongoDB for storage, and Flask for querying and analysis. The use of Docker ensures scalability, while the APIs provide flexible access to the data. This project can be extended and customized for various logistics and supply chain use cases.

--- 

**End of Documentation**

<br/>
<br/>

# **Consumer Code**

The **Consumer Code** (`logistics_data_consumer.py`) is responsible for consuming logistics data from a Kafka topic (`logistics_data`), deserializing it from Avro format, performing data validation, and storing it in a MongoDB collection (`logistics_data`). Below is a detailed explanation of the code, along with sample outputs.

---

### **1. Key Components of the Consumer Code**

#### **1.1. Libraries Used**
- **confluent_kafka**: For Kafka consumer functionality.
- **pymongo**: For interacting with MongoDB.
- **confluent_kafka.schema_registry**: For Avro deserialization using Confluent Schema Registry.
- **threading**: For handling multiple consumers (if needed).

#### **1.2. Kafka Configuration**
The Kafka configuration is defined in the `kafka_config` dictionary:
```python
kafka_config = {
    'bootstrap.servers': 'pkc-921jm.us-east-2.aws.confluent.cloud:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': '6XKBXWERKDEGFDUB',
    'sasl.password': 'Px0Bvj8IhlYWQNSChmL7e6o8BrG5IQrZvEQ0HWx9R0FSJDZi4wYotXoa0q6Na+aj',
    'group.id': 'group11',
    'auto.offset.reset': 'earliest'
}
```
- **bootstrap.servers**: The Kafka broker address.
- **sasl.mechanisms**: The authentication mechanism (PLAIN).
- **security.protocol**: The security protocol (SASL_SSL).
- **sasl.username** and **sasl.password**: Credentials for connecting to the Kafka broker.
- **group.id**: The consumer group ID.
- **auto.offset.reset**: Specifies where to start consuming messages (`earliest` means from the beginning of the topic).

#### **1.3. Schema Registry Configuration**
The Schema Registry is used to deserialize data from Avro format:
```python
schema_registry_client = SchemaRegistryClient({
    'url': 'https://psrc-yorrp.us-east-2.aws.confluent.cloud',
    'basic.auth.user.info': '{}:{}'.format('JSFZ3A3FPHTHTJUO', 'NmDqPb+5CkKIc+KCiuaazHtDHwBOHZXENUH3V2wYuE12VlF58HfoJbkbiDc8GDXV')
})
```
- **url**: The Schema Registry URL.
- **basic.auth.user.info**: Credentials for accessing the Schema Registry.

#### **1.4. Avro Deserializer**
The Avro deserializer is created using the schema fetched from the Schema Registry:
```python
subject_name = 'logistics_data-value'
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str
avro_deserializer = AvroDeserializer(schema_registry_client, schema_str)
```
- **subject_name**: The name of the schema subject (e.g., `logistics_data-value`).
- **schema_str**: The schema definition in string format.

#### **1.5. MongoDB Configuration**
The MongoDB connection is established using `pymongo`:
```python
mongo_client = MongoClient('mongodb+srv://absf1r3:Potato123@mongodbtest.qa6icb1.mongodb.net/?retryWrites=true&w=majority')
db = mongo_client['gds_db']
collection = db['logistics_data']
```
- **MongoClient**: Connects to the MongoDB Atlas cluster.
- **db**: The database (`gds_db`).
- **collection**: The collection (`logistics_data`) where data will be stored.

#### **1.6. Kafka Consumer**
The Kafka consumer is configured with the Avro deserializer:
```python
consumer = DeserializingConsumer({
    'bootstrap.servers': kafka_config['bootstrap.servers'],
    'security.protocol': kafka_config['security.protocol'],
    'sasl.mechanisms': kafka_config['sasl.mechanisms'],
    'sasl.username': kafka_config['sasl.username'],
    'sasl.password': kafka_config['sasl.password'],
    'key.deserializer': StringDeserializer('utf_8'),
    'value.deserializer': avro_deserializer,
    'group.id': kafka_config['group.id'],
    'auto.offset.reset': kafka_config['auto.offset.reset']
})
```
- **key.deserializer**: Deserializes the key (e.g., `GpsProvider`) as a string.
- **value.deserializer**: Deserializes the value (logistics data) using Avro.

#### **1.7. Data Consumption and Validation**
The consumer polls for messages from the Kafka topic and processes them:
```python
try:
    while True:
        msg = consumer.poll(1.0)  # Poll for messages with a timeout of 1 second

        if msg is None:
            continue
        if msg.error():
            print('Consumer error: {}'.format(msg.error()))
            continue

        # Deserialize Avro data
        value = msg.value()
        print("Received message:", value)
        
        # Data validation checks
        if 'bookingID' not in value or value['bookingID'] is None:
            print("Skipping message due to missing or null 'bookingID'.")
            continue

        if not isinstance(value['bookingID'], str):
            print("Skipping message due to 'bookingID' not being a string.")
            continue
        
        # Check for duplicate records
        existing_document = collection.find_one({'bookingID': value['bookingID']})

        if existing_document:
            print(f"Document with bookingID '{value['bookingID']}' already exists. Skipping insertion.")
        else:
            # Insert data into MongoDB
            collection.insert_one(value)
            print("Inserted message into MongoDB:", value)

except KeyboardInterrupt:
    pass
finally:
    # Commit the offset to mark the message as processed
    consumer.commit()
    consumer.close()
    mongo_client.close()
```
- **Polling**: The consumer polls for messages every 1 second.
- **Error Handling**: If there is an error, it is logged, and the consumer continues.
- **Data Validation**:
  - Checks if the `bookingID` field is present and not null.
  - Ensures the `bookingID` is of type `string`.
- **Duplicate Check**: The consumer checks if a document with the same `bookingID` already exists in MongoDB.
- **Data Insertion**: If the document is valid and not a duplicate, it is inserted into MongoDB.

---

### **2. Sample Outputs**

#### **2.1. Received Message**
When a message is received, it is printed in JSON format:
```
Received message: {
    "GpsProvider": "GPSProvider1",
    "BookingID": "12345",
    "Market/Regular ": "Market",
    "vehicle_no": "HR68B5696",
    "Origin_Location": "Delhi",
    "Destination_Location": "Mumbai",
    "Org_lat_lon": "28.7041,77.1025",
    "Des_lat_lon": "19.0760,72.8777",
    "Data_Ping_time": "2023-10-01 12:00:00",
    "Planned_ETA": "2023-10-02 12:00:00",
    "Current_Location": "Jaipur",
    "DestinationLocation": "Mumbai",
    "actual_eta": "2023-10-02 14:00:00",
    "Curr_lat": "26.9124",
    "Curr_lon": "75.7873",
    "ontime": "No",
    "delay": "2 hours",
    ...
}
```

#### **2.2. Skipping Invalid Message**
If a message fails validation, it is skipped:
```
Skipping message due to missing or null 'bookingID'.
```

#### **2.3. Skipping Duplicate Message**
If a document with the same `bookingID` already exists, it is skipped:
```
Document with bookingID '12345' already exists. Skipping insertion.
```

#### **2.4. Inserting Message into MongoDB**
If the message is valid and not a duplicate, it is inserted into MongoDB:
```
Inserted message into MongoDB: {
    "GpsProvider": "GPSProvider1",
    "BookingID": "12345",
    "Market/Regular ": "Market",
    "vehicle_no": "HR68B5696",
    "Origin_Location": "Delhi",
    "Destination_Location": "Mumbai",
    "Org_lat_lon": "28.7041,77.1025",
    "Des_lat_lon": "19.0760,72.8777",
    "Data_Ping_time": "2023-10-01 12:00:00",
    "Planned_ETA": "2023-10-02 12:00:00",
    "Current_Location": "Jaipur",
    "DestinationLocation": "Mumbai",
    "actual_eta": "2023-10-02 14:00:00",
    "Curr_lat": "26.9124",
    "Curr_lon": "75.7873",
    "ontime": "No",
    "delay": "2 hours",
    ...
}
```

---

### **3. Summary**
The **Consumer Code**:
- Consumes logistics data from a Kafka topic.
- Deserializes the data from Avro format using Confluent Schema Registry.
- Performs data validation to ensure data integrity.
- Checks for duplicate records in MongoDB.
- Inserts valid and non-duplicate data into MongoDB.

This code is a critical part of the data pipeline, ensuring that logistics data is processed, validated, and stored in MongoDB for further querying and analysis.

# Other Required Codes

The **logistics_data_api1.py** and **logistics_data_api2.py** scripts are Flask-based RESTful APIs that allow users to query the MongoDB database containing logistics data. Below is a detailed explanation of each API, including their functionality, endpoints, and sample outputs.

---

### **1. logistics_data_api1.py**

#### **1.1. Purpose**
This API provides an endpoint to filter documents in the MongoDB collection (`logistics_data`) based on a specific `vehicle_no`. It returns the matching documents in JSON format.

#### **1.2. Key Components**

##### **1.2.1. Libraries Used**
- **Flask**: For creating the RESTful API.
- **pymongo**: For interacting with MongoDB.
- **bson.ObjectId**: For handling MongoDB ObjectId.

##### **1.2.2. MongoDB Configuration**
The MongoDB connection is established using `pymongo`:
```python
mongo_client = MongoClient('mongodb+srv://absf1r3:Potato123@mongodbtest.qa6icb1.mongodb.net/?retryWrites=true&w=majority')
db = mongo_client['gds_db']
collection = db['logistics_data']
```
- **MongoClient**: Connects to the MongoDB Atlas cluster.
- **db**: The database (`gds_db`).
- **collection**: The collection (`logistics_data`) where the logistics data is stored.

##### **1.2.3. Flask API Endpoint**
The API has a single endpoint (`/api/filter`) that accepts a `vehicle_no` as a query parameter:
```python
@app.route('/api/filter', methods=['GET'])
def filter_documents():
    try:
        # Get the value of the 'vehicle_no' query parameter
        vehicle_no = request.args.get('vehicle_no')

        if not vehicle_no:
            return jsonify({"error": "Missing 'vehicle_no' parameter"}), 400

        # Query MongoDB for documents with the specified 'vehicle_no'
        result = collection.find({'vehicle_no': vehicle_no})

        # Convert the result to a list and handle ObjectId serialization
        result_list = [{'_id': str(doc['_id']), 'vehicle_no': doc['vehicle_no']} for doc in result]

        if result_list:
            return jsonify({"result": result_list}), 200
        else:
            return jsonify({"result": "No matching documents found"}), 404

    except Exception as e:
        error_message = str(e) if e and str(e) else "An unexpected error occurred."
        return jsonify({"error": error_message}), 500
```
- **Endpoint**: `/api/filter`
- **Method**: `GET`
- **Query Parameter**: `vehicle_no` (required)
- **Functionality**:
  - Fetches documents from MongoDB where the `vehicle_no` matches the provided value.
  - Converts the MongoDB `ObjectId` to a string for JSON serialization.
  - Returns the matching documents in JSON format.

#### **1.3. Sample Outputs**

##### **1.3.1. Successful Query**
If documents with the specified `vehicle_no` are found, the API returns:
```json
{
    "result": [
        {
            "_id": "64f8a1b2e4b0a1a2b3c4d5e6",
            "vehicle_no": "HR68B5696"
        },
        {
            "_id": "64f8a1b2e4b0a1a2b3c4d5e7",
            "vehicle_no": "HR68B5696"
        }
    ]
}
```

##### **1.3.2. No Matching Documents**
If no documents match the `vehicle_no`, the API returns:
```json
{
    "result": "No matching documents found"
}
```

##### **1.3.3. Missing Parameter**
If the `vehicle_no` parameter is missing, the API returns:
```json
{
    "error": "Missing 'vehicle_no' parameter"
}
```

##### **1.3.4. Error Handling**
If an unexpected error occurs, the API returns:
```json
{
    "error": "An unexpected error occurred."
}
```

---

### **2. logistics_data_api2.py**

#### **2.1. Purpose**
This API provides an endpoint to aggregate and count the number of vehicles grouped by `GpsProvider`. It returns the aggregated results in JSON format.

#### **2.2. Key Components**

##### **2.2.1. Libraries Used**
- **Flask**: For creating the RESTful API.
- **pymongo**: For interacting with MongoDB.

##### **2.2.2. MongoDB Configuration**
The MongoDB connection is established using `pymongo`:
```python
mongo_client = MongoClient('mongodb+srv://abcd1r3:Shivam123@mongodbtest.qa6icb1.mongodb.net/?retryWrites=true&w=majority')
db = mongo_client['gds_db']
collection = db['logistics_data']
```
- **MongoClient**: Connects to the MongoDB Atlas cluster.
- **db**: The database (`gds_db`).
- **collection**: The collection (`logistics_data`) where the logistics data is stored.

##### **2.2.3. Flask API Endpoint**
The API has a single endpoint (`/api/count`) that aggregates and counts vehicles by `GpsProvider`:
```python
@app.route('/api/count', methods=['GET'])
def count_vehicles_by_gps_provider():
    try:
        # Aggregate the count of vehicles by GpsProvider
        pipeline = [
            {"$group": {"_id": "$GpsProvider", "count": {"$sum": 1}}}
        ]

        result = list(collection.aggregate(pipeline))

        if result:
            return jsonify({"result": result}), 200
        else:
            return jsonify({"result": "No data available"}), 404

    except Exception as e:
        error_message = str(e) if e and str(e) else "An unexpected error occurred."
        return jsonify({"error": error_message}), 500
```
- **Endpoint**: `/api/count`
- **Method**: `GET`
- **Functionality**:
  - Uses MongoDB's aggregation framework to group documents by `GpsProvider` and count the number of vehicles for each provider.
  - Returns the aggregated results in JSON format.

#### **2.3. Sample Outputs**

##### **2.3.1. Successful Aggregation**
If data is available, the API returns:
```json
{
    "result": [
        {
            "_id": "GPSProvider1",
            "count": 10
        },
        {
            "_id": "GPSProvider2",
            "count": 5
        }
    ]
}
```

##### **2.3.2. No Data Available**
If no data is available, the API returns:
```json
{
    "result": "No data available"
}
```

##### **2.3.3. Error Handling**
If an unexpected error occurs, the API returns:
```json
{
    "error": "An unexpected error occurred."
}
```

---

### **3. Summary**

#### **logistics_data_api1.py**
- **Purpose**: Filters documents in MongoDB based on `vehicle_no`.
- **Endpoint**: `/api/filter`
- **Input**: `vehicle_no` (query parameter).
- **Output**: Matching documents or an error message.

#### **logistics_data_api2.py**
- **Purpose**: Aggregates and counts vehicles by `GpsProvider`.
- **Endpoint**: `/api/count`
- **Output**: Aggregated counts or an error message.

Both APIs are designed to provide easy access to the logistics data stored in MongoDB, enabling users to query and analyze the data efficiently. They include error handling and return meaningful responses for various scenarios.