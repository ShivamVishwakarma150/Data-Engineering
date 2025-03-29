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