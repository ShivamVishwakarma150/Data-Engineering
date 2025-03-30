# **When and Why we should use Windowing?**
Windowing is a technique used in stream processing to divide continuous data streams into finite chunks (windows) for easier analysis. Let's dive into when and why we should use windowing! 🚀  

### 1️⃣ **Aggregating Data over Time**  
When dealing with continuous data streams, you often need to perform calculations over a specific time range. Instead of processing an infinite stream, windowing helps break it into manageable time-based segments. ⏳  

✅ **Example:**  
- Calculate the **total number of transactions per hour** 🛒  
- Find the **maximum temperature per day** 🌡️  

Without windowing, you'd have to process an endless stream, making it impossible to compute meaningful statistics.  

---

### 2️⃣ **Detecting Trends**  
If you want to track **patterns** or **changes over time**, windowing is crucial. 📊 It helps in identifying trends by grouping data into meaningful time frames.  

✅ **Example:**  
- Count how many times a **specific error** occurs in logs **every 15 minutes** ⚠️  
- Track **website visits per hour** to see user activity peaks 📈  

By segmenting data into windows, you can spot recurring issues or trends effectively.  

---

### 3️⃣ **Handling Late Data**  
In real-world streaming systems, data might arrive **late** due to network delays, retries, or other factors. 🌍💨 Windowing, when combined with **watermarking**, allows systems to **handle late data gracefully** and maintain accuracy.  

✅ **Example:**  
- If a **sensor reading** from 2 minutes ago arrives **late**, windowing ensures it still gets processed correctly instead of being ignored. 🏭  
- In an **online bidding system**, if a bid arrives slightly **after the deadline**, windowing can be configured to accept it within a grace period. 💰  

Without windowing, delayed data might be **discarded or misprocessed**, leading to **incorrect results**.  

---

### 4️⃣ **Resource Efficiency**  
Processing an **infinite stream** of data **continuously** is resource-intensive. Windowing **reduces** the load by focusing computations on smaller, manageable chunks. ⚡  

✅ **Example:**  
- Instead of analyzing **all chat messages ever sent**, you can analyze messages **per minute** to detect **spam bursts**. 💬🔍  
- Instead of checking **all website traffic logs**, you can process logs **per hour** to generate reports efficiently. 🌐📜  

By limiting the data to **fixed intervals**, windowing helps **optimize memory and CPU usage** while maintaining high performance.  

---

### **Conclusion**  
Windowing is essential when working with streaming data. It helps in:  
✅ Aggregating time-based data efficiently ⏳  
✅ Detecting patterns and trends 📊  
✅ Handling late-arriving data gracefully ⏰  
✅ Reducing computational overhead for better performance ⚡  

It ensures that real-time processing remains **scalable, efficient, and insightful**! 🚀

<br/>
<br/>

# **Understanding Windowed Aggregations in PySpark Streaming** 🚀

#### **📌 What is Windowing in Streaming Data?**
Windowing allows us to **aggregate streaming data over fixed time intervals**, helping us analyze data trends, detect anomalies, and optimize resource usage.

For example, in a **clickstream analysis**, we might want to count how many clicks happen **per user** in **each minute-long window**.

---

## **🅰️ Step 1: Input Data**
We have a stream of JSON-like events representing user interactions (clicks) with timestamps.

### **🔹 Sample Input**
```json
{
  "user_id": "user1",
  "event": "click",
  "eventtimestamp": "2023-07-30T10:00:00Z"
},
{
  "user_id": "user1",
  "event": "click",
  "eventtimestamp": "2023-07-30T10:00:20Z"
},
{
  "user_id": "user2",
  "event": "click",
  "eventtimestamp": "2023-07-30T10:01:00Z"
},
{
  "user_id": "user1",
  "event": "click",
  "eventtimestamp": "2023-07-30T10:01:20Z"
},
{
  "user_id": "user2",
  "event": "click",
  "eventtimestamp": "2023-07-30T10:02:00Z"
}
```

This represents:
- **User1 clicked twice** between **10:00:00 and 10:01:00**.
- **User2 clicked at 10:01:00**.
- **User1 clicked at 10:01:20**.
- **User2 clicked again at 10:02:00**.

---

## **🅱️ Step 2: Implementation in PySpark**

Here’s how we **process this stream** using PySpark’s **windowed aggregation**.

### **🔹 Code Implementation**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window
from pyspark.sql.types import StructType, StringType, TimestampType

# ✅ Define the schema of incoming JSON data
schema = StructType() \
    .add("user_id", StringType()) \
    .add("event", StringType()) \
    .add("eventtimestamp", TimestampType())

# ✅ Initialize Spark Session
spark = SparkSession.builder.appName("WindowedAggregation").getOrCreate()

# ✅ Read the stream data (from a socket)
df = spark.readStream.format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

# ✅ Parse the data from JSON
df = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select(
        col("data.user_id"),
        col("data.event"),
        col("data.eventtimestamp")
    )

# ✅ Perform the windowed aggregation (1-minute windows)
result = df.groupBy(
    window(df.eventtimestamp, "1 minute"),  # 1-minute time window
    df.user_id  # Group by user_id
).count()

# ✅ Output the results to the console
query = result.writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

---

## **🅾️ Step 3: Output of Windowed Aggregation**

### **🔹 Batch 0: Processing First Two Events**
| Window Start-End | User ID | Count |
|------------------|--------|-------|
| 10:00 - 10:01   | user1  | 2     |

**✅ Explanation:**  
- The first two events from **user1** happened between **10:00:00 and 10:00:20**.
- Since they fall in the **same minute window (10:00 - 10:01)**, they are **grouped together**, and Spark counts **2 clicks**.

---

### **🔹 Batch 1: Processing Next Two Events**
| Window Start-End | User ID | Count |
|------------------|--------|-------|
| 10:00 - 10:01   | user1  | 2     |
| 10:01 - 10:02   | user1  | 1     |
| 10:01 - 10:02   | user2  | 1     |

**✅ Explanation:**  
- The event at **10:01:00 (user2)** starts a new **one-minute window (10:01 - 10:02)**.
- The next event at **10:01:20 (user1)** is also within this **new window**.
- **Both users have 1 click each in this window.**

---

### **🔹 Batch 2: Processing the Last Event**
| Window Start-End | User ID | Count |
|------------------|--------|-------|
| 10:00 - 10:01   | user1  | 2     |
| 10:01 - 10:02   | user1  | 1     |
| 10:01 - 10:02   | user2  | 1     |
| 10:02 - 10:03   | user2  | 1     |

**✅ Explanation:**  
- The final event at **10:02:00 (user2)** starts a **new window (10:02 - 10:03)**.
- **User2 has 1 click in this window.**

---

## **🎯 Summary**
1️⃣ **Windowing helps group data over fixed intervals** (e.g., 1-minute windows).  
2️⃣ **Each batch represents a new set of events that fit into the windowed aggregation.**  
3️⃣ **We use PySpark’s `window()` function** to group data by timestamps.  
4️⃣ **Spark updates the aggregation with new data as it arrives.**  

This is useful in **real-time analytics** like:
- **Tracking website user activity** (clicks per minute) 📊
- **Detecting fraud patterns** (suspicious transactions per hour) 🔥
- **Monitoring system logs** (error occurrences per 5 minutes) 🚨

<br/>
<br/>


# **Understanding Stateful Transformations in PySpark Streaming** 🚀

In **PySpark Structured Streaming**, we often need to **maintain state** across multiple batches of streaming data. This is where **stateful transformations** like `mapGroupsWithState` and `flatMapGroupsWithState` come into play.

---

## **🔹 What is Stateful Processing?**
Unlike **stateless** transformations (e.g., `groupBy().count()`), **stateful** transformations maintain information about past events. This is useful when:
- Tracking **session data** (e.g., user sessions on a website).
- Maintaining a **running count** of events over time.
- Detecting **inactive users** based on a timeout.

---

## **🔹 Key Functions for Stateful Processing**
1️⃣ **`mapGroupsWithState(update_func, TimeoutType)`**
   - Maintains a **state per key** (e.g., per user).
   - Updates the state based on new records.
   - Removes old states using **timeout conditions**.

2️⃣ **`flatMapGroupsWithState(update_func, TimeoutType)`**
   - Similar to `mapGroupsWithState`, but can **return multiple rows per key**.
   - Useful for **sessionization** (e.g., returning a session when it ends).

---

## **🅰️ Example Scenario: User Session Tracking**
### **📝 Problem Statement**
We want to track **active sessions** of users based on their click events. If a user does not interact for **30 seconds**, their session ends.

### **📌 Sample Input Stream**
```json
{ "user_id": "user1", "event": "click", "eventtimestamp": "2023-07-30T10:00:00Z" }
{ "user_id": "user1", "event": "click", "eventtimestamp": "2023-07-30T10:00:20Z" }
{ "user_id": "user2", "event": "click", "eventtimestamp": "2023-07-30T10:01:00Z" }
{ "user_id": "user1", "event": "click", "eventtimestamp": "2023-07-30T10:01:20Z" }
{ "user_id": "user2", "event": "click", "eventtimestamp": "2023-07-30T10:02:00Z" }
```
---
## **🅱️ Step 1: Define the Stateful Update Function**
We will define an **update function** that:
1. Keeps track of **session start time**.
2. **Updates** session duration on each event.
3. **Removes old sessions** if the user is inactive for **30 seconds**.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StringType, TimestampType
from pyspark.sql.streaming.state import GroupState, GroupStateTimeout

# ✅ Define schema for incoming JSON data
schema = StructType() \
    .add("user_id", StringType()) \
    .add("event", StringType()) \
    .add("eventtimestamp", TimestampType())

# ✅ Initialize Spark Session
spark = SparkSession.builder.appName("StatefulAggregation").getOrCreate()

# ✅ Read the stream data from a socket (or Kafka, etc.)
df = spark.readStream.format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

# ✅ Parse JSON data
df = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select(
        col("data.user_id"),
        col("data.event"),
        col("data.eventtimestamp")
    )

# ✅ Define the update function for mapGroupsWithState
def update_func(user_id, events, state: GroupState):
    """
    Updates user session state based on new click events.
    If user is inactive for 30 seconds, their session ends.
    """
    # If state does not exist, initialize it
    if not state.exists:
        state.update({"session_start": None, "event_count": 0})

    # Get current state
    session = state.get()
    
    # Process new events
    for event in events:
        if session["session_start"] is None:
            session["session_start"] = event.eventtimestamp
        
        session["event_count"] += 1  # Increment event count
    
    # Update state with new session info
    state.update(session)

    # Set timeout for 30 seconds
    state.setTimeoutDuration("30 seconds")

    # Return session details
    return [(user_id, session["session_start"], session["event_count"])]

# ✅ Apply stateful transformation
session_df = df.groupByKey(lambda row: row.user_id) \
    .mapGroupsWithState(update_func, GroupStateTimeout.ProcessingTimeTimeout)

# ✅ Output session data to the console
query = session_df.writeStream \
    .outputMode("update") \
    .format("console") \
    .start()

query.awaitTermination()
```

---

## **🅾️ Step 2: Understanding the Stateful Logic**
- **Grouping by user_id** → Each user has a separate session.
- **Updating session state** → Each event updates the session's start time and event count.
- **Timeout (30 seconds)** → If no event occurs for **30 seconds**, the session **expires**.

---

## **🅾️ Step 3: Expected Output**
### **🔹 Batch 0**
| User ID | Session Start | Event Count |
|---------|--------------|-------------|
| user1   | 10:00:00     | 1           |

### **🔹 Batch 1**
| User ID | Session Start | Event Count |
|---------|--------------|-------------|
| user1   | 10:00:00     | 2           |
| user2   | 10:01:00     | 1           |

### **🔹 Batch 2 (After 30s inactivity)**
| User ID | Session Start | Event Count |
|---------|--------------|-------------|
| user1   | 10:00:00     | 3           |
| user2   | 10:01:00     | 2           |

---

## **🎯 Key Takeaways**
✅ **Stateful processing maintains memory across streaming batches.**  
✅ **mapGroupsWithState keeps state per key (e.g., user sessions).**  
✅ **State is updated when new events arrive.**  
✅ **Timeouts remove stale state (e.g., inactive users).**  

This approach is useful for:
- **User session tracking** 🎮
- **Real-time fraud detection** 💰
- **IoT event monitoring** 📡

<br/>
<br/>

# **Understanding Stateful Processing with PySpark Streaming** 🚀

This PySpark program **tracks the total transaction amount per user** in real-time using **stateful processing** (`mapGroupsWithState`). It reads transaction data from **Kafka**, processes it, and keeps a running sum of transactions for each user.

---

## **🔹 Key Features of this Code**
1️⃣ **Reads real-time transaction data from Kafka**  
2️⃣ **Parses JSON messages into structured columns**  
3️⃣ **Maintains a stateful aggregation per user (running total of transaction amounts)**  
4️⃣ **Continuously updates the results and prints them to the console**

---

## **🔹 Step-by-Step Explanation**

### **🅰️ Step 1: Defining the Schema**
Since the Kafka messages are in **JSON format**, we need to define a **schema**:
```python
from pyspark.sql.types import StructType, StringType, LongType, DoubleType

schema = StructType() \
    .add("userId", StringType()) \
    .add("transactionId", StringType()) \
    .add("transactionTime", LongType()) \
    .add("amount", DoubleType())
```
- `userId` → Unique identifier for the user (String)
- `transactionId` → Unique identifier for each transaction (String)
- `transactionTime` → Unix timestamp of the transaction (Long)
- `amount` → Transaction amount (Double)

---

### **🅱️ Step 2: Initializing Spark and Reading from Kafka**
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("UserSession").getOrCreate()
```
- This initializes a **Spark session** with the application name `"UserSession"`.

```python
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "transactions") \
    .load()
```
- Reads a **Kafka stream** from topic `"transactions"`.
- Uses **localhost:9092** as the Kafka broker.

---

### **🅾️ Step 3: Parsing and Formatting Data**
```python
from pyspark.sql.functions import col, from_json, from_unixtime

df = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select(
        col("data.userId"),
        col("data.transactionId"),
        from_unixtime(col("data.transactionTime")).alias("transactionTime"),
        col("data.amount")
    )
```
- Extracts the **JSON message** from Kafka’s `value` column.
- Parses JSON into structured **columns**.
- Converts **Unix timestamp** (`transactionTime`) into human-readable format.

---

### **🅲️ Step 4: Defining Stateful Processing**
Now, we define a **state update function** to maintain the **running total of transactions per user**.

```python
def update_func(key, values, state):
    # If the state exists
    if state.exists:
        new_amount = sum([x.amount for x in values])  # Sum new transactions
        state.update(state.get() + new_amount)  # Update state
    else:
        # Initialize the state with the current amount
        state.update(sum([x.amount for x in values]))
    
    return (key, state.get())
```
- **`key`** → The `userId` (grouping key).
- **`values`** → New transactions for the user in this batch.
- **`state`** → Holds the **cumulative transaction amount** for the user.
- If the user **already has a state**, we **add** the new transactions.
- If the user is **new**, we initialize their state.

---

### **🅳️ Step 5: Applying Stateful Processing**
```python
from pyspark.sql.streaming import GroupStateTimeout

result = df.groupBy("userId").mapGroupsWithState(update_func, GroupStateTimeout.NoTimeout)
```
- Groups by `userId` and applies the `update_func`.
- `GroupStateTimeout.NoTimeout` → The state **never expires**.

---

### **🅴️ Step 6: Writing the Results to the Console**
```python
query = result.writeStream \
    .outputMode("update") \
    .format("console") \
    .start()

query.awaitTermination()
```
- Uses **"update" mode** (only changed rows are printed).
- Prints results **continuously** to the console.

---

## **🔹 Sample Input Messages from Kafka**
```json
{ "userId": "user1", "transactionId": "tx1001", "transactionTime": 1700000000, "amount": 50.0 }
{ "userId": "user1", "transactionId": "tx1002", "transactionTime": 1700000020, "amount": 30.0 }
{ "userId": "user2", "transactionId": "tx2001", "transactionTime": 1700000040, "amount": 20.0 }
{ "userId": "user1", "transactionId": "tx1003", "transactionTime": 1700000060, "amount": 40.0 }
{ "userId": "user2", "transactionId": "tx2002", "transactionTime": 1700000080, "amount": 60.0 }
```

---

## **🔹 Expected Output**
| User ID | Total Amount |
|---------|-------------|
| user1   | 50.0        |
| user1   | 80.0        |
| user2   | 20.0        |
| user1   | 120.0       |
| user2   | 80.0        |

### **Batch 0**
| userId | Total Amount |
|--------|-------------|
| user1  | 50.0        |

### **Batch 1**
| userId | Total Amount |
|--------|-------------|
| user1  | 80.0        |
| user2  | 20.0        |

### **Batch 2**
| userId | Total Amount |
|--------|-------------|
| user1  | 120.0       |
| user2  | 80.0        |

---

## **🎯 Key Takeaways**
✅ **Maintains running totals for each user in real-time.**  
✅ **Reads messages from Kafka and processes them using PySpark Structured Streaming.**  
✅ **Uses `mapGroupsWithState` to maintain state across multiple batches.**  
✅ **Outputs the continuously updated totals to the console.**  

This approach is useful for:
- **Banking transactions monitoring** 🏦
- **E-commerce order tracking** 🛒
- **Fraud detection systems** 🔍

---

<br/>
<br/>

# **🔹 Understanding the `state` Variable in `mapGroupsWithState`**
The **`state`** variable in **Spark Structured Streaming** is a **persistent, key-based store** that keeps track of **stateful information** across multiple micro-batches of streaming data.

---

## **🟢 Where Does the `state` Variable Come From?**
The **`state`** variable is **automatically provided** by Spark’s streaming engine when using `mapGroupsWithState`. It is:
- **Maintained per unique key** (e.g., `userId` in our example).
- **Stored in an internal state store** managed by Spark.
- **Updated and retrieved** across micro-batches.
- **Kept alive** as long as data for the key continues to arrive.

---

## **🟢 How Does the `state` Variable Work?**
When `mapGroupsWithState` is used, Spark:
1️⃣ **Reads incoming streaming data.**  
2️⃣ **Groups data by key** (e.g., `userId`).  
3️⃣ **Calls `update_func` for each group** in a micro-batch.  
4️⃣ **Passes the `state` object** to `update_func`:
   - If the key has been seen before, `state.get()` returns the previous value.
   - If it's a new key, `state.exists` is `False`, and `state.get()` returns `None`.
5️⃣ **Updates the `state`** using `state.update(new_value)`.
6️⃣ **Stores the updated state** in Spark’s state store.
7️⃣ **Makes the updated state available** in the next micro-batch.

---

## **🟢 `state` Methods Explained**
| Method | Description | Example Usage |
|--------|-------------|------------------|
| `state.exists` | Checks if the key has an existing state. | `if state.exists:` |
| `state.get()` | Retrieves the stored state for the key. | `current_total = state.get()` |
| `state.update(value)` | Updates the state with a new value. | `state.update(current_total + new_amount)` |
| `state.remove()` | Deletes the state for the key. | `state.remove()` |

---

## **🟢 Example: Stateful Transaction Aggregation**
Here’s an **example** that **maintains a running total of transaction amounts** per `userId`.

### **🔹 Step 1: Define the Update Function**
```python
def update_func(userId, transactions, state):
    if state.exists:
        # Retrieve the previous state (total amount spent)
        current_total = state.get()
    else:
        # If no state exists, start from zero
        current_total = 0.0

    # Calculate the total amount from the new transactions
    new_total = current_total + sum(tx.amount for tx in transactions)

    # Update the state with the new total
    state.update(new_total)

    return (userId, new_total)  # Return updated values
```

---

### **🔹 Step 2: Apply Stateful Processing**
```python
result = df.groupBy("userId").mapGroupsWithState(update_func, GroupStateTimeout.NoTimeout)
```
- `groupBy("userId")` → Groups transactions by `userId`.
- `mapGroupsWithState(update_func, GroupStateTimeout.NoTimeout)` → Calls `update_func` **for each user**.

---

### **🟢 Example Input Stream**
| userId | transactionId | transactionTime | amount |
|--------|--------------|----------------|--------|
| user1  | tx1001      | 1700000000      | 50.0   |
| user1  | tx1002      | 1700000020      | 30.0   |
| user2  | tx2001      | 1700000040      | 20.0   |
| user1  | tx1003      | 1700000060      | 40.0   |
| user2  | tx2002      | 1700000080      | 60.0   |

---

### **🟢 Batch Processing**
#### **Batch 1 (First transactions arrive)**
| userId | Total Amount (state) |
|--------|----------------------|
| user1  | 50.0 |
| user1  | 80.0 |
| user2  | 20.0 |

#### **Batch 2 (More transactions arrive)**
| userId | Total Amount (state) |
|--------|----------------------|
| user1  | 120.0 |
| user2  | 80.0 |

---

## **🟢 How Spark Maintains State**
- **Batch 1:**
  - `user1` → 50.0 → **state updated**
  - `user1` → 80.0 (50+30) → **state updated**
  - `user2` → 20.0 → **state updated**
- **Batch 2:**
  - `user1` → 120.0 (80+40) → **state updated**
  - `user2` → 80.0 (20+60) → **state updated**

Each **new batch** processes the transactions and updates the stored **state**.

---

## **🟢 Key Takeaways**
✅ **Spark automatically maintains state per unique key.**  
✅ **State persists across multiple micro-batches.**  
✅ **State allows for incremental aggregation over infinite streams.**  
✅ **Methods like `state.get()`, `state.update()`, and `state.remove()` help manage state.**  
✅ **Used in real-time applications like session tracking, fraud detection, and running totals.**  

This is how Spark’s **stateful processing** enables powerful real-time computations! 

<br/>
<br/>

# **🔹 Explanation of Streaming with Fixed Timeout State in Spark Structured Streaming**

This **PySpark Structured Streaming** application processes **real-time transaction data** from a **Kafka** topic (`transactions`) and performs **stateful aggregation per user**, keeping track of the **total transaction amount** for each `userId`. It also uses **Event-Time-based Timeout**, which means state will be automatically **cleared after 1 hour from the latest transaction time** for each user.

---

## **🔹 Code Breakdown and Explanation**

### **1️⃣ Define the Schema for Incoming Data**
The schema represents the **JSON structure** of incoming messages from Kafka.

```python
schema = StructType() \
    .add("userId", StringType()) \
    .add("transactionId", StringType()) \
    .add("transactionTime", StringType()) \
    .add("amount", DoubleType())
```

- **`userId`** → User identifier (String)  
- **`transactionId`** → Unique transaction ID (String)  
- **`transactionTime`** → Timestamp of the transaction (String, later converted to Timestamp)  
- **`amount`** → Amount spent in the transaction (Double)  

---

### **2️⃣ Initialize Spark Session**
```python
spark = SparkSession.builder.appName("UserSession").getOrCreate()
```
- Initializes a **Spark Session** named `"UserSession"`.

---

### **3️⃣ Read the Stream from Kafka**
```python
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "transactions") \
    .load()
```
- Reads streaming data from the Kafka **topic** `"transactions"`.
- Kafka **brokers** are running on `"localhost:9092"`.
- The data is received as **raw byte values**.

---

### **4️⃣ Parse the JSON Data**
```python
df = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select(
        col("data.userId"),
        col("data.transactionId"),
        unix_timestamp(col("data.transactionTime"), "yyyy-MM-dd HH:mm:ss").cast("timestamp").alias("transactionTime"),
        col("data.amount")
    )
```
- Converts the Kafka `value` field **from binary to string** (`CAST(value AS STRING)`).
- Parses the **JSON format** into structured columns (`from_json`).
- Extracts relevant fields:
  - `userId`
  - `transactionId`
  - **Converts** `transactionTime` **from string to timestamp** (`unix_timestamp(...).cast("timestamp")`)
  - `amount`

---

### **5️⃣ Define the Stateful Processing Function**
```python
def update_func(key, values, state):
    # If the state exists
    if state.exists:
        new_amount = sum([x.amount for x in values])
        # Update the state with the total amount
        state.update(state.get() + new_amount)
    else:
        # Initialize the state with the current amount
        state.update(sum([x.amount for x in values]))
    
    # Set the event time timeout to be 1 hour from the latest transactionTime
    from pyspark.sql.functions import max
    timeout_timestamp = max([x.transactionTime for x in values]).add(hours=1)
    state.setTimeoutTimestamp(timeout_timestamp.timestamp() * 1000)  # Convert to milliseconds

    return (key, state.get())
```

#### **Function Breakdown**
1. **Maintaining State (User's Total Transaction Amount)**
   - If the `state` **already exists** for a user, it adds the `amount` from the new transactions.
   - If the `state` **does not exist**, it initializes the state with the sum of the received amounts.
   - This ensures **cumulative aggregation per user** across micro-batches.

2. **Setting Timeout (Clearing State after 1 Hour)**
   - It calculates the **maximum** `transactionTime` from the incoming batch.
   - The timeout is set to **1 hour after** the latest `transactionTime`.
   - This means if no transactions arrive for a user within **1 hour**, their state is **automatically cleared**.

**🔴 Note**: There's a small issue in the `timeout_timestamp` calculation:
- Instead of `max([x.transactionTime for x in values]).add(hours=1)`, you should use:
  ```python
  from datetime import timedelta
  timeout_timestamp = max([x.transactionTime for x in values]) + timedelta(hours=1)
  ```

---

### **6️⃣ Apply `mapGroupsWithState` for Stateful Aggregation**
```python
result = df.groupBy("userId").mapGroupsWithState(update_func, GroupStateTimeout.EventTimeTimeout)
```
- Groups data **by `userId`**.
- Applies `mapGroupsWithState` using the `update_func`.
- Uses `GroupStateTimeout.EventTimeTimeout`, which means:
  - The **state expires 1 hour after the latest transaction's event-time**.

---

### **7️⃣ Start Streaming Query**
```python
query = result.writeStream \
    .outputMode("update") \
    .format("console") \
    .start()

query.awaitTermination()
```
- **Writes streaming output to the console**.
- Uses `"update"` mode (only updated records are shown).
- The stream runs **continuously** until manually stopped.

---

## **🔹 Expected Behavior**
1. When **transactions arrive for a `userId`**, the total amount is updated.
2. If **no new transactions arrive for 1 hour**, the user’s state is **automatically cleared**.
3. The system efficiently manages **state size** by removing old data.

---

## **🔹 Example Walkthrough**
### **Input Transactions (Kafka Stream)**
| userId | transactionId | transactionTime       | amount |
|--------|--------------|-----------------------|--------|
| user1  | txn_101      | 2025-03-29 10:00:00   | 100.0  |
| user1  | txn_102      | 2025-03-29 10:15:00   | 50.0   |
| user2  | txn_201      | 2025-03-29 10:30:00   | 200.0  |
| user1  | txn_103      | 2025-03-29 11:30:00   | 30.0   |

### **Streaming Processing**
#### **Batch 1 (Initial transactions)**
```
+-------+--------+
| userId| amount |
+-------+--------+
| user1 | 100.0  |
| user1 | 50.0   |
| user2 | 200.0  |
+-------+--------+
```
#### **Batch 2 (1 hour passes, user1 gets new transaction, user2's state is removed)**
```
+-------+--------+
| userId| amount |
+-------+--------+
| user1 | 30.0   |  # user1 continues, but their previous state is removed due to timeout
+-------+--------+
```

---

## **🔹 Key Takeaways**
1. **`mapGroupsWithState` maintains state per `userId`** across micro-batches.
2. **State is automatically removed after 1 hour of inactivity (Event-Time Timeout).**
3. **Memory-efficient:** Old states are automatically cleaned, avoiding memory leaks.
4. **Improves Accuracy:** Avoids outdated user sessions persisting indefinitely.
5. **Real-world Use Cases:**  
   ✅ Session-based aggregations  
   ✅ Fraud detection (monitoring spending patterns)  
   ✅ User behavior tracking  

---

## **🔹 Fixing Errors**
### **1️⃣ Fix `timeout_timestamp` Calculation**
Replace:
```python
timeout_timestamp = max([x.transactionTime for x in values]).add(hours=1)
```
With:
```python
from datetime import timedelta
timeout_timestamp = max([x.transactionTime for x in values]) + timedelta(hours=1)
```

### **2️⃣ `GroupBy` Issue**
`groupBy("userId")` **does not work with `mapGroupsWithState`**, replace it with:
```python
result = df.groupByKey(lambda x: x.userId).mapGroupsWithState(update_func, GroupStateTimeout.EventTimeTimeout)
```

---

## **🔹 Conclusion**
This Spark Streaming job **efficiently tracks user spending**, manages **stateful aggregation**, and ensures **automatic cleanup of inactive users**. 🚀

<br/>
<br/>

# **Difference Between `groupBy` and `mapGroupsWithState` in Spark Structured Streaming**

In **Spark Structured Streaming**, both `groupBy` and `mapGroupsWithState` are used for **grouping and aggregating streaming data**, but they have fundamental differences in how they manage state across micro-batches.

---

## **🔹 `groupBy` in Spark Structured Streaming**
- `groupBy("column")` is a standard **grouping operation** that can be used with aggregation functions like `count()`, `sum()`, `avg()`, etc.
- The **state is managed internally** by Spark, and the user **cannot define or modify the state** explicitly.
- Used in **windowed aggregations** or simple **cumulative** computations.

### **Example: Using `groupBy` with `sum()`**
```python
df = spark.readStream.format("kafka").option("subscribe", "transactions").load()

result = df.groupBy("userId").agg(sum("amount").alias("total_spent"))

query = result.writeStream.outputMode("update").format("console").start()
query.awaitTermination()
```

### **How It Works**
- Groups incoming streaming data by `userId`.
- Computes the **sum of `amount` spent per user** in each batch.
- **State management is handled internally** by Spark.
- No control over **how state is updated** or **when it expires**.

### **Limitations of `groupBy`**
1. **Limited Control Over State:**  
   - Spark decides how to maintain and store the state.
   - Cannot access or modify intermediate states.
   
2. **No Timeout Handling:**  
   - Aggregations persist **indefinitely**, leading to potential **memory issues**.
   - No way to clear state **after inactivity**.

3. **No Custom State Updates:**  
   - Can only perform predefined **sum(), count(), avg()**, etc.
   - Cannot implement **custom logic** to update or modify state.

---

## **🔹 `mapGroupsWithState` in Spark Structured Streaming**
- `mapGroupsWithState` allows **custom stateful processing** over a **grouped dataset**.
- You can **define**, **update**, and **clear the state** based on **custom logic**.
- **Event-time based timeouts** can be implemented to automatically remove old state.

### **Example: Using `mapGroupsWithState` for Stateful Aggregation**
```python
from pyspark.sql.streaming import GroupState, GroupStateTimeout

# Define the update function
def update_func(key, values, state: GroupState[float]):
    if state.exists:
        state.update(state.get() + sum([x.amount for x in values]))
    else:
        state.update(sum([x.amount for x in values]))

    # Set timeout to 1 hour after the last event
    from datetime import timedelta
    timeout_timestamp = max([x.transactionTime for x in values]) + timedelta(hours=1)
    state.setTimeoutTimestamp(timeout_timestamp.timestamp() * 1000)  

    return (key, state.get())

# Apply stateful aggregation
result = df.groupByKey(lambda x: x.userId).mapGroupsWithState(update_func, GroupStateTimeout.EventTimeTimeout)
```

### **How It Works**
- **Groups data by `userId`** (similar to `groupBy`).
- **Maintains a running sum** of the total `amount` spent per user.
- Uses **`GroupState`** to store and update the sum **across micro-batches**.
- **Clears state automatically** if no transactions occur for 1 hour.

---

## **🔹 Key Differences Between `groupBy` and `mapGroupsWithState`**

| Feature                     | `groupBy` + Aggregation  | `mapGroupsWithState`  |
|-----------------------------|------------------------|------------------------|
| **State Management**         | Implicit (Managed by Spark) | Explicit (Controlled by Developer) |
| **Custom State Updates**     | ❌ Not Possible | ✅ Possible |
| **State Expiry (Timeouts)**  | ❌ No Control | ✅ Custom Timeouts |
| **Memory Efficiency**        | 🚨 Risk of Accumulating State | ✅ Old State Can Be Cleared |
| **Flexibility**              | 🚫 Limited | ✅ High (Arbitrary Logic Allowed) |
| **Use Case**                 | Simple Aggregations (sum, count, avg) | Complex Stateful Processing |

---

## **🔹 When to Use What?**
| Scenario | Use `groupBy` | Use `mapGroupsWithState` |
|----------|--------------|-------------------------|
| **Summing total sales per user** | ✅ | 🚫 |
| **Counting occurrences of an event** | ✅ | 🚫 |
| **User session tracking** | 🚫 | ✅ |
| **Detecting anomalies (fraud detection)** | 🚫 | ✅ |
| **Maintaining state across time-based sessions** | 🚫 | ✅ |
| **Custom business logic (like detecting inactivity)** | 🚫 | ✅ |

---

## **🔹 Summary**
- `groupBy` is **easy to use**, but **lacks control over state**.
- `mapGroupsWithState` allows **custom logic** for state updates **and timeouts**.
- If you need **simple aggregations**, use `groupBy() + sum() or count()`.
- If you need **custom stateful processing** (e.g., user sessions, tracking inactivity), use `mapGroupsWithState`.

🚀 **For real-world streaming applications that require custom logic and memory-efficient state management, `mapGroupsWithState` is the best choice!**

<br/>
<br/>

# Kafka-Spark Streaming Project: Fruit Data Processing

This project demonstrates a real-time data processing pipeline using Kafka for message streaming and Spark for stream processing. The system processes fruit-related data, joining streaming data with static reference data.

## Project Components

### 1. Data Producer (`fruit_producer.py`)

**Purpose**: Publishes fruit data messages to a Kafka topic.

**Key Features**:
- Uses the `confluent_kafka` Python library to create a producer
- Reads JSON data from `fruit_data.json`
- Publishes each record to the "fruit_data" Kafka topic
- Includes a 2-second delay between messages to simulate real-time data

**Data Flow**:
1. Loads JSON data from file
2. Publishes each record to Kafka
3. Prints confirmation of each published message

### 2. Stream Processing (`join_stream_data.py`)

**Purpose**: Consumes the Kafka stream and joins it with static reference data.

**Key Features**:
- Uses PySpark Structured Streaming
- Creates a SparkSession with appropriate configurations
- Reads from the "fruit_data" Kafka topic
- Defines a schema for the JSON data
- Joins streaming data with static CSV data from HDFS
- Writes results to console

**Processing Steps**:
1. Reads streaming data from Kafka
2. Parses JSON values with defined schema
3. Reads static reference data from HDFS (`/tmp/input_data/fruit_dim.csv`)
4. Performs inner join on the "id" field
5. Outputs results to console

### 3. Data Files

#### `fruit_data.json`
- Contains 30 records of fruit data
- Each record has:
  - `id`: Unique identifier
  - `value`: Fruit name
  - `timestamp`: Event time in ISO format

#### `fruit_dim.csv`
- Static reference data with 30 records
- Contains:
  - `id`: Matches IDs from streaming data
  - `category`: Classification (mostly "fruit", with some exceptions like "vegetable" and "spice")

## Data Processing Logic

The core operation is joining streaming data with static reference data:
1. Streaming data provides real-time fruit observations
2. Static data provides category information
3. The join enriches the streaming data with category information

Example transformation:
```
Stream Data: {"id": "1", "value": "apple", "timestamp": "2023-07-30T10:00:00Z"}
Static Data: id=1, category="fruit"

Result: id=1, value="apple", timestamp="2023-07-30T10:00:00Z", category="fruit"
```

## Technical Stack

1. **Apache Kafka**: Message broker for streaming data
   - Producer publishes to "fruit_data" topic
   - Spark consumes from the same topic

2. **Apache Spark**: Stream processing engine
   - Uses Structured Streaming API
   - Performs join between streaming and batch data
   - Local mode with 3 cores for processing

3. **HDFS**: Storage for static reference data
   - CSV file stored at `/tmp/input_data/fruit_dim.csv`

## Execution Flow

1. Start Kafka and Zookeeper services
2. Create "fruit_data" topic in Kafka
3. Run `fruit_producer.py` to publish messages
4. Run `join_stream_data.py` to process the stream
5. Observe joined results in Spark console output

## Potential Use Cases

This pattern is useful for scenarios where:
- Real-time events need enrichment with reference data
- Streaming data requires validation or categorization
- You need to combine high-velocity data with slowly-changing dimensions

The project demonstrates fundamental concepts that can be extended to more complex real-world scenarios like IoT data processing, real-time analytics, or event-driven microservices.

<br/>
<br/>

Here's a detailed explanation of both Python files in the project:

---

## **1. `fruit_producer.py` - Kafka Producer**

### **Purpose**
This script acts as a Kafka producer that reads fruit data from a JSON file and publishes each record to a Kafka topic (`fruit_data`) at a fixed interval.

### **Code Breakdown**

#### **Imports**
```python
import time
import json
from confluent_kafka import Producer
```
- `time`: Used to add delays between messages.
- `json`: Reads the JSON file (`fruit_data.json`).
- `confluent_kafka.Producer`: Kafka Python client for publishing messages.

#### **Kafka Producer Setup**
```python
p = Producer({'bootstrap.servers': 'localhost:9092'})
```
- Creates a Kafka producer connected to a local Kafka broker (`localhost:9092`).

#### **Load JSON Data**
```python
with open('fruit_data.json') as f:
    data = json.load(f)
```
- Opens `fruit_data.json` and loads its contents into a Python list (`data`).

#### **Publish Messages to Kafka**
```python
for row in data:
    p.produce('fruit_data', json.dumps(row))  # Send message to Kafka
    p.flush()  # Ensure message is delivered
    print("Message published -> ", json.dumps(row))
    time.sleep(2)  # 2-second delay between messages
```
- **`p.produce()`**: Publishes each JSON record to the `fruit_data` topic.
- **`p.flush()`**: Forces the producer to send buffered messages immediately (ensures delivery).
- **`time.sleep(2)`**: Introduces a 2-second delay between messages to simulate real-time streaming.

### **Key Features**
- **Producer Configuration**: Connects to Kafka on `localhost:9092`.
- **JSON Data Ingestion**: Reads from `fruit_data.json`.
- **Controlled Publishing**: Messages are sent one by one with a delay.
- **Message Logging**: Prints each published message for debugging.

### **Expected Output**
```
Message published -> {"id": "1", "value": "apple", "timestamp": "2023-07-30T10:00:00Z"}
Message published -> {"id": "2", "value": "banana", "timestamp": "2023-07-30T10:00:10Z"}
...
```
(One message every 2 seconds.)

---

## **2. `join_stream_data.py` - Spark Streaming Job**

### **Purpose**
This script consumes Kafka messages, parses them, and joins them with static reference data (`fruit_dim.csv`) using **PySpark Structured Streaming**.

### **Code Breakdown**

#### **Imports**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, TimestampType
```
- `SparkSession`: Entry point for Spark SQL and DataFrames.
- `from_json`: Parses JSON strings into structured data.
- `col`: Used for column operations.
- `StructType`, `StructField`: Define schema for JSON parsing.
- `StringType`, `TimestampType`: Data types for schema.

#### **Spark Session Configuration**
```python
spark = SparkSession.builder \
    .appName("JoinStreamingAndStatic") \
    .master("local[3]") \
    .config("spark.sql.shuffle.partitions", "2") \
    .config("spark.streaming.stopGracefullyOnShutdown", "true") \
    .config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.0") \
    .getOrCreate()
```
- **`appName`**: Sets the Spark application name.
- **`master("local[3]")`**: Runs Spark locally with 3 cores.
- **`spark.sql.shuffle.partitions=2`**: Reduces shuffle partitions for local testing.
- **`stopGracefullyOnShutdown`**: Ensures clean shutdown.
- **`spark.jars.packages`**: Includes the Kafka connector for Spark.

#### **Read Kafka Stream**
```python
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "fruit_data") \
    .option("startingOffsets", "latest") \
    .load()
```
- **`format("kafka")`**: Specifies Kafka as the data source.
- **`bootstrap.servers`**: Connects to Kafka on `localhost:9092`.
- **`subscribe`**: Listens to the `fruit_data` topic.
- **`startingOffsets=latest`**: Reads only new messages (avoids historical data).

#### **Define JSON Schema**
```python
schema = StructType([
    StructField("id", StringType()),
    StructField("value", StringType()),
    StructField("timestamp", TimestampType())
])
```
- Defines the structure of JSON messages:
  - `id` (string)
  - `value` (string, fruit name)
  - `timestamp` (timestamp)

#### **Parse JSON Data**
```python
df = df.select(from_json(col("value").cast("string"), schema).alias("data")).select("data.*")
```
- **`from_json`**: Parses the JSON string in Kafka messages.
- **`select("data.*")`**: Flattens the parsed JSON into columns.

#### **Load Static Data (CSV)**
```python
df_static = spark.read.csv("/tmp/input_data/fruit_dim.csv", inferSchema=True, header=True)
```
- Reads `fruit_dim.csv` from HDFS (or local path).
- **`inferSchema=True`**: Automatically detects column types.
- **`header=True`**: Uses the first row as column names.

#### **Join Streaming + Static Data**
```python
df_joined = df.join(df_static, df.id == df_static.id, 'inner').drop(df_static.id)
```
- **Inner join** on `id` to enrich streaming data with `category`.
- **`drop(df_static.id)`**: Removes duplicate `id` column.

#### **Write Output to Console**
```python
query = df_joined.writeStream \
    .outputMode("append") \
    .format("console") \
    .option("truncate", "false") \
    .start()

query.awaitTermination()
```
- **`outputMode("append")`**: New data is continuously appended.
- **`format("console")`**: Prints results to console.
- **`truncate=False`**: Ensures full output (no truncation).
- **`awaitTermination()`**: Keeps the streaming job running.

### **Key Features**
- **Real-time Kafka Consumption**: Processes messages as they arrive.
- **Schema Enforcement**: Ensures JSON data matches expected format.
- **Static + Streaming Join**: Combines real-time data with reference data.
- **Console Output**: Displays enriched records in real time.

### **Expected Output**
```
+---+-------------+-------------------+--------+
|id |value        |timestamp          |category|
+---+-------------+-------------------+--------+
|1  |apple        |2023-07-30 10:00:00|fruit   |
|2  |banana       |2023-07-30 10:00:10|fruit   |
|22 |vanilla bean |2023-07-30 10:03:30|spice   |
|26 |zucchini     |2023-07-30 10:04:10|vegetable|
+---+-------------+-------------------+--------+
```
(Continuously updates as new messages arrive.)

---

## **Summary**
| File | Role | Key Operations |
|------|------|---------------|
| `fruit_producer.py` | Kafka Producer | Reads JSON → Publishes to Kafka topic |
| `join_stream_data.py` | Spark Streaming Job | Consumes Kafka → Joins with CSV → Prints enriched data |

This pipeline demonstrates:
✅ **Real-time data ingestion** (Kafka)  
✅ **Stream processing** (Spark Structured Streaming)  
✅ **Data enrichment** (Joining streaming + static data)  
✅ **Scalable architecture** (Can be extended to distributed clusters)

<br/>
<br/>

# **Real-Time Transaction Processing with Kafka & Spark Streaming**

This project demonstrates a **real-time data pipeline** that processes financial transactions using **Kafka for streaming** and **Spark for aggregation**. It includes two variants:
1. **Simple GroupBy Aggregation** (`kafka_spark_groupby.py`)
2. **Windowed GroupBy Aggregation** (`kafka_spark_window_groupby.py`)

---

## **1. Project Components**

### **A. `trx_data_producer.py` (Kafka Producer)**
- **Purpose**: Publishes transaction data to a Kafka topic (`trx_data`).
- **Key Features**:
  - Reads JSON data from `user_transactions.json`.
  - Uses the `confluent_kafka` Python library to produce messages.
  - Includes a **delivery report callback** for error handling.
  - Introduces a **2-second delay** between messages to simulate real-time streaming.

#### **Code Breakdown**
```python
p = Producer({'bootstrap.servers': 'localhost:9092'})  # Kafka broker config
for record in data:
    p.produce('trx_data', json.dumps(record), callback=delivery_report)  # Publish to Kafka
    time.sleep(2)  # Simulate real-time delay
```
- **Output**:  
  ```
  Message delivered to trx_data  
  Message Published -> {"user_id": "user1", "amount": 100, "timestamp": "2023-07-30T10:00:00Z"}
  ```

---

### **B. `kafka_spark_groupby.py` (Spark Streaming - Simple Aggregation)**
- **Purpose**: Consumes Kafka messages and computes **total transaction amounts per user**.
- **Key Features**:
  - Uses **PySpark Structured Streaming**.
  - Groups transactions by `user_id` and sums `amount`.
  - Outputs results in **"complete" mode** (updates full aggregation on each batch).

#### **Code Breakdown**
```python
df = spark.readStream.format("kafka")...  # Read from Kafka
schema = StructType([...])  # Define JSON schema
df = df.select(from_json(...)).select("data.*")  # Parse JSON
df = df.groupBy("user_id").agg(sum("amount").alias("total_amount"))  # Aggregate
query = df.writeStream.outputMode("complete").format("console").start()  # Write to console
```
- **Output**:  
  ```
  +-------+------------+
  |user_id|total_amount|
  +-------+------------+
  |user1  |       10000|
  |user2  |       11000|
  |user3  |       12000|
  +-------+------------+
  ```

---

### **C. `kafka_spark_window_groupby.py` (Spark Streaming - Windowed Aggregation)**
- **Purpose**: Extends the previous script to compute **rolling 3-minute transaction totals**.
- **Key Features**:
  - Uses **`window()`** function for time-based grouping.
  - Aggregates transactions in **3-minute windows**.
  - Outputs results in **"complete" mode**.

#### **Code Breakdown**
```python
df = df.groupBy("user_id", window(df.timestamp, "3 minutes")).agg(sum("amount").alias("total_amount"))
```
- **Output**:  
  ```
  +-------+---------------------------------------------+------------+
  |user_id|window                                      |total_amount|
  +-------+---------------------------------------------+------------+
  |user1  |[2023-07-30 10:00:00, 2023-07-30 10:03:00]  |        500 |
  |user2  |[2023-07-30 10:03:00, 2023-07-30 10:06:00]  |       1300 |
  |user3  |[2023-07-30 10:06:00, 2023-07-30 10:09:00]  |       2100 |
  +-------+---------------------------------------------+------------+
  ```

---

### **D. `user_transactions.json` (Sample Data)**
- Contains **51 transaction records** with:
  - `user_id` (3 users: `user1`, `user2`, `user3`)
  - `amount` (increasing values for demo purposes)
  - `timestamp` (ISO format, spanning 50 minutes)

Example:
```json
{"user_id": "user1", "amount": 100, "timestamp": "2023-07-30T10:00:00Z"}
{"user_id": "user2", "amount": 200, "timestamp": "2023-07-30T10:01:00Z"}
```

---

## **2. Key Concepts & Workflow**
1. **Kafka Producer** (`trx_data_producer.py`)  
   - Publishes transactions to Kafka topic `trx_data`.  
   - Simulates real-time data with a **2-second delay**.

2. **Spark Streaming Job** (Two Variants)  
   - **Simple Aggregation** (`kafka_spark_groupby.py`):  
     - Computes **total spend per user** (all-time).  
   - **Windowed Aggregation** (`kafka_spark_window_groupby.py`):  
     - Computes **rolling 3-minute totals** for each user.  

3. **Output Modes**  
   - **Complete Mode**: Recomputes & displays full results on each batch.  
   - **Append Mode** (not used here): Only shows new records.  

4. **Schema Enforcement**  
   - Defines `StructType` to parse JSON correctly:
     ```python
     schema = StructType([
         StructField("user_id", StringType()),
         StructField("amount", IntegerType()),
         StructField("timestamp", TimestampType())
     ])
     ```

---

## **3. Expected Outputs**
### **A. Simple GroupBy (`kafka_spark_groupby.py`)**
```
Batch: 1
+-------+------------+
|user_id|total_amount|
+-------+------------+
|user1  |        5000|
|user2  |        6000|
|user3  |        7000|
+-------+------------+
```

### **B. Windowed GroupBy (`kafka_spark_window_groupby.py`)**
```
Batch: 1
+-------+---------------------------------------------+------------+
|user_id|window                                      |total_amount|
+-------+---------------------------------------------+------------+
|user1  |[2023-07-30 10:00:00, 2023-07-30 10:03:00]  |        500 |
|user2  |[2023-07-30 10:03:00, 2023-07-30 10:06:00]  |       1300 |
+-------+---------------------------------------------+------------+
```

---

## **4. Use Cases**
✅ **Real-time transaction monitoring**  
✅ **Fraud detection** (unusual spending patterns)  
✅ **User spending analytics** (per time window)  
✅ **Dynamic dashboards** (live updates via Spark Streaming)  

---

## **5. How to Run**
1. **Start Kafka & Zookeeper**  
2. **Run Producer**  
   ```bash
   python trx_data_producer.py
   ```
3. **Run Spark Job**  
   ```bash
   spark-submit kafka_spark_groupby.py  # Simple aggregation
   # OR
   spark-submit kafka_spark_window_groupby.py  # Windowed aggregation
   ```

---

## **Summary**
| File | Role | Key Feature |
|------|------|-------------|
| `trx_data_producer.py` | Kafka Producer | Publishes transaction data |
| `kafka_spark_groupby.py` | Spark Job | Simple `groupBy` aggregation |
| `kafka_spark_window_groupby.py` | Spark Job | **3-minute windowed** aggregation |

This project demonstrates **real-time data processing** with **Kafka + Spark Streaming**, covering both **simple and windowed aggregations**. 🚀


<br/>
<br/>

Here's a detailed breakdown of all three Python files in this Kafka-Spark streaming project:

### 1. `trx_data_producer.py` (Kafka Producer)

**Purpose**:  
Publishes transaction data from a JSON file to a Kafka topic in a simulated real-time stream.

**Key Components**:
```python
from confluent_kafka import Producer
import json
import time

# Delivery callback to handle message delivery reports
def delivery_report(err, msg):
    if err:
        print(f"Delivery failed: {err}")
    else:
        print(f"Delivered to {msg.topic()}")

# Initialize Kafka producer
p = Producer({'bootstrap.servers': 'localhost:9092'})

# Load transaction data
with open('user_transactions.json') as f:
    data = json.load(f)

# Publish each record with 2-second delay
for record in data:
    p.poll(0)  # Serve delivery callbacks
    record_str = json.dumps(record)
    p.produce('trx_data', record_str, callback=delivery_report)
    print("Published:", record_str)
    time.sleep(2)  # Simulate real-time delay

p.flush()  # Ensure all messages are delivered
```

**Key Features**:
- Uses `confluent_kafka.Producer` for message publishing
- Includes delivery callback for error handling
- Simulates real-time streaming with 2-second delays
- Publishes to the `trx_data` topic
- Prints confirmation of each published message

**Data Flow**:
1. Reads JSON file (`user_transactions.json`)
2. Serializes each record to string
3. Publishes to Kafka with delivery confirmation
4. Introduces delay between messages

---

### 2. `kafka_spark_groupby.py` (Basic Spark Aggregation)

**Purpose**:  
Consumes Kafka messages and calculates running totals per user.

**Key Components**:
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, sum
from pyspark.sql.types import *

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("GroupByStreaming") \
    .config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.0") \
    .getOrCreate()

# Define schema for JSON parsing
schema = StructType([
    StructField("user_id", StringType()),
    StructField("amount", IntegerType()),
    StructField("timestamp", TimestampType())
])

# Read from Kafka
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "trx_data") \
    .load()

# Parse JSON and aggregate
df = df.select(
    from_json(col("value").cast("string"), schema).alias("data")
).select("data.*") \
.groupBy("user_id") \
.agg(sum("amount").alias("total_amount"))

# Output to console
query = df.writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

**Key Features**:
- Uses Spark Structured Streaming
- Defines schema for JSON parsing
- Groups by `user_id` and sums amounts
- Uses "complete" output mode to show full aggregates
- Writes results to console

**Processing Logic**:
1. Consumes messages from `trx_data` topic
2. Parses JSON using defined schema
3. Performs `groupBy` + `sum` aggregation
4. Continuously updates and displays results

---

### 3. `kafka_spark_window_groupby.py` (Windowed Spark Aggregation)

**Purpose**:  
Enhances the basic aggregation by adding time-based windowing.

**Key Differences from Basic Version**:
```python
from pyspark.sql.functions import window  # Additional import

# Modified aggregation with 3-minute windows
df = df.groupBy(
    "user_id", 
    window(df.timestamp, "3 minutes")  # Time window
).agg(sum("amount").alias("total_amount"))
```

**Key Features**:
- Adds `window()` function to group by time
- 3-minute non-overlapping windows
- Same "complete" output mode
- Includes window metadata in results

**Window Logic Example**:
For timestamp `10:00:00`, window boundaries would be:
- Start: `10:00:00`
- End: `10:03:00`

**Output Structure**:
```
+-------+---------------------------------------------+------------+
|user_id|window                                      |total_amount|
+-------+---------------------------------------------+------------+
|user1  |[2023-07-30 10:00:00, 2023-07-30 10:03:00]  |        500 |
|user2  |[2023-07-30 10:03:00, 2023-07-30 10:06:00]  |       1300 |
+-------+---------------------------------------------+------------+
```

---

### Comparison Table

| Feature                      | Producer | Basic Spark | Windowed Spark |
|------------------------------|----------|-------------|-----------------|
| Reads from Kafka             | ❌       | ✅          | ✅              |
| Writes to Kafka              | ✅       | ❌          | ❌              |
| Simple aggregation           | ❌       | ✅          | ❌              |
| Time-window aggregation      | ❌       | ❌          | ✅              |
| Output mode                  | N/A      | Complete    | Complete        |
| Requires schema definition   | ❌       | ✅          | ✅              |

---

### Execution Flow

1. **Producer** sends data to Kafka:
   ```
   Published: {"user_id": "user1", "amount": 100, "timestamp": "2023-07-30T10:00:00Z"}
   ```

2. **Spark Job** (Basic) processes and outputs:
   ```
   +-------+------------+
   |user_id|total_amount|
   +-------+------------+
   |user1  |        100 |
   ```

3. **Spark Job** (Windowed) processes and outputs:
   ```
   +-------+---------------------------------------------+------------+
   |user_id|window                                      |total_amount|
   +-------+---------------------------------------------+------------+
   |user1  |[2023-07-30 10:00:00, 2023-07-30 10:03:00]  |        100 |
   ```

---

### Use Case Examples

1. **Basic Aggregation**:
   - Real-time dashboard showing total purchases per customer
   - Simple alerting when thresholds are crossed

2. **Windowed Aggregation**:
   - Detecting spending spikes in 3-minute intervals
   - Time-based trend analysis
   - Session-based analytics

This implementation showcases a complete real-time data pipeline from ingestion (Kafka) to processing (Spark) with two different aggregation strategies.

<br/>
<br/>

# Spark Streaming Assignment: Real-Time Advertisement Data Aggregation - Detailed Explanation

## Assignment Overview

This assignment requires you to build an end-to-end real-time data processing pipeline that:
1. Ingests advertisement data through Kafka
2. Processes it using Spark Streaming with windowed aggregations
3. Stores the results in Cassandra with upsert functionality

## Core Components

### 1. Data Source Requirements
- **Kafka Topic**: `ads_data`
- **Data Format** (JSON with Avro serialization):
  ```json
  {
    "ad_id": "12345",
    "timestamp": "2023-08-23T12:01:05Z",
    "clicks": 5,
    "views": 10,
    "cost": 50.75
  }
  ```

### 2. Implementation Tasks

#### A. Kafka Setup & Mock Data Producer
- **Setup**:
  - Install and configure Confluent Kafka (local or cloud)
  - Create `ads_data` topic
- **Producer Development**:
  - Python script generating random mock data in the specified format
  - Must use **Avro serialization** (not plain JSON)
  - Should continuously publish data to simulate real-time stream

#### B. Spark Streaming Application
- **Data Ingestion**:
  - Use Spark's Kafka connector to consume from `ads_data`
  - Properly deserialize Avro-formatted messages
  - Parse into structured DataFrame with correct schema

- **Windowed Aggregations**:
  - **Window Duration**: 1 minute
  - **Sliding Interval**: 30 seconds
  - Required aggregations:
    - `total_clicks` = SUM(clicks) per ad_id
    - `total_views` = SUM(views) per ad_id
    - `avg_cost_per_view` = SUM(cost)/SUM(views) per ad_id

#### C. Cassandra Integration
- **Table Design**:
  - Primary key: `ad_id`
  - Columns: aggregated metrics + timestamp of last update
- **Write Logic**:
  - **UPSERT** functionality:
    - If ad_id exists: increment clicks/views, recalculate average
    - If new ad_id: insert new record
  - Should handle duplicate data gracefully

### 3. Technical Specifications

| Component | Technology | Key Requirements |
|-----------|------------|------------------|
| Streaming Source | Kafka | Avro serialization, continuous data flow |
| Processing | Spark Structured Streaming | Windowed aggregations, state management |
| Storage | Cassandra | Upsert operations, efficient key-value storage |

### 4. Expected Output Metrics

For each advertisement in each time window, your solution should track:
1. **Engagement Metrics**:
   - Total clicks accumulated
   - Total views accumulated
2. **Financial Metrics**:
   - Average cost per view (total cost/total views)
3. **Temporal Context**:
   - Window start/end timestamps

### 5. Submission Requirements

1. **Code**:
   - Kafka producer script (Python)
   - Spark Streaming application (Scala/Python)
   - Cassandra table schema definition
2. **Report**:
   - Architecture diagram
   - Challenges faced and solutions
   - Sample output with explanation
   - Any optimizations implemented

## Implementation Deep Dive

### Windowed Aggregation Logic Example

```python
# Pseudo-code for Spark windowed aggregation
df.withWatermark("timestamp", "10 minutes") \
  .groupBy(
    window("timestamp", "1 minute", "30 seconds"),
    col("ad_id")
  ).agg(
    sum("clicks").alias("total_clicks"),
    sum("views").alias("total_views"),
    (sum("cost")/sum("views")).alias("avg_cost_per_view")
  )
```

### Cassandra Upsert Pattern

```sql
-- Example Cassandra table schema
CREATE TABLE ad_metrics (
  ad_id text PRIMARY KEY,
  total_clicks bigint,
  total_views bigint,
  avg_cost_per_view decimal,
  last_updated timestamp
);
```

```python
# Spark-Cassandra write logic
df.write \
  .format("org.apache.spark.sql.cassandra") \
  .option("keyspace", "ads") \
  .option("table", "ad_metrics") \
  .mode("append") \  # Actual upsert handled by Cassandra
  .save()
```

## Key Challenges to Address

1. **Data Serialization**:
   - Proper Avro schema handling in both producer and consumer
   - Schema evolution considerations

2. **State Management**:
   - Handling late-arriving data with watermarks
   - Avoiding duplicate counting in sliding windows

3. **Cassandra Optimization**:
   - Efficient upsert operations
   - Partition key design for read performance

4. **Production Considerations**:
   - Fault tolerance (checkpointing in Spark)
   - Monitoring pipeline health
   - Performance tuning for high-volume data

This assignment tests your ability to implement a complete, production-grade streaming pipeline with complex business logic and reliable data persistence.

<br/>
<br/>

# Real-Time Advertisement Data Processing System: Detailed Explanation

This project implements a complete pipeline for processing real-time advertisement data using Kafka, Avro serialization, and Spark Streaming. The system consists of three main components: a producer, a consumer, and a Spark Streaming application.

## 1. System Architecture Overview

```
[Producer (producer.py)] → [Kafka Cluster] → [Spark Streaming (stream_ads.py)]
                                      ↘
                                        → [Direct Consumer (consumer.py)]
```

## 2. Producer Component (`producer.py`)

### Key Features:
- **Data Ingestion**: Reads mock advertisement data from a CSV file
- **Avro Serialization**: Uses Confluent's Schema Registry for schema management
- **Secure Connection**: SASL/SSL authentication to Confluent Cloud

### Detailed Workflow:
1. **Initialization**:
   - Loads environment variables for Kafka and Schema Registry credentials
   - Configures SerializingProducer with Avro serializer
   - Connects to Confluent Schema Registry (hosted on AWS)

2. **Data Preparation**:
   ```python
   df = pd.read_csv('mock_data.csv', dtype={"ad_id": str})
   ```
   - Reads CSV data with explicit type casting for ad_id

3. **Message Production**:
   ```python
   producer.produce(topic=f"{topic}", key=str(index), value=value, on_delivery=delivery_report)
   ```
   - Publishes each row as an Avro-serialized message
   - Includes delivery callback for error handling
   - Flushes after each message (for reliability)

### Security Configuration:
```python
{
    'bootstrap.servers': 'pkc-l7pr2.ap-south-1.aws.confluent.cloud:9092',
    'sasl.mechanisms': 'PLAIN',
    'security.protocol': 'SASL_SSL',
    'sasl.username': kafka_id,
    'sasl.password': kafka_secret_key
}
```

## 3. Consumer Component (`consumer.py`)

### Key Features:
- **Avro Deserialization**: Uses Schema Registry to validate messages
- **Consumer Group**: Part of 'group11' for offset management
- **Error Handling**: Robust message polling with error detection

### Message Processing Logic:
```python
while True:
    msg = consumer.poll(1.0)
    if msg is None: continue
    if msg.error(): print('Consumer error')
    print('Consumed record:', msg.key(), msg.value())
```
- Continuously polls for new messages
- Handles errors gracefully
- Prints deserialized Avro payloads

### Schema Management:
```python
schema_str = schema_registry_client.get_latest_version(subject_name).schema.schema_str
avro_deserializer = AvroDeserializer(schema_registry_client, schema_str)
```
- Dynamically fetches latest schema version
- Ensures schema compatibility

## 4. Spark Streaming Application (`stream_ads.py`)

### Core Functionality:
- **Kafka Integration**: Reads from 'ad_topic' with earliest offset
- **Avro Deserialization**: Uses Spark's built-in from_avro function
- **Stream Processing**: Basic console output (foundation for aggregations)

### Key Configurations:
```python
.config("spark.jars.package", ",".join([
    "org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0",
    "org.apache.spark:spark-avro_2.12:3.5.0"
]))
```
- Includes necessary Kafka and Avro dependencies

### Stream Processing Pipeline:
```python
df = spark.readStream.format("kafka").options(**kafka_options).load()
data = df.select(from_avro("value", schema_str).alias("ctr")).select("ctr.*")
```
1. Reads raw bytes from Kafka
2. Deserializes Avro payload using schema from Registry
3. Explodes nested structure into columns

### Output Configuration:
```python
.writeStream 
  .outputMode("update") 
  .format("console")
  .trigger(processingTime="3 second")
```
- Prints updates to console every 3 seconds
- Uses "update" mode to show only changed records

## 5. Schema Management

All components share the same schema handling approach:
1. Connect to Schema Registry:
   ```python
   SchemaRegistryClient({
     'url': 'https://psrc-kjwmg.ap-southeast-2.aws.confluent.cloud',
     'basic.auth.user.info': f'{schema_id}:{schema_secret}'
   })
   ```
2. Fetch schema by subject name:
   ```python
   subject_name = "ad_topic-value"
   schema_str = client.get_latest_version(subject_name).schema.schema_str
   ```

## 6. Security Implementation

All components implement:
- SASL/SSL authentication
- Environment variables for credentials (via dotenv)
- Encrypted connections to Confluent Cloud

```python
'security.protocol': 'SASL_SSL',
'sasl.mechanisms': 'PLAIN',
'sasl.username': kafka_id,
'sasl.password': kafka_secret_key
```

## 7. Future Enhancement Points

1. **Windowed Aggregations**:
   ```python
   .groupBy(window("timestamp", "1 minute", "30 seconds"), "ad_id")
   .agg(sum("clicks"), avg("cost"))
   ```

2. **Cassandra Sink**:
   ```python
   .format("org.apache.spark.sql.cassandra")
   .option("table", "ad_metrics")
   ```

3. **Watermarking**:
   ```python
   .withWatermark("timestamp", "10 minutes")
   ```

This implementation provides a robust foundation for real-time advertisement analytics, with all components properly secured and leveraging Avro's schema evolution capabilities.

<br/>
<br/>

Here's a detailed explanation of all three Python files in this real-time advertisement data processing system:

---

## **1. `producer.py` (Kafka Producer with Avro Serialization)**

### **Purpose**
This script generates mock advertisement data and publishes it to a Kafka topic (`ad_topic`) using **Avro serialization** for schema enforcement.

### **Key Components**

#### **1. Initialization & Configuration**
- **Kafka Setup**:
  ```python
  kafka_config = {
      'bootstrap.servers': 'pkc-l7pr2.ap-south-1.aws.confluent.cloud:9092',
      'sasl.mechanisms': 'PLAIN',
      'security.protocol': 'SASL_SSL',
      'sasl.username': os.getenv("confluent_kafka_id"),
      'sasl.password': os.getenv("confluent_kafka_secret_key")
  }
  ```
  - Connects to **Confluent Cloud Kafka** with SASL/SSL authentication.
  
- **Schema Registry Integration**:
  ```python
  schema_registry_client = SchemaRegistryClient({
      'url': 'https://psrc-kjwmg.ap-southeast-2.aws.confluent.cloud',
      'basic.auth.user.info': f'{schema_id}:{schema_secret}'
  })
  ```
  - Fetches the latest Avro schema for the topic from **Confluent Schema Registry**.

#### **2. Data Ingestion**
- Reads mock data from `mock_data.csv`:
  ```python
  df = pd.read_csv('mock_data.csv', dtype={"ad_id": str})
  ```
  - Ensures `ad_id` is treated as a string.

#### **3. Message Production**
- **Serialization**:
  ```python
  avro_serializer = AvroSerializer(schema_registry_client, schema_str)
  ```
  - Converts Python dictionaries to Avro-serialized bytes.
  
- **Publishing**:
  ```python
  producer.produce(
      topic="ad_topic",
      key=str(index),  # Simple key (message index)
      value=row._asdict(),  # Avro-serialized value
      on_delivery=delivery_report  # Callback for delivery status
  )
  ```
  - Uses `SerializingProducer` for schema-compliant serialization.
  - Includes a **delivery callback** to log successes/failures.

#### **4. Output Example**
```
User record 0 successfully produced to ad_topic [0] at offset 45
```

---

## **2. `consumer.py` (Kafka Consumer with Avro Deserialization)**

### **Purpose**
Consumes messages from the `ad_topic`, deserializes them using Avro, and prints the results.

### **Key Components**

#### **1. Consumer Configuration**
- **Group Management**:
  ```python
  'group.id': 'group11',
  'auto.offset.reset': 'earliest'
  ```
  - Part of consumer group `group11` for offset tracking.
  - Starts from the earliest message if no offset is committed.

- **Avro Deserialization**:
  ```python
  avro_deserializer = AvroDeserializer(schema_registry_client, schema_str)
  ```
  - Validates messages against the schema from the registry.

#### **2. Message Consumption**
- **Polling Loop**:
  ```python
  while True:
      msg = consumer.poll(1.0)  # Wait up to 1 second
      if msg.error():
          print(f"Error: {msg.error()}")
      else:
          print(f"Key: {msg.key()}, Value: {msg.value()}")
  ```
  - Continuously polls Kafka for new messages.
  - Logs errors and prints deserialized data.

#### **3. Output Example**
```
Key: 0, Value: {
    'ad_id': '12345',
    'timestamp': '2023-08-23T12:01:05Z',
    'clicks': 5,
    'views': 10,
    'cost': 50.75
}
```

---

## **3. `stream_ads.py` (Spark Streaming Application)**

### **Purpose**
Processes the Kafka stream in real-time using Spark Structured Streaming, with support for **Avro deserialization** and future aggregation capabilities.

### **Key Components**

#### **1. Spark Session Setup**
- **Dependencies**:
  ```python
  packages = [
      "org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0",
      "org.apache.spark:spark-avro_2.12:3.5.0"
  ]
  ```
  - Includes Kafka and Avro connectors for Spark.

- **Configuration**:
  ```python
  .config("spark.sql.shuffle.partitions", "2")
  .config("spark.streaming.stopGracefullyOnShutdown", "true")
  ```
  - Optimized for local execution with graceful shutdown.

#### **2. Kafka Source**
- **Stream Definition**:
  ```python
  df = spark.readStream.format("kafka").options(**kafka_options).load()
  ```
  - Subscribes to `ad_topic` with SASL/SSL auth.

#### **3. Avro Deserialization**
- **Schema Handling**:
  ```python
  schema_str = schema_registry_client.get_latest_version("ad_topic-value").schema.schema_str
  ```
  - Dynamically fetches the schema from the registry.

- **Data Parsing**:
  ```python
  data = df.select(from_avro("value", schema_str).alias("ctr")).select("ctr.*")
  ```
  - Uses Spark’s `from_avro` to decode messages into a DataFrame.

#### **4. Console Sink**
- **Output**:
  ```python
  query = data.writeStream
      .outputMode("update")
      .format("console")
      .trigger(processingTime="3 seconds")
      .start()
  ```
  - Prints new data to the console every **3 seconds**.
  - Uses `update` mode to show only changed records.

#### **5. Output Example**
```
Batch: 1
+------+-------------------+------+-----+-----+
|ad_id |timestamp          |clicks|views|cost |
+------+-------------------+------+-----+-----+
|12345 |2023-08-23 12:01:05|5     |10   |50.75|
+------+-------------------+------+-----+-----+
```

---

## **Comparison of Components**

| **Feature**               | **Producer**                          | **Consumer**                          | **Spark Stream**                     |
|---------------------------|---------------------------------------|---------------------------------------|--------------------------------------|
| **Role**                  | Publishes data to Kafka               | Directly consumes messages            | Processes streams in real-time       |
| **Serialization**         | Avro                                  | Avro                                  | Avro                                 |
| **Authentication**        | SASL/SSL                              | SASL/SSL                              | SASL/SSL                             |
| **Schema Management**     | Confluent Schema Registry             | Confluent Schema Registry             | Confluent Schema Registry            |
| **Output**                | Kafka topic                           | Console logs                          | Console or database (future)         |
| **Use Case**              | Data ingestion                        | Debugging/validation                  | Aggregation & analytics              |

---

## **Key Takeaways**
1. **End-to-End Schema Enforcement**: All components use Avro with a central Schema Registry.
2. **Secure Cloud Integration**: SASL/SSL auth for Confluent Cloud.
3. **Spark for Scalability**: Foundation for windowed aggregations (e.g., clicks per minute).
4. **Operational Insights**:
   - Producer: Tracks successful/failed deliveries.
   - Consumer: Validates data integrity.
   - Spark: Enables real-time analytics.

This pipeline can be extended with:
- **Windowed aggregations** in Spark.
- **Cassandra sink** for persistent storage.
- **Watermarking** for late-data handling.

<br/>
<br/>

# **Windowing in Stream Processing: When and Why to Use It**

Windowing is a fundamental concept in stream processing that divides continuous data streams into finite chunks (windows) for analysis. Below is a detailed explanation of when and why to use windowing, along with practical examples.

---

## **1. When to Use Windowing?**
Windowing is useful in the following scenarios:

### **A. Aggregating Data Over Time**
**Use Case**: Calculating metrics over fixed time intervals.  
**Example**:  
- **E-commerce**: "Total sales per hour"  
- **IoT**: "Average temperature per 5-minute interval"  
- **Ad Tech**: "Clicks per ad campaign every 30 minutes"  

**Why?**  
- Raw event streams are unbounded; windowing converts them into finite chunks for aggregation.
- Enables time-based rollups (e.g., hourly/daily reports).

**Implementation (Spark Example)**:
```python
df.groupBy(
    window("timestamp", "1 hour"),
    "ad_id"
).agg(sum("clicks").alias("total_clicks"))
```

---

### **B. Detecting Trends & Patterns**
**Use Case**: Identifying changes in data behavior over time.  
**Example**:  
- **Fraud Detection**: "Unusual spikes in transactions per 10-minute window"  
- **Log Monitoring**: "Error rate trends in 15-minute windows"  
- **Stock Market**: "Moving average of stock prices over 5-minute windows"  

**Why?**  
- Helps observe how metrics evolve (e.g., sudden drops/spikes).
- Enables anomaly detection (e.g., "Alert if errors exceed 100 in 5 minutes").

**Implementation (Spark Example)**:
```python
df.groupBy(
    window("timestamp", "15 minutes")
).agg(count("error").alias("error_count"))
.filter("error_count > 100")
```

---

### **C. Handling Late Data**
**Use Case**: Processing delayed events (common in distributed systems).  
**Example**:  
- **Sensor Data**: "Late-arriving temperature readings due to network lag"  
- **Mobile Analytics**: "Delayed app session logs from offline devices"  

**Why?**  
- Without windowing, late data could be ignored or misaggregated.
- **Watermarking** (combined with windowing) defines how late data can arrive while still being processed.

**Implementation (Spark Example)**:
```python
df.withWatermark("timestamp", "10 minutes")  # Accept data up to 10 mins late
  .groupBy(
      window("timestamp", "5 minutes")
  ).count()
```

---

### **D. Resource Efficiency**
**Use Case**: Reducing memory/compute overhead.  
**Example**:  
- **Clickstream Analysis**: "Compute unique visitors per 1-hour window instead of entire history"  
- **Server Monitoring**: "CPU usage averages per 5-minute window"  

**Why?**  
- Processing infinite streams requires unbounded memory.
- Windowing limits the working dataset to recent intervals.

**Implementation (Spark Example)**:
```python
df.groupBy(
    window("timestamp", "1 hour", "30 minutes")  # Sliding window
).agg(approx_count_distinct("user_id"))
```

---

## **2. Types of Windows**
| Type | Description | Example Use Case |
|------|-------------|------------------|
| **Tumbling** | Fixed-size, non-overlapping windows | "Daily sales totals" |
| **Sliding** | Fixed-size, overlapping windows | "Moving average over 5 mins, updated every 1 min" |
| **Session** | Dynamic windows based on inactivity gaps | "User session analytics" |

### **A. Tumbling Windows**
- **Fixed-length**, contiguous intervals (e.g., every 1 hour).  
- No overlap between windows.  

**Example**:
```python
df.groupBy(
    window("timestamp", "1 hour")  # Tumbling window
).sum("sales")
```

### **B. Sliding Windows**
- **Fixed-length**, but with overlap (e.g., 5-minute window, sliding every 1 minute).  
- Useful for smoother trend analysis.  

**Example**:
```python
df.groupBy(
    window("timestamp", "5 minutes", "1 minute")  # 5-min window, sliding every 1 min
).avg("temperature")
```

### **C. Session Windows**
- **Variable-length** based on activity gaps (e.g., 30-minute inactivity resets the window).  
- Ideal for user behavior analysis.  

**Example**:
```python
df.groupBy(
    session_window("timestamp", "30 minutes")  # Session gap of 30 mins
).count("page_views")
```

---

## **3. Why Windowing is Critical?**
1. **Enables Time-Based Analysis**  
   - Raw streams lack temporal boundaries; windowing adds structure.  
2. **Reduces Computational Load**  
   - Instead of processing all historical data, focus on recent windows.  
3. **Handles Real-World Imperfections**  
   - Late data (watermarking) + out-of-order events.  
4. **Supports Business Logic**  
   - "Hourly active users" or "Peak traffic periods" require windowed aggregates.  

---

## **4. Practical Example (Advertisement Analytics)**
**Scenario**:  
- Calculate **total clicks per ad every 30 seconds**, updated every 10 seconds (sliding window).  
- Handle late data (up to 1 minute delay).  

**Spark Code**:
```python
from pyspark.sql.functions import window, sum

df.withWatermark("timestamp", "1 minute") \
  .groupBy(
      "ad_id",
      window("timestamp", "30 seconds", "10 seconds")  # Sliding window
  ).agg(sum("clicks").alias("total_clicks")) \
  .writeStream \
  .outputMode("update") \
  .format("console") \
  .start()
```

**Output**:
```
+-------+---------------------------------------------+------------+
|ad_id  |window                                      |total_clicks|
+-------+---------------------------------------------+------------+
|ad123  |[2023-08-23 12:00:00, 2023-08-23 12:00:30]  |150         |
|ad456  |[2023-08-23 12:00:10, 2023-08-23 12:00:40]  |220         |
+-------+---------------------------------------------+------------+
```

---

## **5. Key Takeaways**
✅ **Use tumbling windows** for fixed-period reports (e.g., hourly aggregates).  
✅ **Use sliding windows** for rolling metrics (e.g., moving averages).  
✅ **Combine with watermarking** to handle late data gracefully.  
✅ **Optimize resource usage** by limiting computation to relevant time intervals.  

Windowing transforms unstructured streams into actionable insights while addressing real-world challenges like late data and resource constraints.