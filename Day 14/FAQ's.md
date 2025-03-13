# **📌 Cassandra Interview Questions**

1. **What is Apache Cassandra?**  
2. **What is a token in Cassandra?**  
3. **What are the differences between RDBMS and Cassandra?**  
4. **Can you explain what SSTable is in Cassandra?**  
5. **What is a Column Family in Cassandra?**  
6. **Explain CAP theorem. How is it related to Cassandra?**  
7. **What is the purpose of using Cassandra Query Language (CQL)?**  
8. **What is tunable consistency in Cassandra?**  
9. **What are some of the key features of Cassandra?**  
10. **What is a keyspace in Cassandra?**  
11. **What are the different types of Replication Strategies in Cassandra?**  
12. **Explain Compaction in Cassandra.**  
13. **What is the Bloom filter in Cassandra?**  
14. **What is Snitch in Cassandra?**  
15. **What are Cassandra-Stress tools and their usage?**  
16. **What is Gossip protocol in Cassandra?**  
17. **What is a Commit Log in Cassandra?**  
18. **What is Read Repair in Cassandra?**  
19. **How is data stored in Cassandra?**  
20. **Explain the role of the Coordinator node in Cassandra.**  
21. **What is the Hinted Handoff in Cassandra?**  
22. **What is Apache Cassandra's write pattern?**  
23. **What is Lightweight Transaction in Cassandra?**  
24. **What is a Composite Key in Cassandra?**  
25. **Can you explain how Cassandra handles data modifications?**  
26. **What are the different types of tombstones in Cassandra?**  
27. **What is a Super Column in Cassandra?**  
28. **Explain the role of Memtable in Cassandra.**  
29. **What is meant by 'consistent hashing' in Cassandra?**  
30. **What is Time to Live (TTL) in Cassandra?**  
31. **What are seeds in Cassandra?**  
32. **What is the role of the Partitioner in Cassandra?**  
33. **What is the difference between deleting and expiring in Cassandra?**  
34. **What are Repair and Anti-Entropy in Cassandra?**  
35. **What is a Materialized View in Cassandra?**  
36. **What is the role of CQLSH in Cassandra?**  
37. **Explain the difference between a node, a cluster, and a data center in Cassandra.**  
38. **What is a counter column in Cassandra?**  
39. **Explain what "write heavy" and "read heavy" mean in the context of Cassandra.**  
40. **What are dynamic columns in Cassandra?**  
41. **What is a Batch in Cassandra?**  
42. **What is Rack Awareness in Cassandra?**  
43. **What is a Secondary Index in Cassandra?**  
44. **What is the role of the Cassandra.yaml file?**  
45. **What are Collection data types in Cassandra?**  
46. **What is Paxos in Cassandra?**  
47. **What is a wide row in Cassandra?**  
48. **Explain the difference between 'QUORUM' and 'LOCAL_QUORUM' consistency levels.**  
49. **What are the limitations of using Secondary Indexes in Cassandra?**  
50. **What is a Thrift in Cassandra?**  
51. **Explain the differences between 'ONE', 'TWO', 'THREE', and 'ALL' consistency levels in Cassandra.**  
52. **What are prepared statements in Cassandra and why would you use them?**  
53. **What is 'tombstone garbage collection grace seconds'?**  
54. **What is a Super Column Family in Cassandra?**  
55. **How does Cassandra handle concurrent writes?**  
56. **What is the purpose of the System Keyspace in Cassandra?**  
57. **What are some use cases where you would not want to use Cassandra?**  
58. **What is a Column Family Store in Cassandra?**  
59. **How can you secure your Cassandra deployment?**  
60. **What is the Hinted Handoff in Cassandra?**  
61. **How does Cassandra handle conflicts during replication?**  
62. **What is the purpose of Cassandra's Read Repair mechanism?**  
63. **How does the Gossip protocol work in Cassandra?**  
64. **What's the difference between Levelled Compaction and Size Tiered Compaction?**  
65. **What is a Bloom Filter and how does it work in Cassandra?**  
66. **How does Cassandra ensure Durability?**  
67. **What is the purpose of the Commit Log in Cassandra?**  
68. **How can you model time-series data in Cassandra?**  
69. **Explain Lightweight Transactions in Cassandra.**  
70. **How does Cassandra handle data compression?**  
71. **Why does Cassandra not support joins?**  
72. **What is Eventual Consistency in Cassandra?**  
73. **What is the concept of 'Tunable Consistency' in Cassandra?**  
74. **How does data distribution work in multi-datacenter deployments of Cassandra?**  
75. **Why is Cassandra suitable for IoT use cases?**  
76. **Explain how compaction works in Cassandra.**  
77. **What are the different types of keys in Cassandra and how are they used?**  
78. **What are SSTables in Cassandra?**  
79. **How does Cassandra handle failures?**  
80. **What is Cassandra's Snitch and what does it do?**  
81. **What happens when you run out of disk space in Cassandra?**  
82. **What is Apache Cassandra's strategy for handling data evictions?**  
83. **What happens when a Cassandra node goes down during a write operation?**  
84. **How can you minimize read latencies in Cassandra?**  
85. **What is the impact of Consistency Level on Cassandra's performance?**  
86. **What is the purpose of Apache Cassandra's Coordinator node?**  
87. **How can you mitigate the impact of "wide rows" in Cassandra?**  
88. **What is vnode and what is its purpose in Cassandra?**  
89. **How does Cassandra handle large blobs of data?**  
90. **Explain how Tombstones work in Cassandra.**  

---

## **1️⃣ What is Apache Cassandra?**
**Apache Cassandra** is a **highly scalable, distributed NoSQL database** designed to handle large amounts of data **across multiple nodes** with **high availability** and **fault tolerance**.

### **Key Features:**
✔ **Decentralized (Peer-to-Peer)** – No master node, all nodes are equal.  
✔ **Linear Scalability** – Add more nodes for more capacity.  
✔ **High Availability** – No single point of failure.  
✔ **Fault Tolerance** – Data is replicated across multiple nodes.  
✔ **Tunable Consistency** – Balance between strong and eventual consistency.  
✔ **Schema-Free** – Uses a **wide-column store** instead of traditional RDBMS tables.

### **Use Cases:**
✅ IoT & Sensor Data  
✅ E-commerce & Recommendation Systems  
✅ Social Media & Messaging Apps  
✅ Fraud Detection & Security Systems  

---

## **2️⃣ What is a Token in Cassandra?**
A **token** in Cassandra is a **hashed value** used to **determine the placement of data** in the cluster.

### **How It Works?**
- Cassandra **partitions data** using a **hash function** (Murmur3 by default).
- Each node is **responsible for a range of tokens**.
- When data is inserted, **the partition key is hashed**, and Cassandra finds the **node responsible** for that token.

### **Example:**
- **Hash Function:** `Murmur3("customer_id_123") = Token 45321`
- If **Node 1 owns Token Range (40000 - 50000)**, it stores the data.

✅ **Tokens help in efficient data distribution and load balancing.**  

---

## **3️⃣ What are the differences between RDBMS and Cassandra?**

| Feature        | RDBMS (MySQL, PostgreSQL) | Cassandra |
|---------------|------------------|-----------|
| **Data Model** | Tables, Rows, Columns | Wide-Column Store |
| **Schema** | Strict schema (fixed structure) | Flexible schema (columns can vary) |
| **Query Language** | SQL | CQL (Cassandra Query Language) |
| **Scalability** | Vertical (more CPU/RAM on a single server) | Horizontal (add more nodes to a cluster) |
| **Availability** | High Read Consistency | High Availability |
| **Joins & ACID** | Supports Joins, ACID Transactions | No Joins, Eventual Consistency |
| **Partitioning** | Centralized | Distributed across multiple nodes |

✅ **Use Cassandra** when **scalability & high availability** are more important than complex transactions.

---

## **4️⃣ Can you explain what SSTable is in Cassandra?**
An **SSTable (Sorted String Table)** is an **immutable data file** stored on disk in Cassandra.

### **How SSTables Work?**
1. **Writes are first stored in Memtable** (in-memory).
2. **When Memtable is full**, data is flushed to an **SSTable** on disk.
3. **SSTables are immutable** – old versions remain until compaction.
4. **Read operations** scan SSTables to find the latest version.

✅ **SSTables improve write speed and durability** by preventing in-place updates.

---

## **5️⃣ What is a Column Family in Cassandra?**
A **Column Family** is like a **table in RDBMS**, but with **flexible schema**.

### **Example:**
```cql
CREATE TABLE employees (
    department_id int,
    employee_id int PRIMARY KEY,
    first_name text,
    last_name text,
    email text
);
```
- **Each row** can have **different columns** (unlike RDBMS).
- **Column Family = Table + Dynamic Columns**.

✅ **Allows efficient storage & retrieval of semi-structured data**.

---

## **6️⃣ Explain CAP theorem. How is it related to Cassandra?**
**CAP Theorem** states that **a distributed database can guarantee only two out of three**:  

| CAP Property  | Explanation |
|--------------|-------------|
| **C** (Consistency) | Every node has the same data at the same time. |
| **A** (Availability) | System remains operational even if some nodes fail. |
| **P** (Partition Tolerance) | The system continues to function despite network failures. |

### **How Cassandra Handles CAP?**
✔ **Prioritizes AP (Availability + Partition Tolerance)**  
✔ **Supports Eventual Consistency**  
✔ **Allows tunable consistency (strong or weak consistency depending on the requirement).**

---

## **7️⃣ What is the purpose of using Cassandra Query Language (CQL)?**
Cassandra Query Language (CQL) is a **SQL-like language** used to interact with Cassandra.

### **Features of CQL:**
✔ Similar to SQL but **no Joins, Group By, or Foreign Keys**.  
✔ Supports **batch operations** for multiple queries.  
✔ **Schema definition** for tables, indexes, and materialized views.  

### **Example Queries:**
```cql
-- Creating a Table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    first_name text,
    last_name text,
    email text
);

-- Inserting Data
INSERT INTO users (user_id, first_name, last_name, email)
VALUES (uuid(), 'John', 'Doe', 'john.doe@example.com');

-- Selecting Data
SELECT * FROM users WHERE user_id = 123;
```

✅ **CQL provides a structured way to interact with Cassandra efficiently.**

---

## **8️⃣ What is Tunable Consistency in Cassandra?**
**Tunable Consistency** allows Cassandra to balance **Consistency vs. Availability**.

### **Types of Write Consistency Levels:**
| Consistency Level | Description |
|------------------|-------------|
| **ANY** | Write succeeds if **one node (even hinted handoff) stores it**. |
| **ONE** | Write succeeds if **one replica stores the data**. |
| **QUORUM** | Write succeeds if **(Replication Factor/2) + 1 replicas store it**. |
| **ALL** | Write succeeds only if **all replicas store it**. |

### **Example:**
```cql
INSERT INTO users (user_id, first_name) VALUES (123, 'Alice')
USING CONSISTENCY QUORUM;
```
✅ **Allows developers to choose between Strong or Eventual Consistency.**

---

## **9️⃣ What are some of the key features of Cassandra?**
✔ **Distributed Architecture** – No master-slave, all nodes are equal.  
✔ **High Availability** – Fault-tolerant, no single point of failure.  
✔ **Linear Scalability** – Add more nodes to increase capacity.  
✔ **Tunable Consistency** – Choose between strong and eventual consistency.  
✔ **Flexible Schema** – No fixed schema, supports dynamic columns.  
✔ **Automatic Replication** – Ensures data durability.  
✔ **Efficient Writes** – Uses Log-Structured Merge Trees (LSM) for fast inserts.  

✅ **Cassandra is ideal for real-time, high-volume applications**.

---

## **🔟 What is a Keyspace in Cassandra?**
A **Keyspace** is the **top-level namespace** in Cassandra, similar to a **database in RDBMS**.

### **Creating a Keyspace**
```cql
CREATE KEYSPACE my_keyspace
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};
```
### **Key Components:**
✔ **Replication Factor** – Determines how many copies of data exist.  
✔ **Replication Strategy** – Defines how data is distributed (`SimpleStrategy` or `NetworkTopologyStrategy`).  

✅ **Keyspaces allow efficient data partitioning & replication in Cassandra.**

---

### **✅ Summary of Questions 1-10**
1. **Cassandra is a scalable NoSQL database for distributed data storage.**  
2. **Tokens define how data is distributed across nodes.**  
3. **RDBMS vs. Cassandra: Schema-free, horizontally scalable.**  
4. **SSTables store immutable data on disk for fast writes.**  
5. **Column Families are flexible, schema-less tables.**  
6. **CAP Theorem: Cassandra prioritizes Availability & Partition Tolerance.**  
7. **CQL is a SQL-like query language for Cassandra.**  
8. **Tunable Consistency allows balancing performance vs. correctness.**  
9. **Cassandra has features like fault tolerance, high availability & scalability.**  
10. **Keyspace is like a database, defining replication and storage settings.**  


<br/>
<br/>

---

## **1️⃣1️⃣ What are the different types of Replication Strategies in Cassandra?**  
Replication in Cassandra **ensures fault tolerance and availability** by storing multiple copies of data across nodes.

### **Types of Replication Strategies**
| Replication Strategy | Description | Use Case |
|----------------------|-------------|-----------|
| **SimpleStrategy** | Replicates data **sequentially** to the next available nodes. Best for **single data center** setups. | Small clusters, single DC |
| **NetworkTopologyStrategy** | Distributes replicas **intelligently across multiple data centers** and racks for fault tolerance. | Large-scale production deployments |
| **OldNetworkTopologyStrategy** | Older version of **NetworkTopologyStrategy**, not commonly used. | Deprecated |

📌 **Example of Setting Replication Strategy:**
```cql
CREATE KEYSPACE my_keyspace 
WITH replication = { 'class': 'NetworkTopologyStrategy', 'DC1': 3, 'DC2': 2 };
```
✅ **NetworkTopologyStrategy is recommended for production deployments.**

---

## **1️⃣2️⃣ Explain Compaction in Cassandra.**  
**Compaction is the process of merging multiple SSTables into one** to optimize reads and reclaim storage space.

### **Types of Compaction Strategies**
| Strategy | Description | Best For |
|----------|-------------|----------|
| **Size-Tiered Compaction (STCS)** | Merges SSTables when a threshold number of similar-sized tables exist. | **Write-heavy workloads** |
| **Leveled Compaction (LCS)** | Organizes SSTables into levels to reduce read amplification. | **Read-heavy workloads** |
| **Time-Window Compaction (TWCS)** | Compacts SSTables based on time windows. | **Time-series data** |

📌 **Example of Setting Compaction Strategy:**
```cql
ALTER TABLE employees 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
✅ **Compaction improves query speed by reducing unnecessary SSTables.**

---

## **1️⃣3️⃣ What is the Bloom Filter in Cassandra?**  
A **Bloom filter** is a **probabilistic data structure** that helps **quickly check if a partition exists in an SSTable**.

### **How It Works**
- Each **partition key** is **hashed** and mapped into a **bit array**.
- If **any bit is 0**, the partition **does not exist** in the SSTable.
- If **all bits are 1**, the partition **might exist**, so Cassandra performs a full scan.

✅ **Reduces unnecessary disk reads, improving read performance.**

---

## **1️⃣4️⃣ What is Snitch in Cassandra?**  
A **Snitch** tells Cassandra **which nodes are in which racks & data centers** for better replica placement.

### **Types of Snitches**
| Snitch Type | Description | Best For |
|-------------|-------------|----------|
| **SimpleSnitch** | Ignores racks & DCs, distributes data evenly. | Single data center |
| **GossipingPropertyFileSnitch** | Automatically detects DC & rack info. | Multi-DC production setups |
| **Ec2Snitch** | Maps racks to AWS availability zones. | AWS cloud deployments |

📌 **Example of Configuring Snitch (`cassandra.yaml` file)**:
```yaml
endpoint_snitch: GossipingPropertyFileSnitch
```
✅ **Snitches help optimize network latency & fault tolerance.**

---

## **1️⃣5️⃣ What are Cassandra-Stress tools and their usage?**  
Cassandra provides the **`cassandra-stress`** tool for **performance testing & benchmarking**.

### **Common Commands**
| Command | Description |
|---------|-------------|
| `cassandra-stress write n=10000` | Writes **10,000 rows** to test write performance. |
| `cassandra-stress read n=5000` | Reads **5,000 rows** to measure read latency. |
| `cassandra-stress mixed ratio\(write=3, read=1\)` | Simulates **a mix of 3 writes for every 1 read**. |

✅ **`cassandra-stress` helps in performance tuning and capacity planning.**

---

## **1️⃣6️⃣ What is Gossip Protocol in Cassandra?**  
The **Gossip Protocol** is a **peer-to-peer communication mechanism** that allows nodes to exchange information **about the cluster’s health & topology**.

### **How It Works**
1️⃣ **Each node gossips with a few random nodes** every second.  
2️⃣ They exchange information about **alive & dead nodes**.  
3️⃣ This information spreads across the cluster **eventually**.

✅ **Gossip ensures cluster-wide communication without a single point of failure.**

---

## **1️⃣7️⃣ What is a Commit Log in Cassandra?**  
A **Commit Log** is a **write-ahead log** that ensures durability **before data is written to Memtable**.

### **How Writes Work in Cassandra**
1️⃣ **Data is written to the Commit Log (disk).**  
2️⃣ **Data is stored in Memtable (RAM).**  
3️⃣ **Once Memtable is full, it flushes to an SSTable.**  

✅ **Ensures data durability even if the system crashes.**

---

## **1️⃣8️⃣ What is Read Repair in Cassandra?**  
**Read Repair** is a mechanism used to **fix inconsistent data across replicas** during read operations.

### **How It Works**
1️⃣ A **read request** is sent to multiple replicas.  
2️⃣ If there is **data inconsistency**, Cassandra **reconciles using the latest timestamp**.  
3️⃣ **Updated data is written back to stale replicas** to repair them.

📌 **Example of Forcing Read Repair:**
```cql
SELECT * FROM employees USING CONSISTENCY QUORUM;
```
✅ **Ensures data consistency without affecting write performance.**

---

## **1️⃣9️⃣ How is data stored in Cassandra?**  
Cassandra stores data in a **Log-Structured Merge (LSM) Tree** format.

### **Data Storage Process**
1️⃣ **Write Request** → Written to **Commit Log** (disk).  
2️⃣ **Stored in Memtable** (RAM).  
3️⃣ **Flushed as an immutable SSTable** (disk) when Memtable is full.  
4️⃣ **Old SSTables are compacted periodically** to optimize storage.

✅ **LSM Trees improve write performance by avoiding random disk writes.**

---

## **2️⃣0️⃣ Explain the role of the Coordinator node in Cassandra.**  
A **Coordinator Node** is the **first node that receives a client request**.

### **Coordinator Responsibilities**
1️⃣ **Determines which nodes store the required data.**  
2️⃣ **Forwards the request** to replica nodes.  
3️⃣ **Ensures consistency based on the set Consistency Level (CL).**  
4️⃣ **Merges read responses & returns final data to the client.**

✅ **The Coordinator plays a key role in query routing and consistency management.**

---

## **📌 Summary of Questions 11-20**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **11** | Replication Strategies | `SimpleStrategy` (single DC), `NetworkTopologyStrategy` (multi-DC) |
| **12** | Compaction | **STCS, LCS, TWCS** optimize SSTables for better reads |
| **13** | Bloom Filter | Prevents unnecessary disk reads by quickly checking SSTables |
| **14** | Snitch | Defines **rack & data center awareness** for replica placement |
| **15** | Cassandra-Stress | Tool for **performance testing** of read/write queries |
| **16** | Gossip Protocol | **Nodes exchange state info** every second to maintain cluster health |
| **17** | Commit Log | Ensures **data durability before writing to Memtable** |
| **18** | Read Repair | Fixes **outdated replicas during reads** |
| **19** | Data Storage | **Commit Log → Memtable → SSTable → Compaction** |
| **20** | Coordinator Node | **Routes requests, ensures consistency, and merges responses** |

---

<br/>
<br/>


## **2️⃣1️⃣ What is Hinted Handoff in Cassandra?**  
**Hinted Handoff** is a mechanism that **temporarily stores writes for unavailable nodes** to ensure **eventual consistency**.

### **How It Works**
1️⃣ A write request **fails** because a replica node is **down**.  
2️⃣ The Coordinator node **stores a hint** about the write.  
3️⃣ When the failed node **comes back online**, the Coordinator **replays the hints** to update it.  

✅ **Ensures data durability even if a node goes temporarily offline.**  

📌 **Example of Configuring Hinted Handoff (`cassandra.yaml`):**
```yaml
hinted_handoff_enabled: true
max_hint_window_in_ms: 10800000  # 3 hours
```
❌ **Limitation:** If a node remains **down too long**, hints may be **discarded**.

---

## **2️⃣2️⃣ What is Apache Cassandra's write pattern?**  
Cassandra uses a **Log-Structured Merge (LSM) Tree** model for **efficient writes**.

### **Write Process**
1️⃣ **Client writes data** → Sent to **Coordinator node**.  
2️⃣ **Data is written to the Commit Log** (disk) for durability.  
3️⃣ **Data is also stored in Memtable** (RAM) for fast access.  
4️⃣ **Memtable flushes data to an SSTable** (disk) when full.  
5️⃣ **Compaction merges SSTables** to optimize storage.  

✅ **Cassandra writes are super fast because they avoid random disk writes!**  

---

## **2️⃣3️⃣ What is Lightweight Transaction (LWT) in Cassandra?**  
**LWT (CAS - Compare and Set)** allows **conditional updates** using the **Paxos protocol**.

### **Example**
```cql
INSERT INTO users (user_id, email) 
VALUES (1, 'john@example.com') 
IF NOT EXISTS;
```
📌 **Ensures** the row is inserted **only if it doesn’t already exist**.

### **How LWT Works**
✔ Uses **Paxos Consensus** to **ensure data consistency**.  
✔ Includes **four phases** (Prepare, Accept, Commit, Learn).  
❌ **Slower than normal writes** because of multiple phases.  

✅ **Best for:** Unique username/email constraints, financial transactions.

---

## **2️⃣4️⃣ What is a Composite Key in Cassandra?**  
A **Composite Key** consists of a **Partition Key + Clustering Columns**.

### **Example**
```cql
CREATE TABLE employees (
    department_id int,
    employee_id int,
    last_name text,
    PRIMARY KEY ((department_id), employee_id, last_name)
);
```
✔ **Partition Key**: `department_id` → **Distributes data across nodes**.  
✔ **Clustering Keys**: `employee_id, last_name` → **Sorts data inside the partition**.

✅ **Allows efficient searching and sorting within partitions.**

---

## **2️⃣5️⃣ Can you explain how Cassandra handles data modifications?**  
Cassandra **never updates in place**; it **appends new versions of data** instead.

### **Steps for Data Modification**
1️⃣ **Update/Delete request is written to Memtable.**  
2️⃣ A **timestamp is added** to track versions.  
3️⃣ **Old data remains in SSTables** (until compaction removes it).  
4️⃣ During **reads, the latest timestamp wins**.  

📌 **Deletes use "Tombstones"** instead of actually removing data.

✅ **Efficient but requires compaction to remove old data.**

---

## **2️⃣6️⃣ What are the different types of tombstones in Cassandra?**  
**Tombstones** are **markers** for deleted data. They exist **until compaction removes them**.

### **Types of Tombstones**
| Tombstone Type | Description |
|---------------|-------------|
| **Cell Tombstone** | Marks a **specific column** as deleted. |
| **Row Tombstone** | Marks an **entire row** as deleted. |
| **Partition Tombstone** | Marks a **full partition** as deleted. |
| **Range Tombstone** | Deletes **multiple rows in a range**. |

✅ **Deletes are only finalized when compaction removes tombstones!**

📌 **Example:**  
```cql
DELETE FROM employees WHERE department_id = 1;
```
✔ **A tombstone is created**, and the data is removed later.

---

## **2️⃣7️⃣ What is a Super Column in Cassandra?**  
A **Super Column** is a **deprecated feature** that allowed a **two-level column hierarchy**.

### **Structure:**
```
SuperColumnFamily
 ├── Row1
 │   ├── SuperColumn1
 │   │   ├── Column1: Value1
 │   │   ├── Column2: Value2
 │   ├── SuperColumn2
 │       ├── Column3: Value3
```
📌 **Why is it deprecated?**  
- Hard to manage.  
- Slower than **wide rows with clustering columns**.  
- **Replaced by nested tables and collections.**

✅ **Instead, use Tables with Clustering Keys.**

---

## **2️⃣8️⃣ Explain the role of Memtable in Cassandra.**  
A **Memtable** is an **in-memory structure** that **stores writes before flushing to SSTables**.

### **How Memtable Works**
1️⃣ **New data is written to Memtable** (fast, in-memory).  
2️⃣ **Also written to Commit Log** (for durability).  
3️⃣ **When Memtable reaches its limit, it flushes to SSTable**.  

✅ **Speeds up writes by reducing disk I/O.**  

📌 **Example: Setting Memtable Threshold (`cassandra.yaml`)**
```yaml
memtable_flush_period_in_ms: 60000  # Flush every 60 seconds
```
✔ **If a node crashes before flushing, Commit Log recovers lost writes!**

---

## **2️⃣9️⃣ What is meant by 'consistent hashing' in Cassandra?**  
**Consistent Hashing** distributes data **evenly across nodes** without major rebalancing.

### **How It Works**
✔ Each **node gets a token range** (e.g., `Node 1: 0-100`, `Node 2: 101-200`).  
✔ When **a new node is added**, **only part of the data** moves (not all).  
✔ **Prevents hotspots** by distributing writes across the cluster.

✅ **Ensures load balancing and fault tolerance.**

📌 **Example of Token Ranges**
```
Node A (Token 0 - 50)
Node B (Token 51 - 100)
Node C (Token 101 - 150)
```
✔ **If Node B fails, its data is automatically handled by neighbors.**

---

## **3️⃣0️⃣ What is Time to Live (TTL) in Cassandra?**  
TTL (**Time To Live**) **automatically deletes data after a set time**.

### **Example: Setting TTL**
```cql
INSERT INTO employees (employee_id, first_name, email) 
VALUES (5001, 'John', 'john@example.com') 
USING TTL 86400;
```
📌 **This row will expire after 1 day (86,400 seconds).**

### **How TTL Works**
✔ **TTL columns are deleted automatically** when time expires.  
✔ **Stored as tombstones** and removed during compaction.  

✅ **Great for expiring temporary data (e.g., session tokens, logs).**

---

## **📌 Summary of Questions 21-30**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **21** | Hinted Handoff | Stores failed writes & replays when nodes recover. |
| **22** | Write Pattern | Uses **Commit Log, Memtable, SSTable** for efficiency. |
| **23** | Lightweight Transactions | Ensures atomic updates using **Paxos protocol**. |
| **24** | Composite Key | Uses **Partition Key + Clustering Keys** for sorting. |
| **25** | Data Modifications | Writes **never update in place**, old data remains until compaction. |
| **26** | Tombstones | Mark deleted data **until compaction removes it**. |
| **27** | Super Column | **Deprecated**, replaced by tables with clustering keys. |
| **28** | Memtable | Stores recent writes **in memory** before flushing to disk. |
| **29** | Consistent Hashing | **Evenly distributes data across nodes** to prevent hotspots. |
| **30** | TTL | **Automatically expires data** after a set time. |

---


<br/>
<br/>


## **3️⃣1️⃣ What are seeds in Cassandra?**
**Seeds** are **initial contact points** that help new nodes **discover other nodes in the cluster**.

### **Role of Seed Nodes:**
✔ **Bootstrapping New Nodes:** When a new node joins, it contacts seed nodes to get cluster information.  
✔ **Facilitating Gossip Communication:** Seed nodes help propagate cluster state efficiently.  
✔ **Not Special Nodes:** Seed nodes are normal Cassandra nodes, but they **do not** have extra privileges.  

📌 **Example Configuration (`cassandra.yaml`):**
```yaml
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
      - seeds: "192.168.1.1,192.168.1.2"
```
✅ **Clusters should have at least two seeds per data center for reliability.**

---

## **3️⃣2️⃣ What is the role of the Partitioner in Cassandra?**
A **Partitioner** decides **how data is distributed** across nodes.

### **Types of Partitioners:**
| Partitioner | Description | Default? |
|-------------|-------------|---------|
| **Murmur3Partitioner** | Uses **Murmur3 hash function** for even distribution. | ✅ Default |
| **RandomPartitioner** | Uses a **random hash function** (deprecated). | ❌ No |
| **ByteOrderedPartitioner** | **Stores data in sorted order**, useful for range queries. | ❌ No |

📌 **Example: Setting Partitioner in (`cassandra.yaml`):**
```yaml
partitioner: org.apache.cassandra.dht.Murmur3Partitioner
```
✅ **Partitioners ensure balanced data distribution and prevent hotspots.**

---

## **3️⃣3️⃣ What is the difference between deleting and expiring in Cassandra?**
| **Aspect** | **Deleting** | **Expiring (TTL)** |
|-----------|------------|------------|
| **Trigger** | Explicit `DELETE` query | `TTL` (Time-To-Live) on insert |
| **Data Removal** | Marked with **tombstones** | Auto-expires after TTL |
| **Cleanup** | Removed during **compaction** | Removed automatically when TTL expires |

📌 **Example of Deleting:**
```cql
DELETE FROM employees WHERE employee_id = 5001;
```
📌 **Example of TTL (Expiring Data):**
```cql
INSERT INTO employees (employee_id, email) 
VALUES (5001, 'john@example.com') USING TTL 86400;
```
✅ **TTL is useful for caching and session expiration.**

---

## **3️⃣4️⃣ What are Repair and Anti-Entropy in Cassandra?**
✔ **Repair** ensures data consistency across replicas.  
✔ **Anti-Entropy** detects and fixes inconsistencies.

### **Types of Repairs**
| Type | Description |
|------|-------------|
| **Full Repair** | Syncs all partitions, expensive but ensures correctness. |
| **Incremental Repair** | Repairs **only new updates**, reducing overhead. |
| **Read Repair** | Fixes inconsistencies **during reads**. |

📌 **Running a Manual Repair Command:**
```bash
nodetool repair
```
✅ **Regular repairs prevent data inconsistencies across replicas.**

---

## **3️⃣5️⃣ What is a Materialized View in Cassandra?**
A **Materialized View (MV)** is a **precomputed query result** that maintains synchronization with the base table.

### **Use Case:**
✔ Efficient for **alternative query patterns** without creating multiple copies of data.  

📌 **Example: Creating an MV for fast lookups by email**
```cql
CREATE MATERIALIZED VIEW employees_by_email AS
SELECT employee_id, first_name, last_name, email
FROM employees
WHERE email IS NOT NULL
PRIMARY KEY (email, employee_id);
```
✅ **MVs improve read performance but have write overhead.**

---

## **3️⃣6️⃣ What is the role of CQLSH in Cassandra?**
**CQLSH (Cassandra Query Language Shell)** is a **command-line interface** for executing CQL queries.

### **Common CQLSH Commands**
| Command | Description |
|---------|-------------|
| `cqlsh` | Open the CQL shell |
| `DESCRIBE KEYSPACES;` | List all keyspaces |
| `USE keyspace_name;` | Select a keyspace |
| `SELECT * FROM employees;` | Query a table |
| `EXIT;` | Quit CQLSH |

✅ **CQLSH is the primary tool for managing Cassandra clusters interactively.**

---

## **3️⃣7️⃣ Explain the difference between a node, a cluster, and a data center in Cassandra.**
| **Component** | **Description** |
|-------------|----------------|
| **Node** | A single **Cassandra instance** storing data. |
| **Cluster** | A group of nodes working together. |
| **Data Center** | A collection of nodes **grouped for replication & latency optimization**. |

📌 **Example Setup**
```
Cluster: E-commerce App
├── Data Center 1 (North America)
│   ├── Node 1
│   ├── Node 2
│   ├── Node 3
├── Data Center 2 (Europe)
│   ├── Node 4
│   ├── Node 5
│   ├── Node 6
```
✅ **Data centers optimize performance for geographically distributed users.**

---

## **3️⃣8️⃣ What is a Counter Column in Cassandra?**
A **Counter Column** stores a **numeric value** that can be **incremented or decremented**.

📌 **Example: Keeping track of product views**
```cql
CREATE TABLE product_views (
    product_id int PRIMARY KEY,
    view_count counter
);
```
📌 **Incrementing the Counter:**
```cql
UPDATE product_views SET view_count = view_count + 1 WHERE product_id = 101;
```
✅ **Useful for analytics (e.g., page views, likes, shares).**

---

## **3️⃣9️⃣ Explain what "write heavy" and "read heavy" mean in the context of Cassandra.**
✔ **Write Heavy**: More writes than reads (e.g., IoT logs, analytics).  
✔ **Read Heavy**: More reads than writes (e.g., user profiles, recommendation systems).  

📌 **Tuning for Write-Heavy Workloads**
- Use **Size-Tiered Compaction Strategy (STCS)**.  
- Set **higher write consistency levels** (`QUORUM`).  

📌 **Tuning for Read-Heavy Workloads**
- Use **Leveled Compaction Strategy (LCS)**.  
- Enable **Materialized Views** for optimized queries.  

✅ **Cassandra is best suited for write-heavy workloads!**

---

## **4️⃣0️⃣ What are dynamic columns in Cassandra?**
**Dynamic Columns** allow **storing variable columns per row**, making Cassandra **schema-flexible**.

📌 **Example: Users with different metadata fields**
```cql
CREATE TABLE user_data (
    user_id int PRIMARY KEY,
    metadata map<text, text>
);
```
📌 **Inserting Dynamic Columns:**
```cql
INSERT INTO user_data (user_id, metadata) 
VALUES (1, {'age': '25', 'location': 'New York'});
```
✅ **Allows flexibility in storing unstructured data.**

---

## **📌 Summary of Questions 31-40**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **31** | Seeds | Help new nodes discover the cluster. |
| **32** | Partitioner | Distributes data across nodes using hashing. |
| **33** | Delete vs TTL | `DELETE` marks tombstones, `TTL` auto-expires data. |
| **34** | Repair & Anti-Entropy | Ensures data consistency across replicas. |
| **35** | Materialized View | Precomputed table for optimized reads. |
| **36** | CQLSH | CLI tool for running Cassandra queries. |
| **37** | Node vs Cluster vs DC | Node = single instance, Cluster = group of nodes, DC = logical grouping. |
| **38** | Counter Column | Stores incrementing values (e.g., likes, views). |
| **39** | Write-Heavy vs Read-Heavy | **Cassandra is optimized for write-heavy workloads**. |
| **40** | Dynamic Columns | Allow flexible schema with **Map, Set, and List**. |

---


<br/>
<br/>

## **4️⃣1️⃣ What is a Batch in Cassandra?**  
A **Batch Statement** in Cassandra **groups multiple write queries** into a single transaction, improving efficiency.

### **Types of Batch Queries**
| Batch Type | Description |
|------------|-------------|
| **Logged Batch** | Uses a **batch log** to ensure atomic execution (default). |
| **Unlogged Batch** | Executes all statements independently, with no guarantee of atomicity. |

📌 **Example: Inserting multiple records using a Batch Statement**
```cql
BEGIN BATCH
INSERT INTO employees (employee_id, first_name, last_name, department_id) VALUES (5001, 'Alice', 'Brown', 1);
INSERT INTO employees (employee_id, first_name, last_name, department_id) VALUES (5002, 'Bob', 'Smith', 2);
APPLY BATCH;
```
✅ **Best for reducing network overhead when inserting/updating multiple rows.**  
❌ **Avoid using Batches for different partitions, as it can lead to performance issues.**

---

## **4️⃣2️⃣ What is Rack Awareness in Cassandra?**  
**Rack Awareness** ensures **replicas are spread across different racks** in a data center, increasing fault tolerance.

### **Why is Rack Awareness Important?**
✔ **Prevents data loss** if an entire rack fails.  
✔ **Reduces network latency** by ensuring efficient data distribution.  

📌 **Example: Configuring Rack Awareness (`cassandra-rackdc.properties`)**
```yaml
dc=datacenter1
rack=rack1
```
✅ **Ensures better fault tolerance and optimized data placement.**

---

## **4️⃣3️⃣ What is a Secondary Index in Cassandra?**  
A **Secondary Index** allows **queries on non-primary key columns**.

📌 **Example: Creating an Index on `email` column**
```cql
CREATE INDEX ON employees(email);
```
📌 **Query Using Index:**
```cql
SELECT * FROM employees WHERE email = 'alice@example.com';
```
❌ **Limitations of Secondary Indexes**  
- **Not efficient for high-cardinality columns** (e.g., timestamps, UUIDs).  
- **Slower than partition key queries**.

✅ **Use when filtering by non-primary key columns but prefer Materialized Views if possible.**

---

## **4️⃣4️⃣ What is the role of the `cassandra.yaml` file?**  
The `cassandra.yaml` file is **the main configuration file** for a Cassandra cluster.

### **Key Parameters in `cassandra.yaml`**
| Parameter | Description |
|-----------|-------------|
| `cluster_name` | Defines the name of the Cassandra cluster. |
| `num_tokens` | Number of virtual nodes (vnodes) per node. |
| `seed_provider` | List of seed nodes for cluster discovery. |
| `endpoint_snitch` | Defines topology strategy (e.g., `GossipingPropertyFileSnitch`). |

📌 **Example: Setting Cluster Name**
```yaml
cluster_name: MyCassandraCluster
```
✅ **Modifying `cassandra.yaml` allows tuning Cassandra’s performance and topology.**

---

## **4️⃣5️⃣ What are Collection Data Types in Cassandra?**  
Cassandra supports **Collection Data Types** for storing **multiple values in a single column**.

### **Types of Collections**
| Type | Description | Example |
|------|-------------|---------|
| **List** | Ordered collection | `['Alice', 'Bob', 'Charlie']` |
| **Set** | Unique, unordered collection | `{'apple', 'banana', 'grape'}` |
| **Map** | Key-value pairs | `{'name': 'Alice', 'age': 30}` |

📌 **Example: Creating a Table with Collections**
```cql
CREATE TABLE users (
    user_id int PRIMARY KEY,
    emails set<text>,
    phone_numbers list<text>,
    metadata map<text, text>
);
```
✅ **Collections allow flexible, schema-free storage.**  
❌ **Can become inefficient if they grow too large.**

---

## **4️⃣6️⃣ What is Paxos in Cassandra?**  
**Paxos** is a **consensus protocol** used in **Lightweight Transactions (LWT)** for strong consistency.

### **Paxos Phases in Cassandra**
| Phase | Description |
|-------|-------------|
| **Prepare** | Propose a value and check if another value is already committed. |
| **Accept** | If no previous value exists, the proposal is accepted. |
| **Commit** | The transaction is finalized and applied. |
| **Learn** | Other nodes learn about the committed value. |

📌 **Example: Using Paxos for Unique Username Enforcement**
```cql
INSERT INTO users (user_id, username) 
VALUES (1, 'alice123') IF NOT EXISTS;
```
✅ **Ensures ACID-like transactions but has higher latency.**  

---

## **4️⃣7️⃣ What is a Wide Row in Cassandra?**  
A **Wide Row** refers to **a single partition containing many rows**, making reads efficient.

### **Example: Time-Series Data (Sensor Readings)**
```cql
CREATE TABLE sensor_data (
    device_id int,
    timestamp timestamp,
    temperature float,
    PRIMARY KEY (device_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```
✔ **Partition Key (`device_id`)** keeps all data for a device in one node.  
✔ **Clustering Key (`timestamp`)** allows fast retrieval of recent data.  

✅ **Ideal for time-series, logs, and analytics use cases.**  
❌ **Can cause performance issues if a partition grows too large.**

---

## **4️⃣8️⃣ Explain the difference between `QUORUM` and `LOCAL_QUORUM` consistency levels.**
| **Consistency Level** | **Description** | **Best Use Case** |
|----------------------|----------------|------------------|
| **QUORUM** | Majority of replicas across **all data centers** must respond. | Ensures **stronger consistency** but **higher latency**. |
| **LOCAL_QUORUM** | Majority of replicas in **local data center** must respond. | Optimized for **low-latency queries in multi-DC setups**. |

📌 **Example of `QUORUM`:**
```cql
SELECT * FROM employees USING CONSISTENCY QUORUM;
```
✅ **Use `LOCAL_QUORUM` when minimizing cross-DC latency.**

---

## **4️⃣9️⃣ What are the limitations of using Secondary Indexes in Cassandra?**  
**Secondary Indexes** are **not efficient** in Cassandra due to its distributed nature.

### **Limitations**
❌ **Not efficient for high-cardinality columns** (e.g., timestamps, UUIDs).  
❌ **Increases read latency** because queries must scan multiple nodes.  
❌ **Indexes are stored per node**, not globally.  

📌 **Better Alternatives:**  
✔ **Use Materialized Views for better indexing.**  
✔ **Denormalize data using query-specific tables.**  

✅ **Only use Secondary Indexes for low-cardinality columns (e.g., "status = 'Active'").**

---

## **5️⃣0️⃣ What is Thrift in Cassandra?**  
**Thrift** was the original API for interacting with Cassandra, but **it is now deprecated in favor of CQL**.

### **Why was Thrift Replaced?**
| Feature | Thrift | CQL |
|---------|--------|-----|
| **Query Language** | Low-level API | SQL-like syntax |
| **Schema Flexibility** | Complex | Easier to manage |
| **Performance** | Slower | Optimized for modern workloads |

📌 **Thrift Commands (Deprecated)**  
```thrift
get users['user_id']['name'];
```
✅ **CQL is now the standard interface for Cassandra queries.**

---

## **📌 Summary of Questions 41-50**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **41** | Batch Queries | Group multiple writes for efficiency. |
| **42** | Rack Awareness | Spreads replicas across racks for fault tolerance. |
| **43** | Secondary Index | Allows queries on non-PK columns but is inefficient. |
| **44** | `cassandra.yaml` | Main config file for cluster settings. |
| **45** | Collection Types | Supports `List`, `Set`, and `Map` for flexible storage. |
| **46** | Paxos | Ensures consistency in Lightweight Transactions (LWT). |
| **47** | Wide Row | Stores many rows in a single partition for fast reads. |
| **48** | `QUORUM` vs `LOCAL_QUORUM` | `LOCAL_QUORUM` reduces cross-DC latency. |
| **49** | Secondary Index Limitations | Inefficient for high-cardinality data. |
| **50** | Thrift | Deprecated API, replaced by CQL. |

---


<br/>
<br/>


## **5️⃣1️⃣ Explain the differences between `ONE`, `TWO`, `THREE`, and `ALL` consistency levels in Cassandra.**

Consistency levels in Cassandra define **how many replica nodes** must acknowledge a read/write operation before it is considered successful.

| **Consistency Level** | **Description** | **Best Use Case** |
|----------------------|----------------|------------------|
| **ONE** | The request must be acknowledged by **at least one replica**. | Fastest writes, but may return stale data. |
| **TWO** | Requires **two replicas** to acknowledge the write/read. | Moderate consistency, slightly more reliable than `ONE`. |
| **THREE** | Requires **three replicas** to acknowledge the operation. | Higher consistency but slightly slower. |
| **ALL** | All replicas in the cluster must acknowledge the request. | Strongest consistency, but high latency and low availability. |

📌 **Example: Setting Write Consistency Level**
```cql
INSERT INTO employees (employee_id, first_name) VALUES (5001, 'Alice') 
USING CONSISTENCY QUORUM;
```
✅ **Higher consistency levels ensure up-to-date data but can reduce availability.**

---

## **5️⃣2️⃣ What are Prepared Statements in Cassandra and why would you use them?**

A **Prepared Statement** is a **precompiled CQL query** that improves performance by reducing parsing overhead.

### **Why Use Prepared Statements?**
✔ **Better Performance:** Query parsing happens **only once**, then reused.  
✔ **Prevents SQL Injection:** Queries are **parameterized**, preventing malicious input.  

📌 **Example: Using a Prepared Statement in Python**
```python
from cassandra.cluster import Cluster

cluster = Cluster(['127.0.0.1'])
session = cluster.connect('my_keyspace')

prepared_stmt = session.prepare("INSERT INTO users (user_id, name) VALUES (?, ?)")
session.execute(prepared_stmt, (123, 'Alice'))
```
✅ **Prepared statements improve execution speed and security.**

---

## **5️⃣3️⃣ What is 'Tombstone Garbage Collection Grace Seconds' in Cassandra?**

A **Tombstone** is a marker for deleted data. **Garbage Collection Grace Seconds** is the **time Cassandra waits before permanently removing tombstones**.

### **Why is this important?**
✔ Ensures **all replicas receive the delete request** before removing data.  
✔ Prevents **deleted data from reappearing** due to inconsistent replication.  

📌 **Configuring Tombstone GC Grace Period (`cassandra.yaml`):**
```yaml
gc_grace_seconds: 864000  # (10 days)
```
✅ **Default value is 10 days, but can be adjusted based on replication delays.**

---

## **5️⃣4️⃣ What is a Super Column Family in Cassandra?**

A **Super Column Family** was an **old schema design** that contained **nested columns within columns**.

### **Example (Deprecated Design)**
```
Users (Super Column Family)
├── user_id: 101
│   ├── Super Column: Address
│   │   ├── Street: "123 Main St"
│   │   ├── City: "New York"
```
📌 **Why is it deprecated?**
❌ **Complex queries**.  
❌ **Inefficient storage**.  
❌ **Replaced by tables with clustering columns**.

✅ **Instead, use a normal table with a composite primary key.**

---

## **5️⃣5️⃣ How does Cassandra handle concurrent writes?**

Cassandra handles **concurrent writes** efficiently because of its **append-only architecture**.

### **Why is it Fast?**
✔ **No Locking:** Writes happen in **Memtable** first, reducing contention.  
✔ **Timestamps Resolve Conflicts:** The **latest timestamp wins** if data conflicts.  
✔ **Multiple Nodes Accept Writes:** No **single master bottleneck**.  

📌 **Example: Concurrent Write Process**
1. **Two clients write to the same row.**
2. **Cassandra stores both versions with timestamps.**
3. **The most recent timestamp is returned on read.**

✅ **Cassandra is optimized for high-speed, concurrent writes.**

---

## **5️⃣6️⃣ What is the purpose of the System Keyspace in Cassandra?**

The **System Keyspace** stores **metadata** about the Cassandra cluster.

### **Important Tables in the System Keyspace**
| Table | Description |
|-------|-------------|
| `system.local` | Information about the current node. |
| `system.peers` | Info about other nodes in the cluster. |
| `system.schema_keyspaces` | Stores details about keyspaces. |

📌 **Example: Querying the System Keyspace**
```cql
SELECT * FROM system.local;
```
✅ **Useful for monitoring cluster health and topology.**

---

## **5️⃣7️⃣ What are some use cases where you would NOT want to use Cassandra?**

| **Use Case** | **Why Not Use Cassandra?** |
|-------------|-------------------------|
| **Strict ACID Transactions** | Cassandra lacks **full ACID compliance** (use PostgreSQL/MySQL instead). |
| **Frequent Joins & Aggregations** | No support for **Joins or Group By** (use PostgreSQL). |
| **Small Data Sets** | Overhead of replication isn’t needed for small applications. |
| **Graph Data Relationships** | Cassandra isn’t optimized for **complex graph queries** (use Neo4j). |

✅ **Cassandra is best for large-scale, high-availability applications.**

---

## **5️⃣8️⃣ What is a Column Family Store in Cassandra?**

A **Column Family Store** is a **storage engine** that organizes data into **wide-column tables**.

### **How It Works**
✔ **Rows can have different columns** (unlike relational DBs).  
✔ **Stored as SSTables on disk**.  
✔ **Uses **Memtables** for fast writes.  

📌 **Example Table Structure**
```cql
CREATE TABLE users (
    user_id int PRIMARY KEY,
    name text,
    email text
);
```
✅ **Column Families allow flexible schema evolution.**

---

## **5️⃣9️⃣ How can you secure your Cassandra deployment?**

| **Security Measure** | **Description** |
|----------------------|----------------|
| **Authentication & Authorization** | Use **role-based access control (RBAC)**. |
| **Encryption** | Enable **SSL/TLS for data in transit**. |
| **Firewall & Network Isolation** | Restrict **node access** using firewalls. |
| **Audit Logging** | Track **who accessed what data**. |

📌 **Example: Enabling Authentication (`cassandra.yaml`)**
```yaml
authenticator: PasswordAuthenticator
authorizer: CassandraAuthorizer
```
✅ **Proper security measures protect against unauthorized access.**

---

## **6️⃣0️⃣ What happens when a Cassandra node goes down during a write operation?**

✔ **Writes are still accepted** because Cassandra is **peer-to-peer**.  
✔ **Hinted Handoff stores missed writes** until the node recovers.  
✔ **Gossip detects the failure** and updates the cluster state.  
✔ **Read Repair and Anti-Entropy repair inconsistencies** later.

📌 **Example: Handling Node Failure**
1. **Client writes data** → One replica is **down**.  
2. **Coordinator stores hints** → Once the node is back, **hints are replayed**.  
3. **If node remains down too long**, **repair must be run manually**.

✅ **Cassandra ensures availability, even if nodes fail.**

---

## **📌 Summary of Questions 51-60**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **51** | Consistency Levels | `ONE`, `TWO`, `THREE`, `ALL` balance speed vs. correctness. |
| **52** | Prepared Statements | Improve performance and prevent SQL injection. |
| **53** | Tombstone GC Grace | Delays deletion to ensure all replicas sync. |
| **54** | Super Column Family | Deprecated, replaced by clustering columns. |
| **55** | Concurrent Writes | Timestamp-based conflict resolution. |
| **56** | System Keyspace | Stores metadata about the Cassandra cluster. |
| **57** | When NOT to Use Cassandra | ACID transactions, small datasets, graph data. |
| **58** | Column Family Store | Organizes data into wide-column tables. |
| **59** | Security Best Practices | **Authentication, encryption, firewalls, logging**. |
| **60** | Node Failure Handling | **Hinted Handoff + Repair ensures availability**. |

---


<br/>
<br/>

## **6️⃣1️⃣ How does Cassandra handle conflicts during replication?**  
Cassandra follows an **eventual consistency model**, meaning **conflicts can arise** when multiple replicas receive different versions of the same data.

### **Conflict Resolution Mechanisms**
✔ **Timestamp-based Resolution** (Latest Write Wins)  
✔ **Read Repair** (Fixes inconsistencies during reads)  
✔ **Hinted Handoff** (Stores missed writes for failed nodes)  
✔ **Anti-Entropy Repair** (Runs periodically to sync data)

📌 **Example: Two conflicting writes**  
1️⃣ **Client A writes `age = 25` (timestamp: 1000)**  
2️⃣ **Client B writes `age = 30` (timestamp: 1005)**  
3️⃣ **Cassandra picks the latest timestamp (`age = 30`)**  

✅ **Ensures high availability but may result in lost updates if timestamps aren't managed properly.**  

---

## **6️⃣2️⃣ What is the purpose of Cassandra's Read Repair mechanism?**  
Read Repair **ensures consistency across replicas** by fixing outdated copies of data **during reads**.

### **How Read Repair Works**
1️⃣ Client reads data from multiple replicas.  
2️⃣ If replicas **have different values**, Cassandra **chooses the latest version**.  
3️⃣ The outdated replicas are **updated with the latest value**.  

📌 **Example:**  
- **Replica 1:** `balance = 100`  
- **Replica 2:** `balance = 120` (latest update)  
- Read Repair ensures **Replica 1 gets updated to 120**.  

✅ **Reduces data inconsistencies without manual intervention.**

---

## **6️⃣3️⃣ How does the Gossip Protocol work in Cassandra?**  
The **Gossip Protocol** is a **peer-to-peer communication mechanism** that keeps nodes updated about the cluster's state.

### **How It Works**
✔ Each node **sends state updates** to **random peers** every second.  
✔ Nodes **exchange information** about other nodes they’ve gossiped with.  
✔ **If a node fails, others detect it** and mark it as "DOWN."  

📌 **Example: Node A gossips to Node B about Node C**
```
Node A → Node B: "Hey, Node C is alive."
Node B → Node D: "I heard Node C is alive from A."
```
✅ **Ensures all nodes eventually receive cluster-wide updates.**

---

## **6️⃣4️⃣ What's the difference between Leveled Compaction and Size-Tiered Compaction?**  
Cassandra **compacts SSTables** periodically to optimize reads and reduce storage usage.

| **Compaction Strategy** | **Description** | **Best Use Case** |
|----------------------|----------------|------------------|
| **Size-Tiered Compaction (STCS)** | Merges **similar-sized SSTables** when a threshold is reached. | Best for **write-heavy** workloads. |
| **Leveled Compaction (LCS)** | Organizes SSTables into **non-overlapping levels**, improving read efficiency. | Best for **read-heavy** workloads. |

📌 **Example of Setting Compaction Strategy**
```cql
ALTER TABLE users 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
✅ **LCS optimizes reads, while STCS is better for frequent writes.**

---

## **6️⃣5️⃣ What is a Bloom Filter and how does it work in Cassandra?**  
A **Bloom Filter** is a **probabilistic data structure** used to **check if a partition exists in an SSTable** **before doing a full scan**.

### **How Bloom Filters Work**
✔ Cassandra **hashes partition keys** into a bit array.  
✔ If a key **isn't in the bit array**, **the partition doesn't exist** in that SSTable.  
✔ If the key **might exist**, Cassandra performs **an actual lookup**.

📌 **Example**
- Looking for `user_id = 1234`
- **Bloom filter says "Maybe Exists"** → Perform disk lookup  
- **Bloom filter says "Does Not Exist"** → Avoids unnecessary I/O  

✅ **Speeds up read performance by reducing disk scans.**

---

## **6️⃣6️⃣ How does Cassandra ensure Durability?**  
Durability means **no data loss**, even if a node crashes.

### **How Cassandra Ensures Durability**
✔ **Writes are first stored in the Commit Log** (disk-based).  
✔ **Data is written to Memtable** (RAM) for fast access.  
✔ **Memtable flushes data to SSTables** periodically.  
✔ **Hinted Handoff and Replication** ensure data isn't lost during failures.  

📌 **Example: Configuring Commit Log Durability (`cassandra.yaml`)**
```yaml
commitlog_sync: batch
commitlog_sync_period_in_ms: 1000  # Sync every second
```
✅ **Ensures no data is lost, even during a crash.**

---

## **6️⃣7️⃣ What is the purpose of the Commit Log in Cassandra?**  
A **Commit Log** is a **write-ahead log** that guarantees **data durability before it’s written to Memtable**.

### **How It Works**
✔ **Step 1:** Write goes to **Commit Log (disk)**.  
✔ **Step 2:** Data is stored in **Memtable (RAM)**.  
✔ **Step 3:** When Memtable is full, data is **flushed to an SSTable**.  
✔ **Step 4:** After a successful flush, the **Commit Log entry is deleted**.

📌 **Example: Setting Commit Log Parameters (`cassandra.yaml`)**
```yaml
commitlog_segment_size_in_mb: 64
```
✅ **Ensures data is never lost, even if the node crashes.**

---

## **6️⃣8️⃣ How can you model time-series data in Cassandra?**  
Time-series data (e.g., sensor logs, stock prices) require **efficient storage & retrieval**.

### **Best Practices for Time-Series Data**
✔ **Use a composite primary key** (`device_id, timestamp`).  
✔ **Use clustering order** (`DESC`) for faster retrieval.  
✔ **Set TTL** for automatic data expiration.  

📌 **Example: Storing IoT Sensor Readings**
```cql
CREATE TABLE sensor_data (
    device_id int,
    timestamp timestamp,
    temperature float,
    PRIMARY KEY (device_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```
✅ **Ensures fast retrieval of the latest sensor readings.**

---

## **6️⃣9️⃣ Explain Lightweight Transactions (LWT) in Cassandra.**  
LWTs use the **Paxos consensus protocol** to ensure **atomic operations** in Cassandra.

### **LWT Phases**
1️⃣ **Prepare**: Propose a transaction.  
2️⃣ **Accept**: Confirm agreement on the proposed change.  
3️⃣ **Commit**: Apply the change if consensus is reached.  
4️⃣ **Learn**: Broadcast the result to replicas.  

📌 **Example: Preventing Duplicate Usernames**
```cql
INSERT INTO users (user_id, username) 
VALUES (1, 'alice123') IF NOT EXISTS;
```
✅ **Ensures uniqueness but is slower than normal writes.**

---

## **7️⃣0️⃣ How does Cassandra handle data compression?**  
Cassandra uses **compression to reduce storage space and improve read performance**.

### **Available Compression Algorithms**
| Compression Algorithm | Description |
|----------------------|-------------|
| **LZ4** | High speed, low compression ratio. |
| **Snappy** | Balanced speed & compression. |
| **Deflate** | High compression, but slower. |

📌 **Example: Enabling Compression on a Table**
```cql
ALTER TABLE users 
WITH compression = { 'class': 'LZ4Compressor' };
```
✅ **Compression reduces disk usage and speeds up read performance.**

---

## **📌 Summary of Questions 61-70**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **61** | Conflict Resolution | **Latest timestamp wins**, Read Repair, Anti-Entropy Repair. |
| **62** | Read Repair | Fixes inconsistencies **during reads**. |
| **63** | Gossip Protocol | Nodes exchange status updates every second. |
| **64** | Leveled vs. Size-Tiered Compaction | LCS = better reads, STCS = better writes. |
| **65** | Bloom Filter | **Prevents unnecessary disk reads**. |
| **66** | Durability | Commit Log + Replication ensure no data loss. |
| **67** | Commit Log | Guarantees durability before Memtable writes. |
| **68** | Time-Series Data | **Partition key + Clustering key (timestamp DESC)**. |
| **69** | LWT | Ensures **atomic updates** using Paxos. |
| **70** | Compression | **LZ4, Snappy, Deflate** improve storage efficiency. |

---


<br/>
<br/>


## **7️⃣1️⃣ Why does Cassandra not support joins?**  
Cassandra **does not support joins** because it prioritizes **high availability, scalability, and distributed architecture**.

### **Why Joins Are Inefficient in Cassandra**
✔ **Joins require scanning multiple partitions** → Expensive in a distributed system.  
✔ **Increases network traffic** → Slows down performance.  
✔ **Not optimized for distributed storage** → Query execution across nodes is costly.

### **Alternatives to Joins in Cassandra**
✔ **Denormalization** → Store related data together for fast lookups.  
✔ **Materialized Views** → Precompute frequently queried relationships.  
✔ **Secondary Indexes (use with caution)** → Index non-primary key columns.

📌 **Example: Denormalizing Instead of Using Joins**
```cql
CREATE TABLE orders_by_customer (
    customer_id int,
    order_id int,
    order_date timestamp,
    total_amount float,
    PRIMARY KEY (customer_id, order_id)
);
```
✅ **Joins are replaced by query-optimized table structures.**

---

## **7️⃣2️⃣ What is Eventual Consistency in Cassandra?**  
**Eventual Consistency** means **all replicas will have the same data eventually**, even if some updates are delayed.

### **How It Works**
✔ Writes are **asynchronously replicated** across nodes.  
✔ Reads **may return stale data** until updates are propagated.  
✔ **Read Repair and Anti-Entropy Repair** help synchronize data over time.

📌 **Example:**
1. **Client A updates `balance = 100` on Node 1**  
2. **Client B reads from Node 2 (which still has `balance = 90`)**  
3. **Later, Read Repair updates Node 2 to `balance = 100`**

✅ **Ensures high availability but may allow temporary inconsistencies.**

---

## **7️⃣3️⃣ What is the concept of 'Tunable Consistency' in Cassandra?**  
**Tunable Consistency** allows adjusting the **balance between consistency and availability** per query.

### **Write Consistency Levels**
✔ **ONE, TWO, THREE** → Acknowledgment from **1-3 nodes**.  
✔ **QUORUM** → Majority of replicas.  
✔ **ALL** → All replicas must confirm.

### **Read Consistency Levels**
✔ **ONE** → Reads from **one replica** (fast, but may return stale data).  
✔ **QUORUM** → Reads from **a majority of replicas**.  
✔ **ALL** → Reads from **all replicas** (slow but strongest consistency).

📌 **Example: Setting Tunable Consistency for Reads**
```cql
SELECT * FROM users USING CONSISTENCY QUORUM;
```
✅ **Allows fine-tuning between performance and consistency needs.**

---

## **7️⃣4️⃣ How does data distribution work in multi-data center deployments of Cassandra?**  
In multi-data center deployments, Cassandra **replicates data across multiple data centers** to ensure **fault tolerance and availability**.

### **Key Concepts in Multi-DC Replication**
✔ **NetworkTopologyStrategy** → Replicates data across data centers.  
✔ **Local Quorum vs. Global Quorum** → Reads/writes can be **confined to a local data center** for low latency.  
✔ **Gossip Protocol** → Keeps nodes aware of cross-DC topology.

📌 **Example: Setting Up Multi-DC Replication**
```cql
CREATE KEYSPACE my_keyspace 
WITH replication = { 'class': 'NetworkTopologyStrategy', 'DC1': 3, 'DC2': 2 };
```
✅ **Ensures high availability and geo-redundancy.**

---

## **7️⃣5️⃣ Why is Cassandra suitable for IoT use cases?**  
Cassandra is **well-suited for IoT applications** because it efficiently handles **time-series data, high write rates, and distributed deployments**.

### **Why Cassandra is a Good Fit for IoT**
✔ **Handles high-velocity data ingestion**.  
✔ **Supports massive scalability** (horizontal scaling).  
✔ **Optimized for time-series queries** (partitioning by timestamp).  
✔ **Fault tolerance** → Ensures uptime for critical IoT applications.

📌 **Example: Storing IoT Sensor Data**
```cql
CREATE TABLE sensor_data (
    device_id int,
    timestamp timestamp,
    temperature float,
    PRIMARY KEY (device_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```
✅ **Ideal for storing large-scale, real-time sensor data.**

---

## **7️⃣6️⃣ Explain how compaction works in Cassandra.**  
**Compaction** is the process of **merging SSTables** to reduce storage overhead and improve read performance.

### **Types of Compaction**
✔ **Size-Tiered Compaction Strategy (STCS)** → Merges SSTables of similar sizes.  
✔ **Leveled Compaction Strategy (LCS)** → Organizes SSTables into levels.  
✔ **Time-Window Compaction Strategy (TWCS)** → Merges time-based SSTables for time-series data.

📌 **Example: Setting Leveled Compaction**
```cql
ALTER TABLE users WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
✅ **Compaction ensures efficient storage and faster reads.**

---

## **7️⃣7️⃣ What are the different types of keys in Cassandra and how are they used?**  
Cassandra uses **different key types** for **efficient data distribution and retrieval**.

| **Key Type** | **Purpose** | **Example** |
|-------------|------------|------------|
| **Primary Key** | Uniquely identifies each row | `PRIMARY KEY (user_id)` |
| **Partition Key** | Determines how data is distributed across nodes | `PRIMARY KEY ((user_id), order_id)` |
| **Clustering Key** | Orders rows within a partition | `PRIMARY KEY (user_id, order_id)` |
| **Composite Key** | Combines partition & clustering keys | `PRIMARY KEY ((customer_id), order_id, item_id)` |

✅ **Keys define how Cassandra stores and retrieves data efficiently.**

---

## **7️⃣8️⃣ What are SSTables in Cassandra?**  
**SSTables (Sorted String Tables)** are **immutable, disk-based storage files** that store data **persistently**.

### **How SSTables Work**
✔ **Created when Memtable is flushed** to disk.  
✔ **Stored in sorted order** for fast retrieval.  
✔ **Never modified** (new writes create new SSTables).  
✔ **Compaction merges multiple SSTables**.

📌 **Example: SSTable Lifecycle**
1. **Write data to Memtable (RAM)**  
2. **Flush to SSTable (disk)**  
3. **Merge SSTables during compaction**  

✅ **SSTables make Cassandra’s writes efficient and durable.**

---

## **7️⃣9️⃣ How does Cassandra handle failures?**  
Cassandra is **fault-tolerant** and **self-healing**, ensuring **high availability**.

### **Failure Handling Mechanisms**
✔ **Replication** → Copies data across multiple nodes.  
✔ **Hinted Handoff** → Temporarily stores writes for failed nodes.  
✔ **Gossip Protocol** → Detects node failures.  
✔ **Read Repair & Anti-Entropy Repair** → Synchronizes data inconsistencies.  

📌 **Example: Recovering from a Node Failure**
1. **Node goes down** → Other nodes continue serving requests.  
2. **Hinted Handoff replays missed writes** when the node comes back.  
3. **Admin runs `nodetool repair`** to sync data across replicas.  

✅ **Ensures continuous availability even during failures.**

---

## **8️⃣0️⃣ What is Cassandra's Snitch and what does it do?**  
A **Snitch** in Cassandra **determines network topology** and helps **nodes locate replicas efficiently**.

### **Types of Snitches**
✔ **SimpleSnitch** → Basic, single data center setup.  
✔ **GossipingPropertyFileSnitch (Recommended)** → Automatically learns network topology.  
✔ **Ec2Snitch** → Optimized for AWS cloud deployments.  
✔ **RackInferringSnitch** → Assigns nodes to racks based on IP addresses.

📌 **Example: Configuring Snitch (`cassandra.yaml`)**
```yaml
endpoint_snitch: GossipingPropertyFileSnitch
```
✅ **Ensures efficient replica placement for performance and fault tolerance.**

---

## **📌 Summary of Questions 71-80**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **71** | No Joins in Cassandra | Denormalization, Materialized Views instead. |
| **72** | Eventual Consistency | Data syncs across replicas over time. |
| **73** | Tunable Consistency | Adjust read/write consistency per query. |
| **74** | Multi-DC Replication | `NetworkTopologyStrategy` for geo-redundancy. |
| **75** | IoT Use Cases | Handles **high writes, time-series data** efficiently. |
| **76** | Compaction | Merges SSTables to optimize reads. |
| **77** | Key Types | **Partition, Clustering, Composite Keys**. |
| **78** | SSTables | Immutable, disk-based storage format. |
| **79** | Failure Handling | **Hinted Handoff, Replication, Read Repair**. |
| **80** | Snitch | Determines network topology

<br/>
<br/>

## **8️⃣1️⃣ What happens when you run out of disk space in Cassandra?**  
When a Cassandra node **runs out of disk space**, it **stops accepting writes** to prevent corruption.  

### **How Cassandra Handles Low Disk Space**  
✔ **Writes fail with an `OutOfDisk` error**.  
✔ **Read operations still work**, but no new data can be written.  
✔ **Compaction might fail**, causing disk fragmentation.  

### **Preventing Disk Space Issues**  
✔ **Monitor disk usage with `nodetool status`**.  
✔ **Enable auto-compaction to free up space**.  
✔ **Scale out by adding more nodes** (Cassandra scales horizontally).  

📌 **Example: Checking Disk Space Usage**  
```bash
df -h  # Linux command to check disk usage
nodetool info | grep Load  # Shows disk space used per node
```
✅ **To recover, free up space or add more nodes to balance the load.**

---

## **8️⃣2️⃣ What is Apache Cassandra's strategy for handling data evictions?**  
Cassandra **does not evict data automatically** like traditional caching systems. Instead, it **uses TTL and Compaction** to remove old data.  

### **Data Eviction Strategies in Cassandra**  
✔ **TTL (Time-to-Live):** Auto-expires data after a set period.  
✔ **Compaction:** Merges SSTables and deletes tombstones.  
✔ **Memtable Flush:** Old data is flushed to disk when memory is full.  

📌 **Example: Using TTL for Auto-Expiration**  
```cql
INSERT INTO users (user_id, name) 
VALUES (1001, 'Alice') 
USING TTL 86400;  -- Deletes after 1 day
```
✅ **Ensures outdated data does not take up unnecessary space.**  

---

## **8️⃣3️⃣ What happens when a Cassandra node goes down during a write operation?**  
Cassandra is **highly fault-tolerant**, so **writes still succeed even if some nodes are down**.

### **Handling Node Failures During Writes**  
✔ **Hinted Handoff:** Stores missed writes temporarily and applies them when the node comes back.  
✔ **Replication Factor:** Ensures copies of data exist on other nodes.  
✔ **Read Repair:** Fixes inconsistencies when the node recovers.  

📌 **Example: Writing Data with Replication Factor 3**  
```cql
INSERT INTO orders (order_id, amount) 
VALUES (101, 500) USING CONSISTENCY QUORUM;
```
- If **one node is down**, the write still succeeds on **two replicas**.  
- The down node **gets updated later via Hinted Handoff or Repair**.  

✅ **Ensures no data loss, even during failures.**

---

## **8️⃣4️⃣ How can you minimize read latencies in Cassandra?**  
Read performance in Cassandra depends on **proper data modeling and tuning**.

### **Ways to Reduce Read Latency**  
✔ **Use Primary Key queries** instead of filtering.  
✔ **Enable Caching (Row & Key Cache)** for frequent lookups.  
✔ **Use Materialized Views** for precomputed queries.  
✔ **Choose Leveled Compaction Strategy (LCS)** for read-heavy workloads.  
✔ **Avoid Secondary Indexes for high-cardinality fields**.  

📌 **Example: Enabling Row Cache for Faster Reads**  
```cql
ALTER TABLE users 
WITH caching = { 'keys': 'ALL', 'rows_per_partition': '10' };
```
✅ **Optimized data access ensures faster read queries.**

---

## **8️⃣5️⃣ What is the impact of Consistency Level on Cassandra's performance?**  
**Higher consistency levels** provide **stronger guarantees** but **reduce availability and speed**.

### **Performance Trade-offs of Different Consistency Levels**  
| **Consistency Level** | **Performance Impact** | **Use Case** |
|----------------------|----------------|------------------|
| **ONE** | Fastest, but may return stale data. | Real-time analytics, logs. |
| **QUORUM** | Balances consistency & speed. | Banking, payments. |
| **ALL** | Strongest consistency, slowest. | Critical data, ledgers. |

📌 **Example: Writing Data with High Consistency**  
```cql
INSERT INTO transactions (id, amount) 
VALUES (2001, 1000) USING CONSISTENCY ALL;
```
✅ **Tuning consistency helps balance speed and correctness.**

---

## **8️⃣6️⃣ What is the purpose of Apache Cassandra's Coordinator node?**  
The **Coordinator node** is the **first node that receives a client request** and forwards it to replica nodes.

### **Responsibilities of the Coordinator Node**  
✔ **Routes queries to the appropriate replica nodes.**  
✔ **Applies consistency level rules before sending a response.**  
✔ **Performs Read Repair if needed.**  

📌 **Example: Coordinator Routing a Write Request**
1. **Client sends write request to `Node A` (Coordinator).**  
2. **Node A forwards the write to replica nodes.**  
3. **After enough replicas acknowledge, Node A responds to the client.**

✅ **Ensures efficient query routing and consistency management.**

---

## **8️⃣7️⃣ How can you mitigate the impact of "wide rows" in Cassandra?**  
Wide rows **occur when a partition contains too many records**, slowing down reads.

### **How to Handle Wide Rows in Cassandra**  
✔ **Use Composite Partition Keys** to distribute data across nodes.  
✔ **Limit rows per partition** (avoid exceeding **100MB per partition**).  
✔ **Use Time-to-Live (TTL)** for automatic deletion of old data.  

📌 **Example: Breaking Large Partitions Using Composite Keys**  
```cql
CREATE TABLE sensor_data (
    device_id int,
    day int,
    timestamp timestamp,
    value float,
    PRIMARY KEY ((device_id, day), timestamp)
);
```
✅ **Prevents slow queries and high memory usage.**

---

## **8️⃣8️⃣ What is vnode and what is its purpose in Cassandra?**  
**Virtual Nodes (vnodes)** allow **better data distribution and scalability** in a Cassandra cluster.

### **Advantages of vnodes**  
✔ **Evenly distributes data across nodes**.  
✔ **Makes cluster expansion easier** (no manual token assignment).  
✔ **Improves fault tolerance** (data is spread across more nodes).  

📌 **Example: Setting vnodes in `cassandra.yaml`**  
```yaml
num_tokens: 256  # Default number of vnodes per node
```
✅ **Vnodes simplify node management and balance workloads automatically.**

---

## **8️⃣9️⃣ How does Cassandra handle large blobs of data?**  
Cassandra **can store large binary objects (BLOBs), but it's not optimized for very large files**.

### **Handling Large Data in Cassandra**  
✔ **Use BLOB type for binary data (images, PDFs).**  
✔ **Store large files externally (S3, HDFS) and keep metadata in Cassandra.**  
✔ **Chunk large data across multiple partitions** to avoid performance issues.  

📌 **Example: Storing a Small BLOB in Cassandra**  
```cql
CREATE TABLE user_files (
    user_id int PRIMARY KEY,
    file_data blob
);
```
✅ **For very large data, it's better to store references instead of the full file.**

---

## **9️⃣0️⃣ Explain how Tombstones work in Cassandra.**  
A **Tombstone** is a **marker that indicates a deleted row or column** in Cassandra.

### **How Tombstones Work**  
✔ **Deletes do not remove data immediately**.  
✔ **Tombstones are stored until compaction removes them**.  
✔ **Avoid too many tombstones, as they slow down queries**.  

📌 **Example: Deleting Data (Creates a Tombstone)**  
```cql
DELETE FROM users WHERE user_id = 1001;
```
✔ The row is **not deleted immediately** but marked as a tombstone.  
✔ **After `gc_grace_seconds`, compaction removes the tombstone.**  

✅ **Too many tombstones can cause performance issues ("Tombstone Overhead").**

---

## **📌 Summary of Questions 81-90**
| **Q#** | **Topic** | **Key Points** |
|-------|----------|-------------|
| **81** | Out of Disk Space | Writes stop, scale out or free space. |
| **82** | Data Eviction | Uses **TTL and Compaction** instead of auto-eviction. |
| **83** | Node Failure During Writes | **Hinted Handoff, Read Repair, Replication**. |
| **84** | Minimize Read Latency | **Caching, Primary Key Queries, LCS**. |
| **85** | Consistency vs. Performance | `ONE` (fast), `QUORUM` (balanced), `ALL` (strong). |
| **86** | Coordinator Node | Routes queries and ensures consistency. |
| **87** | Wide Row Mitigation | **Composite Keys, Partitioning, TTL**. |
| **88** | Virtual Nodes (vnodes) | Improve **load balancing & fault tolerance**. |
| **89** | Handling Large Data | Store references, **use BLOBs for small files**. |
| **90** | Tombstones | Delayed deletes, removed by **compaction**. |

---

