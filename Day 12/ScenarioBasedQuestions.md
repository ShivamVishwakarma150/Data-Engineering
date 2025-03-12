# **📌 Scenario-Based MongoDB Interview Questions**  

Below is a **list of real-world MongoDB scenarios** that test your understanding of **database design, performance tuning, transactions, replication, and scaling**.

---

## **📂 1. Database Design Scenarios**  
1. **E-commerce Platform:** How would you design a MongoDB schema to store products, orders, and customers efficiently?  
2. **Social Media Network:** How would you design a database to handle user profiles, posts, likes, and comments?  
3. **Blogging Platform:** How would you structure a blogging system where users can write articles, comment, and like posts?  
4. **Real-Time Chat Application:** How would you design a schema for a chat system where messages need to be stored and retrieved in real-time?  
5. **Healthcare System:** How would you design a MongoDB database to store patient records, appointments, and medical histories?  

---

## **📊 2. Query Optimization & Performance Scenarios**  
6. **Slow Query Optimization:** You notice a query taking several seconds to execute. How would you identify and fix the issue?  
7. **Indexing Strategy:** You need to optimize search queries on a collection containing millions of documents. How would you design an efficient indexing strategy?  
8. **Pagination Handling:** How would you implement efficient pagination for a product catalog with millions of records?  
9. **Handling Large Data Inserts:** If an application needs to insert millions of records in real-time, how would you optimize the process?  
10. **Reducing Write Latency:** Your application has frequent writes. How can you reduce write latency while maintaining data integrity?  

---

## **🛠 3. Transactions & Consistency Scenarios**  
11. **Atomic Operations:** How would you ensure atomicity when updating multiple documents that must be consistent?  
12. **Inventory Management:** How would you ensure product stock updates remain consistent when multiple users place orders simultaneously?  
13. **Financial Transactions:** How would you design a MongoDB transaction system to handle money transfers between users?  
14. **Multi-Document Transactions:** Your application requires updates to multiple collections in a single transaction. How would you implement this in MongoDB?  
15. **Rollback Mechanism:** How would you handle a situation where a transaction fails and needs to be rolled back?  

---

## **📡 4. Replication & High Availability Scenarios**  
16. **Replica Set Failover:** What happens if the primary node in a replica set fails? How can you ensure high availability?  
17. **Read Preference Strategy:** Your application needs fast read performance. How would you configure read preference to balance between primary and secondary nodes?  
18. **Ensuring Data Durability:** How would you configure Write Concern to ensure data durability in a high-traffic system?  
19. **Handling Network Partitions:** How would you design a MongoDB system to remain functional during a network failure?  
20. **Backup & Disaster Recovery:** How would you set up an automated backup and recovery system for MongoDB?  

---

## **📡 5. Sharding & Scaling Scenarios**  
21. **Sharding Implementation:** How would you enable and configure sharding for a collection with billions of documents?  
22. **Choosing the Right Shard Key:** How would you select the best shard key to ensure balanced data distribution?  
23. **Uneven Shard Load:** What steps would you take if one shard is handling significantly more traffic than others?  
24. **Cross-Shard Queries:** How would you handle queries that need to retrieve data from multiple shards efficiently?  
25. **Resharding Strategy:** If the shard key is no longer optimal, how would you migrate to a new shard key without downtime?  

---

## **📡 6. Migration & Data Handling Scenarios**  
26. **SQL to MongoDB Migration:** How would you migrate a relational database to MongoDB while preserving relationships?  
27. **Large Data Set Migration:** How would you transfer a large collection from one database to another with minimal downtime?  
28. **Changing Collection Schema:** How would you modify a schema in a running production system without downtime?  
29. **Merging Data from Multiple Collections:** How would you restructure data spread across multiple collections into a single optimized collection?  
30. **Handling Soft Deletes:** How would you implement a "soft delete" mechanism without physically deleting documents?  

---

## **📡 7. Security & Compliance Scenarios**  
31. **Securing a MongoDB Cluster:** How would you prevent unauthorized access to a MongoDB database in production?  
32. **Data Encryption:** How would you implement data encryption for sensitive information stored in MongoDB?  
33. **Role-Based Access Control (RBAC):** How would you set up MongoDB users with different access privileges?  
34. **Preventing Injection Attacks:** How would you protect against NoSQL injection vulnerabilities?  
35. **Audit Logging:** How would you track and log changes made to the database for compliance purposes?  

---

## **📡 8. Real-Time Data & Analytics Scenarios**  
36. **Building a Real-Time Analytics Dashboard:** How would you design a MongoDB system for real-time data aggregation and visualization?  
37. **IoT Data Processing:** How would you store and analyze sensor data coming from thousands of IoT devices in real-time?  
38. **User Activity Tracking:** How would you design a system to track and analyze user behavior on a website?  
39. **Event-Driven Processing:** How would you handle real-time event streaming and processing in MongoDB?  
40. **Time-Series Data Handling:** How would you efficiently store and retrieve time-series data in MongoDB?  

---

## **📡 9. Logging & Monitoring Scenarios**  
41. **Log Storage Optimization:** Your application generates millions of logs daily. How would you store and manage them efficiently?  
42. **Slow Query Logging:** How would you enable logging of slow queries and optimize them?  
43. **Monitoring Query Performance:** How would you track and analyze query performance over time?  
44. **Handling High Write Throughput Logs:** How would you ensure MongoDB handles high-frequency log writes without performance degradation?  
45. **Automated Alerts:** How would you configure alerts to notify the team of database performance issues?  

---

## **📡 10. Miscellaneous MongoDB Scenarios**  
46. **Offline Mode with Sync:** How would you design an application that allows offline data access and syncs with MongoDB when online?  
47. **Multi-Tenant SaaS Application:** How would you design a multi-tenant system where each customer has an isolated dataset?  
48. **Geospatial Queries:** How would you design a system for location-based services (e.g., nearby restaurants)?  
49. **MongoDB in a Microservices Architecture:** How would you design MongoDB usage for a microservices-based system?  
50. **Versioning in MongoDB:** How would you track changes and maintain historical versions of documents?  

---

# **📌 Summary - Scenario-Based MongoDB Questions**
| **Category** | **Questions** |
|-------------|--------------|
| **Database Design** | 1-5 |
| **Query Optimization & Performance** | 6-10 |
| **Transactions & Consistency** | 11-15 |
| **Replication & High Availability** | 16-20 |
| **Sharding & Scaling** | 21-25 |
| **Migration & Data Handling** | 26-30 |
| **Security & Compliance** | 31-35 |
| **Real-Time Data & Analytics** | 36-40 |
| **Logging & Monitoring** | 41-45 |
| **Miscellaneous MongoDB** | 46-50 |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 1-5 (Explained in Detail)**  

---

## **1️⃣ Scenario: E-commerce Platform – Designing a MongoDB Schema for Products, Orders, and Customers**  
🔹 **Requirements:**  
- Customers can **browse and purchase products**.  
- Each order must store **customer details, products, and total cost**.  
- Products can have **multiple variations (size, color, brand, etc.)**.  

### **🔹 Schema Design**
✅ **Customers Collection**
```js
db.customers.insertOne({
    _id: ObjectId("650c3c8e12f3f6a19f92e2a3"),
    name: "Alice Johnson",
    email: "alice@example.com",
    phone: "+1-123-456-7890",
    address: {
        street: "123 Main St",
        city: "Los Angeles",
        state: "CA",
        zipcode: "90001"
    },
    createdAt: new Date()
});
```

✅ **Products Collection**
```js
db.products.insertOne({
    _id: ObjectId("650c3c8e12f3f6a19f92e2b4"),
    name: "Wireless Headphones",
    brand: "Sony",
    category: "Electronics",
    price: 99.99,
    stock: 150,
    variations: [
        { color: "Black", model: "WH-1000XM4" },
        { color: "Silver", model: "WH-1000XM5" }
    ],
    createdAt: new Date()
});
```

✅ **Orders Collection**
```js
db.orders.insertOne({
    _id: ObjectId("650c3c8e12f3f6a19f92e3c5"),
    customerId: ObjectId("650c3c8e12f3f6a19f92e2a3"),
    products: [
        { productId: ObjectId("650c3c8e12f3f6a19f92e2b4"), quantity: 1, price: 99.99 }
    ],
    totalAmount: 99.99,
    orderStatus: "Confirmed",
    orderDate: new Date()
});
```

🚀 **Use Case:** **E-commerce websites (Amazon, Flipkart, Shopify).**

---

## **2️⃣ Scenario: Social Media Network – Handling User Profiles, Posts, Likes, and Comments**  
🔹 **Requirements:**  
- Users can **create posts, comment, and like other posts**.  
- Each post stores **text, images, and reactions**.  
- Comments should be **linked to posts and users**.  

### **🔹 Schema Design**
✅ **Users Collection**
```js
db.users.insertOne({
    _id: ObjectId("660c3c8e12f3f6a19f92a1b1"),
    name: "John Doe",
    email: "john@example.com",
    friends: [ObjectId("660c3c8e12f3f6a19f92a1b2"), ObjectId("660c3c8e12f3f6a19f92a1b3")],
    createdAt: new Date()
});
```

✅ **Posts Collection**
```js
db.posts.insertOne({
    _id: ObjectId("660c3c8e12f3f6a19f92a2c3"),
    userId: ObjectId("660c3c8e12f3f6a19f92a1b1"),
    content: "Just visited New York! Amazing city!",
    images: ["nyc1.jpg", "nyc2.jpg"],
    likes: [ObjectId("660c3c8e12f3f6a19f92a1b2"), ObjectId("660c3c8e12f3f6a19f92a1b3")],
    comments: [
        { userId: ObjectId("660c3c8e12f3f6a19f92a1b2"), text: "Wow! That looks amazing!", createdAt: new Date() }
    ],
    createdAt: new Date()
});
```

🚀 **Use Case:** **Social media platforms (Facebook, Twitter, Instagram).**

---

## **3️⃣ Scenario: Blogging Platform – Handling Articles, Comments, and Likes**  
🔹 **Requirements:**  
- Users can **write, edit, and delete blog posts**.  
- Users can **comment on articles**.  
- Articles should support **tags and categories** for better searchability.  

### **🔹 Schema Design**
✅ **Users Collection**
```js
db.users.insertOne({
    _id: ObjectId("670c3c8e12f3f6a19f92b3d1"),
    name: "Emma Watson",
    email: "emma@example.com",
    createdAt: new Date()
});
```

✅ **Articles Collection**
```js
db.articles.insertOne({
    _id: ObjectId("670c3c8e12f3f6a19f92b3e2"),
    authorId: ObjectId("670c3c8e12f3f6a19f92b3d1"),
    title: "Why MongoDB is the Best NoSQL Database?",
    content: "MongoDB offers great scalability and flexibility...",
    tags: ["MongoDB", "NoSQL", "Databases"],
    likes: [ObjectId("670c3c8e12f3f6a19f92b3d1")],
    comments: [
        { userId: ObjectId("670c3c8e12f3f6a19f92b3d1"), text: "Great article!", createdAt: new Date() }
    ],
    createdAt: new Date()
});
```

🚀 **Use Case:** **Medium, WordPress, Dev.to**.

---

## **4️⃣ Scenario: Real-Time Chat Application – Handling Messages and Conversations**  
🔹 **Requirements:**  
- Users should be able to **send & receive messages in real-time**.  
- Messages should be **stored in conversations** between users.  
- Each message should have a **status (sent, delivered, read)**.  

### **🔹 Schema Design**
✅ **Users Collection**
```js
db.users.insertOne({
    _id: ObjectId("680c3c8e12f3f6a19f92c4f1"),
    name: "Michael Scott",
    email: "michael@example.com",
    createdAt: new Date()
});
```

✅ **Conversations Collection**
```js
db.conversations.insertOne({
    _id: ObjectId("680c3c8e12f3f6a19f92c4f2"),
    participants: [ObjectId("680c3c8e12f3f6a19f92c4f1"), ObjectId("680c3c8e12f3f6a19f92c4f3")],
    messages: [
        { senderId: ObjectId("680c3c8e12f3f6a19f92c4f1"), text: "Hello!", status: "sent", createdAt: new Date() }
    ],
    createdAt: new Date()
});
```

🚀 **Use Case:** **WhatsApp, Slack, Facebook Messenger**.

---

## **5️⃣ Scenario: Healthcare System – Handling Patients, Appointments, and Medical Records**  
🔹 **Requirements:**  
- Patients should have **personal details, medical history, and appointments**.  
- Doctors should be able to **view patient details and prescribe treatments**.  
- Appointments should be **scheduled and updated dynamically**.  

### **🔹 Schema Design**
✅ **Patients Collection**
```js
db.patients.insertOne({
    _id: ObjectId("690c3c8e12f3f6a19f92d5g1"),
    name: "Sarah Brown",
    email: "sarah@example.com",
    age: 30,
    medicalHistory: [
        { condition: "Diabetes", diagnosedAt: new Date("2022-03-15") }
    ],
    createdAt: new Date()
});
```

✅ **Appointments Collection**
```js
db.appointments.insertOne({
    _id: ObjectId("690c3c8e12f3f6a19f92d5g2"),
    patientId: ObjectId("690c3c8e12f3f6a19f92d5g1"),
    doctor: "Dr. John Smith",
    appointmentDate: new Date("2024-07-10T10:00:00"),
    status: "Scheduled",
    createdAt: new Date()
});
```

🚀 **Use Case:** **Hospital Management Systems, Telemedicine Apps**.

---

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 6-10 (Query Optimization & Performance)**  

These scenarios focus on **optimizing slow queries, indexing strategies, pagination, bulk inserts, and reducing write latency** in MongoDB.  

---

## **6️⃣ Scenario: Slow Query Optimization**  
🔹 **Problem:** A query is taking **several seconds** to execute.  
🔹 **Requirements:**  
- Identify why the query is slow.  
- Optimize it using indexes and query techniques.  

### **🔹 Solution: Steps to Identify & Fix Slow Queries**  

✅ **Step 1: Analyze the Query Execution Plan**  
```js
db.orders.find({ customerId: "C123" }).explain("executionStats");
```
- **Check if the query is performing a COLLSCAN (Collection Scan)** instead of an INDEXSCAN.  

✅ **Step 2: Create an Index on `customerId` for Faster Lookups**  
```js
db.orders.createIndex({ customerId: 1 });
```

✅ **Step 3: Optimize Query with Projection (Retrieve Only Required Fields)**  
```js
db.orders.find({ customerId: "C123" }, { _id: 0, orderId: 1, totalAmount: 1 });
```

🚀 **Use Case:** **Large e-commerce databases where order retrieval must be fast.**  

---

## **7️⃣ Scenario: Indexing Strategy for Fast Search Queries**  
🔹 **Problem:** Searching for products by name and category is slow.  
🔹 **Requirements:**  
- Improve search query performance.  
- Use compound indexes.  

### **🔹 Solution: Using Indexing for Faster Search**  

✅ **Step 1: Create a Compound Index on `name` and `category`**  
```js
db.products.createIndex({ name: 1, category: 1 });
```

✅ **Step 2: Use an Efficient Query That Matches the Index**  
```js
db.products.find({ name: "Laptop", category: "Electronics" });
```

✅ **Step 3: Use a Covered Query (Only Retrieve Indexed Fields)**  
```js
db.products.find({ category: "Electronics" }, { name: 1, price: 1, _id: 0 });
```

🚀 **Use Case:** **Online stores where product search needs to be near-instantaneous.**  

---

## **8️⃣ Scenario: Efficient Pagination Handling**  
🔹 **Problem:** The UI needs to display paginated product listings.  
🔹 **Requirements:**  
- Implement efficient pagination without performance degradation.  

### **🔹 Solution: Using `skip()` and `limit()` Efficiently**  

✅ **Step 1: Implement Offset-Based Pagination**  
```js
db.products.find().skip(50).limit(10);
```
🚨 **Problem with `skip()`:** MongoDB **still scans the previous records**, slowing down queries for large datasets.

✅ **Step 2: Use Indexed Pagination for Better Performance**  
```js
db.products.find({ _id: { $gt: last_seen_id } }).limit(10);
```
- This method **avoids scanning previous records**.

🚀 **Use Case:** **Social media feeds, e-commerce product listings.**  

---

## **9️⃣ Scenario: Handling Large Data Inserts Efficiently**  
🔹 **Problem:** A system needs to insert millions of records daily.  
🔹 **Requirements:**  
- Optimize bulk inserts for performance.  

### **🔹 Solution: Use `insertMany()` Instead of `insertOne()`**  

✅ **Step 1: Insert Data in Batches Using `insertMany()`**
```js
db.orders.insertMany([
    { orderId: 101, customerId: "C123", totalAmount: 50.99 },
    { orderId: 102, customerId: "C124", totalAmount: 75.49 }
]);
```
🚨 **Avoid inserting records one by one, as it causes high latency.**  

✅ **Step 2: Disable Indexing Temporarily During Bulk Inserts**  
```js
db.products.dropIndex("category_1");
db.products.insertMany([...]);  // Insert Data
db.products.createIndex({ category: 1 });  // Re-enable Index
```
- **Disabling indexes during bulk inserts** improves performance.

🚀 **Use Case:** **Log processing, IoT sensor data ingestion.**  

---

## **🔟 Scenario: Reducing Write Latency in a High-Write System**  
🔹 **Problem:** A high-traffic application is experiencing slow writes.  
🔹 **Requirements:**  
- Optimize MongoDB write performance.  
- Maintain **data durability**.  

### **🔹 Solution: Using Write Concern & Index Optimization**  

✅ **Step 1: Adjust Write Concern Based on Requirements**  
```js
db.orders.insertOne(
    { orderId: 103, customerId: "C125", totalAmount: 120.75 },
    { writeConcern: { w: 1, j: false } }  // Faster but may lose data in crash
);
```
- **`w: 1`** → Acknowledges write after it reaches **primary node** (faster).  
- **`j: false`** → **Disables journaling** (reduces durability but improves speed).  

✅ **Step 2: Reduce Indexing Overhead for High-Write Collections**  
```js
db.logs.dropIndex("createdAt_1");
```
- **Fewer indexes = Faster writes**.

✅ **Step 3: Use Sharding for Write Scalability**  
```js
sh.enableSharding("ecommerceDB");
sh.shardCollection("ecommerceDB.orders", { orderId: "hashed" });
```

🚀 **Use Case:** **Real-time analytics, gaming leaderboards, high-frequency trading systems.**  

---

# **📌 Summary - MongoDB Query Optimization Scenarios (6-10)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **6. Slow Query Optimization** | Query takes seconds to execute | **Analyze with `.explain()`, use Indexing, optimize projections** |
| **7. Indexing Strategy** | Searching by fields is slow | **Use Compound Indexes, Covered Queries** |
| **8. Pagination Handling** | `skip()` is slow for large datasets | **Use Indexed Pagination (`_id > last_seen_id`)** |
| **9. Bulk Insert Optimization** | Millions of records per day | **Use `insertMany()`, disable indexing temporarily** |
| **10. Write Latency Reduction** | Slow writes under high load | **Optimize Write Concern, Reduce Indexing, Use Sharding** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 11-15 (Transactions & Consistency)**  

These scenarios focus on **ensuring data consistency, atomic transactions, handling concurrent writes, and rollback mechanisms in MongoDB**.

---

## **1️⃣1️⃣ Scenario: Ensuring Atomicity in Multi-Document Updates**  
🔹 **Problem:** A **banking system** needs to update multiple documents **atomically** when transferring money.  
🔹 **Requirements:**  
- If one update fails, the entire operation should **roll back**.  
- Both sender and receiver balances must remain **consistent**.  

### **🔹 Solution: Use Transactions in MongoDB (v4.0+)**  

✅ **Step 1: Start a Session and a Transaction**
```js
const session = db.getMongo().startSession();
session.startTransaction();

try {
    // Deduct balance from sender
    db.accounts.updateOne(
        { _id: "A123" },
        { $inc: { balance: -500 } },
        { session }
    );

    // Add balance to receiver
    db.accounts.updateOne(
        { _id: "B456" },
        { $inc: { balance: 500 } },
        { session }
    );

    // Commit transaction if both updates succeed
    session.commitTransaction();
} catch (error) {
    session.abortTransaction();  // Rollback if any operation fails
}

session.endSession();
```
🔹 **Why use transactions?**  
- Ensures **atomicity** across multiple documents.  
- Prevents **partial updates** that could leave inconsistent data.  

🚀 **Use Case:** **Banking, digital wallets, financial applications.**  

---

## **1️⃣2️⃣ Scenario: Handling Concurrent Order Placements in E-Commerce**  
🔹 **Problem:** Multiple users are trying to **buy the same product**, but stock should not go below zero.  
🔹 **Requirements:**  
- Ensure **correct stock updates** under high concurrency.  
- Prevent **overselling** when multiple users order the last product.  

### **🔹 Solution: Use `findAndModify()` for Atomic Stock Updates**  

✅ **Step 1: Deduct Stock Atomically**
```js
const order = db.products.findOneAndUpdate(
    { _id: ObjectId("650c3c8e12f3f6a19f92e2b4"), stock: { $gt: 0 } },
    { $inc: { stock: -1 } },
    { returnNewDocument: true }
);
```
🔹 **How it works:**  
- Ensures **atomic stock deduction**.  
- **If stock is `0`**, update fails, preventing overselling.  

🚀 **Use Case:** **E-commerce platforms (Amazon, Flipkart, Shopify).**  

---

## **1️⃣3️⃣ Scenario: Ensuring Data Integrity When Updating Related Collections**  
🔹 **Problem:** A **user profile update** should also update associated records (e.g., orders, reviews).  
🔹 **Requirements:**  
- Ensure **all updates are successful** or **rollback if any fail**.  

### **🔹 Solution: Use Transactions for Consistency Across Collections**  

✅ **Step 1: Update User Profile & Related Orders in a Single Transaction**
```js
const session = db.getMongo().startSession();
session.startTransaction();

try {
    // Update User Profile
    db.users.updateOne(
        { _id: ObjectId("U123") },
        { $set: { email: "newemail@example.com" } },
        { session }
    );

    // Update All Orders for That User
    db.orders.updateMany(
        { userId: ObjectId("U123") },
        { $set: { userEmail: "newemail@example.com" } },
        { session }
    );

    // Commit Transaction
    session.commitTransaction();
} catch (error) {
    session.abortTransaction();  // Rollback if any update fails
}

session.endSession();
```
🔹 **Why use a transaction?**  
- Ensures **email change reflects in both users & orders collection**.  
- Prevents **data inconsistencies** if one update fails.  

🚀 **Use Case:** **User profile management in SaaS applications.**  

---

## **1️⃣4️⃣ Scenario: Multi-Document Transactions in a Shopping Cart System**  
🔹 **Problem:** A **shopping cart checkout process** needs to update multiple documents together.  
🔹 **Requirements:**  
- Deduct stock **only if payment succeeds**.  
- Prevent **orphaned orders** if the payment fails.  

### **🔹 Solution: Use Transactions for Atomic Checkout**  

✅ **Step 1: Perform Order & Payment Update in One Transaction**
```js
const session = db.getMongo().startSession();
session.startTransaction();

try {
    // Create Order Document
    const order = db.orders.insertOne(
        {
            userId: ObjectId("U123"),
            items: [{ productId: "P101", quantity: 2 }],
            totalAmount: 200,
            status: "Processing",
            createdAt: new Date()
        },
        { session }
    );

    // Deduct Stock
    db.products.updateOne(
        { _id: "P101", stock: { $gte: 2 } },
        { $inc: { stock: -2 } },
        { session }
    );

    // Mark Order as Paid
    db.orders.updateOne(
        { _id: order.insertedId },
        { $set: { status: "Paid" } },
        { session }
    );

    // Commit Transaction
    session.commitTransaction();
} catch (error) {
    session.abortTransaction();  // Rollback if payment or stock update fails
}

session.endSession();
```
🔹 **Why use transactions?**  
- Ensures **payment and stock update happen together**.  
- If any part fails, **order remains unprocessed**.  

🚀 **Use Case:** **E-commerce checkout flow.**  

---

## **1️⃣5️⃣ Scenario: Handling Failed Transactions with Rollback Mechanism**  
🔹 **Problem:** An **employee payroll update** must be rolled back if any part fails.  
🔹 **Requirements:**  
- Update **salary in payroll**.  
- Update **tax deductions**.  
- If either fails, **revert the entire transaction**.  

### **🔹 Solution: Implementing a Rollback Strategy in MongoDB**  

✅ **Step 1: Use Transactions with Rollback**
```js
const session = db.getMongo().startSession();
session.startTransaction();

try {
    // Update Payroll
    db.payroll.updateOne(
        { employeeId: "E123" },
        { $set: { salary: 70000 } },
        { session }
    );

    // Update Tax Deductions
    db.taxes.updateOne(
        { employeeId: "E123" },
        { $set: { taxDeducted: 5000 } },
        { session }
    );

    // Commit Transaction if Both Updates Succeed
    session.commitTransaction();
} catch (error) {
    session.abortTransaction();  // Rollback if any step fails
}

session.endSession();
```
🔹 **Why use transactions?**  
- Prevents **partial payroll updates**.  
- Ensures **data consistency** across collections.  

🚀 **Use Case:** **Payroll processing, financial applications.**  

---

# **📌 Summary - MongoDB Transactions & Consistency Scenarios (11-15)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **11. Atomic Multi-Document Updates** | Money transfer requires consistent updates | **Use Transactions (`commitTransaction()`, `abortTransaction()`)** |
| **12. Concurrent Order Placement** | Prevent overselling in e-commerce | **Use `findAndModify()` for atomic updates** |
| **13. Updating Related Collections** | User profile update should reflect in multiple collections | **Use transactions to ensure consistency** |
| **14. Shopping Cart Checkout** | Payment and stock update must be atomic | **Use a single transaction for both updates** |
| **15. Rollback on Failed Transactions** | Payroll update must be reverted if any part fails | **Use `abortTransaction()` for rollback** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 16-20 (Replication & High Availability)**  

These scenarios focus on **MongoDB replication, failover handling, read preferences, network partitioning, and backup strategies**.

---

## **1️⃣6️⃣ Scenario: Handling Replica Set Failover in MongoDB**  
🔹 **Problem:** If the **primary node fails**, the application must continue functioning **without downtime**.  
🔹 **Requirements:**  
- Automatically **promote a secondary node** to primary.  
- Ensure **no data loss** during failover.  

### **🔹 Solution: Configure a MongoDB Replica Set for Automatic Failover**  

✅ **Step 1: Create a Replica Set with Three Nodes**  
```bash
mongod --replSet "rs0" --port 27017 --dbpath /data/rs0 --bind_ip_all
mongod --replSet "rs0" --port 27018 --dbpath /data/rs1 --bind_ip_all
mongod --replSet "rs0" --port 27019 --dbpath /data/rs2 --bind_ip_all
```

✅ **Step 2: Initialize the Replica Set**  
```js
rs.initiate({
    _id: "rs0",
    members: [
        { _id: 0, host: "localhost:27017", priority: 2 },
        { _id: 1, host: "localhost:27018", priority: 1 },
        { _id: 2, host: "localhost:27019", priority: 1 }
    ]
});
```
🔹 **How Failover Works:**  
- If the **primary fails**, MongoDB **automatically elects the secondary** with the highest priority as the new primary.  
- The application continues **without downtime**.  

🚀 **Use Case:** **High-availability applications that require 24/7 uptime (e.g., banking systems, stock markets).**  

---

## **1️⃣7️⃣ Scenario: Configuring Read Preferences for Load Balancing**  
🔹 **Problem:** The primary node is **overloaded** with both **reads and writes**.  
🔹 **Requirements:**  
- Distribute **read traffic across secondaries** while keeping **writes on the primary**.  

### **🔹 Solution: Use Read Preferences to Distribute Load**  

✅ **Step 1: Configure Read Preference to Use Secondary Nodes**
```js
db.getMongo().setReadPref("secondaryPreferred");
```
✅ **Step 2: Specify Read Preference in Application Connection**  
```js
const client = new MongoClient("mongodb://primaryNode,secondaryNode1,secondaryNode2/?readPreference=secondaryPreferred");
```

🔹 **Why Use `secondaryPreferred`?**  
- **Reads go to secondary nodes**, reducing load on the primary.  
- If no secondary is available, **reads fall back to the primary**.  

🚀 **Use Case:** **Analytics dashboards, reporting systems where real-time data consistency is not critical.**  

---

## **1️⃣8️⃣ Scenario: Ensuring Data Durability with Write Concern**  
🔹 **Problem:** The application must **ensure that writes are safely stored** across multiple nodes before confirming a transaction.  
🔹 **Requirements:**  
- **Prevent data loss** during sudden crashes.  
- Confirm writes only when **replicated to multiple nodes**.  

### **🔹 Solution: Use Write Concern for Safe Writes**  

✅ **Step 1: Use `writeConcern: "majority"` to Ensure Durability**
```js
db.orders.insertOne(
    { orderId: 5001, status: "Processing", totalAmount: 250.00 },
    { writeConcern: { w: "majority", j: true, wtimeout: 5000 } }
);
```
🔹 **Explanation:**  
- **`w: "majority"`** → Write is **acknowledged only when the majority of replica set members confirm it**.  
- **`j: true`** → Ensures the write is **flushed to disk (journaling enabled)**.  
- **`wtimeout: 5000`** → If nodes don’t acknowledge within **5 seconds**, the write fails.  

🚀 **Use Case:** **Banking & healthcare applications where data consistency is critical.**  

---

## **1️⃣9️⃣ Scenario: Handling Network Partitions in a Distributed MongoDB Setup**  
🔹 **Problem:** Some **shards or replica set members become unreachable** due to a **network partition**.  
🔹 **Requirements:**  
- Ensure that **the cluster continues functioning** even if some nodes become unavailable.  
- Prevent **data inconsistencies** when the network is restored.  

### **🔹 Solution: Configure MongoDB to Handle Partition Tolerance (`P` in CAP Theorem)**  

✅ **Step 1: Use Majority Write Concern to Prevent Split Brain Issues**
```js
db.orders.insertOne(
    { orderId: 5002, status: "Shipped" },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
);
```
✅ **Step 2: Ensure Read Operations Work Even During Partition**
```js
db.getMongo().setReadPref("nearest");
```
🔹 **How It Works:**  
- **Ensures only the majority replica members accept writes**, preventing split-brain scenarios.  
- **Allows reads from the nearest available node**, reducing latency.  

🚀 **Use Case:** **Global applications where MongoDB runs in multiple data centers across different regions.**  

---

## **2️⃣0️⃣ Scenario: Setting Up Automated Backups and Disaster Recovery**  
🔹 **Problem:** MongoDB must be backed up **regularly** to **recover from failures** or **accidental deletions**.  
🔹 **Requirements:**  
- Implement an **automated backup strategy**.  
- Support **point-in-time recovery**.  

### **🔹 Solution: Use `mongodump` for Regular Backups**  

✅ **Step 1: Automate Daily Backups Using a Cron Job**
```bash
crontab -e
```
```bash
0 2 * * * mongodump --uri="mongodb://localhost:27017" --out=/backups/mongo-$(date +\%F)
```
🔹 **Explanation:**  
- Runs **`mongodump` every day at 2 AM** and saves it with a **timestamp**.  

✅ **Step 2: Restore from Backup Using `mongorestore`**
```bash
mongorestore --uri="mongodb://localhost:27017" --drop /backups/mongo-2024-07-10
```
🔹 **`--drop`** → Ensures the database is **dropped and restored cleanly**.  

🚀 **Use Case:** **E-commerce sites, fintech apps, healthcare databases requiring disaster recovery plans.**  

---

# **📌 Summary - MongoDB Replication & High Availability Scenarios (16-20)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **16. Replica Set Failover** | Ensure automatic failover when primary node fails | **Set up a 3-node replica set (`rs.initiate()`)** |
| **17. Read Preference Load Balancing** | Primary node overloaded with read requests | **Use `secondaryPreferred` to distribute read load** |
| **18. Data Durability with Write Concern** | Prevent data loss during crashes | **Use `writeConcern: "majority"` for safe writes** |
| **19. Handling Network Partitions** | Some nodes become unreachable | **Use majority writes & nearest reads to avoid split-brain** |
| **20. Automated Backups & Recovery** | Need regular backups to prevent data loss | **Use `mongodump` with cron jobs & `mongorestore` for recovery** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 21-25 (Sharding & Scaling)**  

These scenarios focus on **sharding implementation, choosing the right shard key, handling uneven shard loads, cross-shard queries, and resharding strategies** in MongoDB.

---

## **2️⃣1️⃣ Scenario: Implementing Sharding for a Large MongoDB Collection**  
🔹 **Problem:** A collection has **billions of documents**, and a **single server cannot handle the load**.  
🔹 **Requirements:**  
- Enable **sharding** to distribute data across multiple servers.  
- Ensure **high availability** and **scalability**.  

### **🔹 Solution: Enable Sharding for the Database and Collection**  

✅ **Step 1: Enable Sharding for the Database**  
```js
sh.enableSharding("ecommerceDB");
```
✅ **Step 2: Choose a Shard Key & Shard the Collection**  
```js
sh.shardCollection("ecommerceDB.orders", { orderId: "hashed" });
```
🔹 **Why Use a Hashed Shard Key?**  
- Ensures **even data distribution** across shards.  
- Prevents **hotspots** (overloading a single shard).  

🚀 **Use Case:** **E-commerce platforms handling millions of orders per day.**  

---

## **2️⃣2️⃣ Scenario: Choosing the Right Shard Key for Balanced Data Distribution**  
🔹 **Problem:** If a **bad shard key** is chosen, one shard may get **more data than others**, causing an **unbalanced cluster**.  
🔹 **Requirements:**  
- Select a **shard key** that ensures **even data distribution**.  
- Avoid **shard hotspots**.  

### **🔹 Solution: Evaluate Different Shard Key Options**  

✅ **Option 1: Bad Shard Key (Leads to Hotspots)**
```js
sh.shardCollection("ecommerceDB.orders", { customerId: 1 });
```
🔹 **Why This Is Bad?**  
- If **some customers place more orders**, their shard gets **overloaded**.  

✅ **Option 2: Better Shard Key (Random Distribution)**
```js
sh.shardCollection("ecommerceDB.orders", { orderId: "hashed" });
```
🔹 **Why This Works?**  
- **Evenly distributes data across all shards.**  

🚀 **Use Case:** **Global SaaS applications with multi-tenant databases.**  

---

## **2️⃣3️⃣ Scenario: Fixing Uneven Shard Load (Shard Imbalance Issue)**  
🔹 **Problem:** One shard has **significantly more data** than others, causing **performance bottlenecks**.  
🔹 **Requirements:**  
- **Rebalance data** to ensure even load.  

### **🔹 Solution: Manually Move Chunks or Reshard Data**  

✅ **Step 1: Check Shard Status**  
```js
sh.status();
```
✅ **Step 2: Move Chunks from an Overloaded Shard**  
```js
sh.moveChunk("ecommerceDB.orders", { orderId: 1000 }, "shard002");
```
✅ **Step 3: Enable Automatic Balancer (If Disabled)**  
```js
sh.startBalancer();
```
🔹 **How This Works?**  
- **Moves chunks from overloaded shards** to less-used shards.  
- **Ensures even data distribution.**  

🚀 **Use Case:** **Databases that grow rapidly and need continuous load balancing.**  

---

## **2️⃣4️⃣ Scenario: Efficient Cross-Shard Queries**  
🔹 **Problem:** Querying data from **multiple shards** causes **performance degradation**.  
🔹 **Requirements:**  
- **Optimize cross-shard queries** to avoid slow response times.  
- Minimize **scatter-gather operations**.  

### **🔹 Solution: Use Targeted Queries Based on Shard Key**  

✅ **Step 1: Avoid Non-Shard Key Queries (Slow)**
```js
db.orders.find({ customerName: "John Doe" });
```
🔹 **Why This is Bad?**  
- **Scans all shards**, making queries slow.  

✅ **Step 2: Use Shard Key in Queries (Fast)**
```js
db.orders.find({ orderId: 1001 });
```
🔹 **Why This is Fast?**  
- **MongoDB directly routes the query** to the correct shard.  

🚀 **Use Case:** **High-performance applications needing real-time analytics.**  

---

## **2️⃣5️⃣ Scenario: Resharding a Collection Without Downtime**  
🔹 **Problem:** The **current shard key is inefficient**, and **resharding is needed**.  
🔹 **Requirements:**  
- Migrate to a **new shard key** without downtime.  

### **🔹 Solution: Migrate Data to a New Sharded Collection**  

✅ **Step 1: Create a New Collection with the Correct Shard Key**  
```js
sh.shardCollection("ecommerceDB.orders_new", { customerId: "hashed" });
```
✅ **Step 2: Migrate Data from the Old Collection**
```js
db.orders.aggregate([{ $merge: { into: "orders_new" } }]);
```
✅ **Step 3: Rename Collections for Seamless Transition**  
```js
db.orders.renameCollection("orders_old");
db.orders_new.renameCollection("orders");
```
🔹 **Why This Works?**  
- **Avoids downtime** by gradually moving data.  
- The **application continues running** without interruptions.  

🚀 **Use Case:** **Any MongoDB-based system needing a schema upgrade in production.**  

---

# **📌 Summary - MongoDB Sharding & Scaling Scenarios (21-25)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **21. Implementing Sharding** | Single server cannot handle large data volumes | **Enable sharding & use hashed shard keys** |
| **22. Choosing the Right Shard Key** | Data is not evenly distributed across shards | **Pick a key with high cardinality & random distribution** |
| **23. Fixing Uneven Shard Load** | One shard has more data than others | **Use `sh.moveChunk()` & enable balancer** |
| **24. Optimizing Cross-Shard Queries** | Queries are slow due to scatter-gather | **Use shard key in queries to target specific shards** |
| **25. Resharding Without Downtime** | Existing shard key is inefficient | **Migrate data to a new collection & rename it** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 26-30 (Migration & Data Handling)**  

These scenarios focus on **migrating data from SQL to MongoDB, handling large dataset migrations, modifying schemas in production, merging collections, and implementing soft deletes**.

---

## **2️⃣6️⃣ Scenario: Migrating Data from SQL to MongoDB While Preserving Relationships**  
🔹 **Problem:** A company is moving from **MySQL to MongoDB**, but they need to keep **relationships between data** (e.g., users and their orders).  
🔹 **Requirements:**  
- Convert **relational tables into MongoDB documents**.  
- Maintain **data integrity** during migration.  
- Optimize schema for **NoSQL design patterns**.  

### **🔹 Solution: Convert Tables into Embedded Documents or References**  

✅ **Step 1: Export SQL Data as JSON**  
```sql
SELECT * FROM users INTO OUTFILE '/path/users.json'
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```
✅ **Step 2: Import Data into MongoDB**  
```bash
mongoimport --db ecommerceDB --collection users --file /path/users.json --jsonArray
```
✅ **Step 3: Optimize Schema (Embed Orders Inside Users)**
```js
db.users.insertOne({
    _id: ObjectId("U123"),
    name: "Alice",
    email: "alice@example.com",
    orders: [
        { orderId: "O101", total: 250.00, status: "Shipped" },
        { orderId: "O102", total: 100.00, status: "Processing" }
    ]
});
```
🔹 **Why This Works?**  
- **Reduces the need for joins** (MongoDB doesn’t support them).  
- **Speeds up queries** by storing related data together.  

🚀 **Use Case:** **Companies migrating from MySQL, PostgreSQL, or Oracle to MongoDB.**  

---

## **2️⃣7️⃣ Scenario: Handling Large Dataset Migration with Minimal Downtime**  
🔹 **Problem:** A production database contains **terabytes of data**, and **migrating it must not cause downtime**.  
🔹 **Requirements:**  
- Migrate data **without blocking reads/writes**.  
- Ensure **data consistency** between old and new collections.  

### **🔹 Solution: Migrate Data in Batches & Use `$merge`**  

✅ **Step 1: Enable Writes to Both Old & New Collections**  
```js
db.new_orders.insertMany(db.old_orders.find().limit(1000).toArray());
```
✅ **Step 2: Automate Incremental Data Migration**  
```js
db.old_orders.aggregate([
    { $match: { migrated: { $exists: false } } },  
    { $merge: { into: "new_orders" } }
]);
```
✅ **Step 3: Rename Collections for a Seamless Transition**  
```js
db.old_orders.renameCollection("old_orders_backup");
db.new_orders.renameCollection("orders");
```
🔹 **Why This Works?**  
- **Minimizes downtime** by migrating data gradually.  
- **Ensures no data loss** by maintaining a backup.  

🚀 **Use Case:** **Migrating to a new schema in a live system.**  

---

## **2️⃣8️⃣ Scenario: Modifying Collection Schema Without Downtime**  
🔹 **Problem:** A live application needs to **add a new field to an existing collection**, but updating all documents at once would slow the system.  
🔹 **Requirements:**  
- Modify the schema **without causing performance issues**.  
- Ensure **backward compatibility**.  

### **🔹 Solution: Use Default Values & Update Incrementally**  

✅ **Step 1: Add a Default Value for New Documents**  
```js
db.users.updateMany({}, { $set: { isVerified: false } });
```
✅ **Step 2: Ensure Old Queries Work With Missing Fields**  
```js
db.users.find({}, { name: 1, email: 1, isVerified: { $ifNull: ["$isVerified", false] } });
```
✅ **Step 3: Modify Application Code to Handle Both Old & New Data**  
```js
const user = db.users.findOne({ email: "john@example.com" });
const isVerified = user.isVerified ?? false;  // Fallback for old documents
```
🔹 **Why This Works?**  
- **No downtime** because MongoDB is schema-less.  
- **Prevents application crashes** due to missing fields.  

🚀 **Use Case:** **Live system upgrades where new fields must be added gradually.**  

---

## **2️⃣9️⃣ Scenario: Merging Data from Multiple Collections into One**  
🔹 **Problem:** A system stores customer data in **separate collections** (`users`, `addresses`, `orders`), but queries require frequent joins.  
🔹 **Requirements:**  
- Merge related documents into a **single optimized collection**.  
- Improve **query performance**.  

### **🔹 Solution: Use Aggregation `$lookup` & `$merge`**  

✅ **Step 1: Merge User & Address Data Using `$lookup`**  
```js
db.users.aggregate([
    {
        $lookup: {
            from: "addresses",
            localField: "_id",
            foreignField: "userId",
            as: "address"
        }
    },
    { $merge: { into: "users_combined" } }
]);
```
✅ **Step 2: Merge Orders Data as an Embedded Array**  
```js
db.users_combined.aggregate([
    {
        $lookup: {
            from: "orders",
            localField: "_id",
            foreignField: "userId",
            as: "orders"
        }
    },
    { $merge: { into: "users_final" } }
]);
```
🔹 **Why This Works?**  
- **Speeds up queries** by reducing cross-collection lookups.  
- **Optimizes storage** by storing related data together.  

🚀 **Use Case:** **E-commerce platforms where user details and order history are frequently queried together.**  

---

## **3️⃣0️⃣ Scenario: Implementing Soft Deletes Instead of Physical Deletes**  
🔹 **Problem:** Deleting documents **permanently removes them**, but the system requires an **undo feature**.  
🔹 **Requirements:**  
- Implement **soft delete** by **marking documents as deleted** instead of removing them.  
- Allow users to **restore deleted records**.  

### **🔹 Solution: Use a `deletedAt` Field Instead of `deleteOne()`**  

✅ **Step 1: Mark Documents as Deleted Instead of Removing Them**  
```js
db.users.updateOne(
    { email: "john@example.com" },
    { $set: { deletedAt: new Date() } }
);
```
✅ **Step 2: Exclude Deleted Documents from Queries**  
```js
db.users.find({ deletedAt: { $exists: false } });
```
✅ **Step 3: Restore a Soft Deleted Document**  
```js
db.users.updateOne(
    { email: "john@example.com" },
    { $unset: { deletedAt: "" } }
);
```
🔹 **Why This Works?**  
- **Preserves deleted data** for recovery.  
- **Prevents accidental loss** of important records.  

🚀 **Use Case:** **CRM systems, SaaS platforms needing data recovery options.**  

---

# **📌 Summary - MongoDB Migration & Data Handling Scenarios (26-30)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **26. SQL to MongoDB Migration** | Convert relational data while keeping relationships | **Use JSON import & embedded documents** |
| **27. Large Dataset Migration** | Move terabytes of data with no downtime | **Migrate in batches, use `$merge` for incremental updates** |
| **28. Schema Modification Without Downtime** | Add new fields to a live collection | **Use default values & update gradually** |
| **29. Merging Multiple Collections** | Queries require frequent joins | **Use `$lookup` & `$merge` to store related data together** |
| **30. Implementing Soft Deletes** | Need to recover deleted records | **Use a `deletedAt` field instead of permanent deletion** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 31-35 (Security & Compliance)**  

These scenarios focus on **securing MongoDB databases, implementing encryption, role-based access control, preventing NoSQL injection, and auditing changes for compliance**.

---

## **3️⃣1️⃣ Scenario: Securing a MongoDB Cluster from Unauthorized Access**  
🔹 **Problem:** The database is **open to the internet**, making it vulnerable to **unauthorized access**.  
🔹 **Requirements:**  
- **Restrict access** to trusted IPs.  
- **Enforce authentication & authorization**.  

### **🔹 Solution: Enable Authentication & IP Whitelisting**  

✅ **Step 1: Enable Authentication in MongoDB Configuration (`mongod.conf`)**  
```yaml
security:
  authorization: enabled
```
✅ **Step 2: Create an Admin User with Access Control**  
```js
use admin;
db.createUser({
    user: "adminUser",
    pwd: "securePassword123",
    roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase"]
});
```
✅ **Step 3: Restrict Access to Trusted IPs (`iptables` Example)**  
```bash
sudo iptables -A INPUT -p tcp --dport 27017 -s 192.168.1.100 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 27017 -j DROP
```
🔹 **Why This Works?**  
- **Authentication ensures only authorized users access the DB**.  
- **IP whitelisting blocks unauthorized external connections**.  

🚀 **Use Case:** **Preventing database breaches in enterprise applications.**  

---

## **3️⃣2️⃣ Scenario: Encrypting Sensitive Data in MongoDB**  
🔹 **Problem:** The database stores **sensitive user data (e.g., passwords, financial details)**, and encryption is required.  
🔹 **Requirements:**  
- Encrypt **data at rest** and **in transit**.  
- Ensure **only authorized applications** can decrypt sensitive fields.  

### **🔹 Solution: Enable TLS/SSL & Use Field-Level Encryption**  

✅ **Step 1: Enable TLS/SSL Encryption for Secure Connections**  
```yaml
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /etc/ssl/mongodb.pem
    CAFile: /etc/ssl/ca.pem
```
✅ **Step 2: Encrypt Specific Fields Using MongoDB Client-Side Field Encryption**  
```js
db.users.insertOne({
    name: "Alice",
    email: "alice@example.com",
    ssn: encrypt("123-45-6789")
});
```
✅ **Step 3: Retrieve Decrypted Data in the Application**  
```js
const decryptedData = decrypt(db.users.findOne({ name: "Alice" }).ssn);
```
🔹 **Why This Works?**  
- **TLS/SSL secures network connections**.  
- **Field-level encryption protects specific sensitive data**.  

🚀 **Use Case:** **Protecting customer data in healthcare & finance industries.**  

---

## **3️⃣3️⃣ Scenario: Implementing Role-Based Access Control (RBAC) in MongoDB**  
🔹 **Problem:** Different users need **different levels of access** (e.g., admins can modify data, analysts can only read).  
🔹 **Requirements:**  
- Restrict database **permissions based on roles**.  
- Prevent unauthorized **data modifications**.  

### **🔹 Solution: Create Role-Based User Permissions**  

✅ **Step 1: Create a Read-Only Role for Analysts**  
```js
use myDatabase;
db.createUser({
    user: "analyst",
    pwd: "readOnlyPass",
    roles: [{ role: "read", db: "myDatabase" }]
});
```
✅ **Step 2: Create a Role for Data Editors**  
```js
db.createUser({
    user: "editor",
    pwd: "editorPass",
    roles: [{ role: "readWrite", db: "myDatabase" }]
});
```
✅ **Step 3: Assign Admin Role for Full Access**  
```js
db.createUser({
    user: "admin",
    pwd: "adminSecurePass",
    roles: ["dbAdmin", "userAdminAnyDatabase"]
});
```
🔹 **Why This Works?**  
- **RBAC ensures users have the minimum required privileges**.  
- **Prevents accidental or malicious modifications by unauthorized users**.  

🚀 **Use Case:** **Multi-tenant SaaS platforms needing strict access control.**  

---

## **3️⃣4️⃣ Scenario: Preventing NoSQL Injection Attacks in MongoDB**  
🔹 **Problem:** A **malicious user tries to manipulate queries** using NoSQL injection techniques.  
🔹 **Requirements:**  
- Prevent attackers from **executing unauthorized queries**.  
- Ensure **user inputs are properly sanitized**.  

### **🔹 Solution: Use Parameterized Queries & Input Validation**  

✅ **Step 1: Never Directly Concatenate User Input in Queries (Bad Example)**  
```js
// 🚨 UNSAFE QUERY (Vulnerable to NoSQL Injection)
db.users.find({ email: req.query.email });
```
✅ **Step 2: Use Parameterized Queries Instead**  
```js
const safeEmail = sanitize(req.query.email);  // Sanitize Input
db.users.find({ email: safeEmail });
```
✅ **Step 3: Validate Input Before Query Execution**  
```js
const schema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required()
});

const { error } = schema.validate(req.body);
if (error) throw new Error("Invalid Input");
```
🔹 **Why This Works?**  
- **Prevents query manipulation by attackers**.  
- **Ensures only valid data is passed to MongoDB**.  

🚀 **Use Case:** **Web applications that handle user authentication & sensitive data.**  

---

## **3️⃣5️⃣ Scenario: Auditing & Logging Changes for Compliance (GDPR, HIPAA, PCI-DSS)**  
🔹 **Problem:** The company needs to **track changes to sensitive data** to comply with **regulations (GDPR, HIPAA, PCI-DSS)**.  
🔹 **Requirements:**  
- Maintain **audit logs** of all data modifications.  
- Track **who made the change, what was changed, and when**.  

### **🔹 Solution: Implement an Audit Trail Using MongoDB Change Streams**  

✅ **Step 1: Enable Change Streams to Capture Data Modifications**  
```js
const changeStream = db.users.watch();
changeStream.on("change", (change) => {
    printjson(change);
});
```
✅ **Step 2: Store Change Logs in an Audit Collection**  
```js
db.auditLogs.insertOne({
    userId: "U123",
    action: "UPDATE",
    fieldChanged: "email",
    oldValue: "old@example.com",
    newValue: "new@example.com",
    modifiedBy: "adminUser",
    modifiedAt: new Date()
});
```
✅ **Step 3: Periodically Archive Audit Logs to Optimize Storage**  
```js
db.auditLogs.aggregate([
    { $match: { modifiedAt: { $lt: new Date("2024-01-01") } } },
    { $merge: { into: "auditLogsArchive" } }
]);
```
🔹 **Why This Works?**  
- **Tracks all data modifications for regulatory compliance**.  
- **Ensures accountability by logging user actions**.  

🚀 **Use Case:** **Organizations in healthcare, finance, and legal industries needing compliance tracking.**  

---

# **📌 Summary - MongoDB Security & Compliance Scenarios (31-35)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **31. Securing MongoDB Cluster** | Prevent unauthorized access | **Enable authentication & IP whitelisting** |
| **32. Encrypting Data** | Protect sensitive information | **Use TLS/SSL & field-level encryption** |
| **33. Role-Based Access Control** | Restrict user privileges | **Create users with specific roles (RBAC)** |
| **34. Preventing NoSQL Injection** | Malicious query manipulation | **Use parameterized queries & input validation** |
| **35. Auditing Data Changes** | Track modifications for compliance | **Use MongoDB Change Streams for audit logs** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 36-40 (Real-Time Data & Analytics)**  

These scenarios focus on **building real-time analytics dashboards, processing IoT data, tracking user activity, handling event-driven processing, and managing time-series data efficiently**.

---

## **3️⃣6️⃣ Scenario: Building a Real-Time Analytics Dashboard**  
🔹 **Problem:** A company needs a **real-time dashboard** that updates instantly with new data (e.g., sales, user sign-ups).  
🔹 **Requirements:**  
- Process **high-frequency data updates**.  
- Display **real-time analytics** with **aggregations**.  

### **🔹 Solution: Use MongoDB Change Streams & Aggregation Pipelines**  

✅ **Step 1: Enable Change Streams to Capture Live Updates**  
```js
const changeStream = db.orders.watch();
changeStream.on("change", (change) => {
    console.log("New order update:", change);
});
```
✅ **Step 2: Aggregate Data for Real-Time Metrics**  
```js
db.orders.aggregate([
    { $match: { status: "Completed" } },
    { $group: { _id: "$productId", totalSales: { $sum: "$amount" } } }
]);
```
✅ **Step 3: Push Data to Frontend Dashboard**  
```js
socket.emit("newData", latestMetrics);
```
🔹 **Why This Works?**  
- **Captures live updates without polling the database**.  
- **Enables real-time analytics** using aggregation.  

🚀 **Use Case:** **Stock market tracking, live sales dashboards, fraud detection systems.**  

---

## **3️⃣7️⃣ Scenario: Storing & Analyzing IoT Sensor Data in Real-Time**  
🔹 **Problem:** A company collects **millions of data points from IoT devices** (e.g., temperature sensors, GPS trackers) every second.  
🔹 **Requirements:**  
- Efficiently **store, process, and query time-series data**.  
- Support **high write throughput**.  

### **🔹 Solution: Use MongoDB Time-Series Collections**  

✅ **Step 1: Create a Time-Series Collection**  
```js
db.createCollection("sensor_data", {
    timeseries: { timeField: "timestamp", metaField: "deviceId", granularity: "seconds" }
});
```
✅ **Step 2: Insert IoT Data in Real-Time**  
```js
db.sensor_data.insertOne({
    deviceId: "sensor_123",
    timestamp: new Date(),
    temperature: 28.5,
    humidity: 65
});
```
✅ **Step 3: Aggregate Data for Trend Analysis**  
```js
db.sensor_data.aggregate([
    { $match: { deviceId: "sensor_123" } },
    { $group: { _id: { $hour: "$timestamp" }, avgTemp: { $avg: "$temperature" } } }
]);
```
🔹 **Why This Works?**  
- **Optimized for time-series data** (reduces storage & improves query performance).  
- **Fast analytics & trend detection**.  

🚀 **Use Case:** **Smart home monitoring, fleet tracking, industrial IoT applications.**  

---

## **3️⃣8️⃣ Scenario: Tracking User Activity & Clickstream Data**  
🔹 **Problem:** A website wants to **analyze user behavior** (e.g., page views, clicks, time spent).  
🔹 **Requirements:**  
- Store **high-frequency user events** efficiently.  
- Generate **behavioral insights** using aggregations.  

### **🔹 Solution: Use Event Logging with TTL Indexes**  

✅ **Step 1: Store Clickstream Data in MongoDB**  
```js
db.user_activity.insertOne({
    userId: "U456",
    event: "page_view",
    page: "/product/123",
    timestamp: new Date()
});
```
✅ **Step 2: Analyze User Behavior with Aggregations**  
```js
db.user_activity.aggregate([
    { $match: { userId: "U456" } },
    { $group: { _id: "$page", visits: { $sum: 1 } } },
    { $sort: { visits: -1 } }
]);
```
✅ **Step 3: Automatically Delete Old Activity Data Using TTL Index**  
```js
db.user_activity.createIndex({ "timestamp": 1 }, { expireAfterSeconds: 2592000 });  // Deletes data after 30 days
```
🔹 **Why This Works?**  
- **Efficiently stores high-frequency event logs**.  
- **Provides real-time user insights**.  

🚀 **Use Case:** **E-commerce recommendation engines, ad targeting, A/B testing platforms.**  

---

## **3️⃣9️⃣ Scenario: Handling Event-Driven Processing in MongoDB**  
🔹 **Problem:** A system needs to **process events asynchronously** (e.g., sending email notifications, updating logs).  
🔹 **Requirements:**  
- Ensure **event-driven workflows** without blocking the main application.  
- Process **events reliably**.  

### **🔹 Solution: Use a Job Queue Collection with Status Updates**  

✅ **Step 1: Store Events in a Collection**  
```js
db.event_queue.insertOne({
    eventType: "send_email",
    userId: "U789",
    email: "user@example.com",
    status: "pending",
    createdAt: new Date()
});
```
✅ **Step 2: Process Events in a Background Worker**  
```js
const event = db.event_queue.findOneAndUpdate(
    { status: "pending" },
    { $set: { status: "processing" } }
);
sendEmail(event.email);
db.event_queue.updateOne({ _id: event._id }, { $set: { status: "completed" } });
```
✅ **Step 3: Periodically Clean Up Old Processed Events**  
```js
db.event_queue.deleteMany({ status: "completed", createdAt: { $lt: new Date(Date.now() - 86400000) } });
```
🔹 **Why This Works?**  
- **Asynchronous processing prevents slowdowns**.  
- **Ensures event execution is tracked properly**.  

🚀 **Use Case:** **Order processing, email notifications, real-time fraud detection.**  

---

## **4️⃣0️⃣ Scenario: Managing Time-Series Data for Performance & Scalability**  
🔹 **Problem:** A company needs to **store and analyze massive time-series data** from logs, metrics, or analytics.  
🔹 **Requirements:**  
- Efficient storage for **timestamped data**.  
- Fast **query performance for trends**.  

### **🔹 Solution: Use Time-Series Collections & Bucketing Strategy**  

✅ **Step 1: Create a Time-Series Collection**  
```js
db.createCollection("logs", {
    timeseries: { timeField: "timestamp", metaField: "service", granularity: "minutes" }
});
```
✅ **Step 2: Store Data in Optimized Buckets**  
```js
db.logs.insertOne({
    service: "auth_service",
    timestamp: new Date(),
    errorCount: 5
});
```
✅ **Step 3: Query Data Efficiently Using Time Ranges**  
```js
db.logs.aggregate([
    { $match: { timestamp: { $gte: ISODate("2024-07-01T00:00:00Z") } } },
    { $group: { _id: { $dayOfMonth: "$timestamp" }, totalErrors: { $sum: "$errorCount" } } }
]);
```
🔹 **Why This Works?**  
- **Optimized storage using MongoDB’s time-series capabilities**.  
- **Fast aggregations for analytics & reporting**.  

🚀 **Use Case:** **Server logs, error tracking, monitoring tools like Prometheus.**  

---

# **📌 Summary - MongoDB Real-Time Data & Analytics Scenarios (36-40)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **36. Real-Time Analytics Dashboard** | Need live updates for sales, sign-ups | **Use Change Streams & Aggregations** |
| **37. IoT Data Processing** | Store high-frequency sensor data efficiently | **Use Time-Series Collections** |
| **38. User Activity Tracking** | Analyze website clicks, user sessions | **Store events & use TTL indexes** |
| **39. Event-Driven Processing** | Handle async tasks like email notifications | **Use a Job Queue Collection** |
| **40. Managing Time-Series Data** | Store logs & metrics with fast queries | **Use time-series collections & bucketing** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 41-45 (Logging & Monitoring)**  

These scenarios focus on **log storage optimization, tracking slow queries, monitoring query performance, handling high write throughput, and setting up automated alerts in MongoDB**.

---

## **4️⃣1️⃣ Scenario: Optimizing Log Storage in MongoDB**  
🔹 **Problem:** The system generates **millions of logs daily**, leading to **high storage costs and slow queries**.  
🔹 **Requirements:**  
- Store logs **efficiently** without affecting query performance.  
- Automatically **delete old logs** after a specific period.  

### **🔹 Solution: Use Capped Collections & TTL Indexes**  

✅ **Step 1: Create a Capped Collection (Fixed-Size Storage)**  
```js
db.createCollection("logs", { capped: true, size: 104857600, max: 500000 });
```
🔹 **Why?**  
- **Prevents unlimited storage growth** by maintaining a fixed size.  

✅ **Step 2: Automatically Expire Old Logs Using TTL Index**  
```js
db.logs.createIndex({ "timestamp": 1 }, { expireAfterSeconds: 604800 });  // Deletes logs older than 7 days
```
🔹 **Why?**  
- **Keeps recent logs while automatically deleting old data.**  

🚀 **Use Case:** **Cloud logging systems, API request logs, application event logs.**  

---

## **4️⃣2️⃣ Scenario: Tracking Slow Queries in MongoDB**  
🔹 **Problem:** Some queries are **taking too long to execute**, slowing down the application.  
🔹 **Requirements:**  
- Identify **which queries are slow**.  
- Optimize **database performance**.  

### **🔹 Solution: Enable MongoDB Profiler & Analyze Query Performance**  

✅ **Step 1: Enable Profiler for Slow Queries (>100ms)**  
```js
db.setProfilingLevel(1, 100);
```
✅ **Step 2: Retrieve Slowest Queries**  
```js
db.system.profile.find().sort({ millis: -1 }).limit(5);
```
✅ **Step 3: Optimize Queries Using Indexing**  
```js
db.orders.createIndex({ customerId: 1, orderDate: 1 });
```
🔹 **Why?**  
- **Detects slow queries & improves performance with indexing**.  

🚀 **Use Case:** **Debugging slow database performance in high-traffic applications.**  

---

## **4️⃣3️⃣ Scenario: Monitoring Query Performance Over Time**  
🔹 **Problem:** The database is **handling millions of queries per hour**, and performance needs **continuous monitoring**.  
🔹 **Requirements:**  
- Track **query execution times**.  
- Identify **performance bottlenecks**.  

### **🔹 Solution: Use MongoDB `serverStatus()` & Query Profiler**  

✅ **Step 1: Get Real-Time Query Performance Stats**  
```js
db.serverStatus().metrics.queryExecutor;
```
✅ **Step 2: Enable Profiler & Monitor Query Patterns**  
```js
db.setProfilingLevel(1);
db.system.profile.find({ op: "query" }).sort({ millis: -1 }).limit(10);
```
✅ **Step 3: Log Query Performance for Future Analysis**  
```js
db.query_logs.insertMany(db.system.profile.find().toArray());
```
🔹 **Why?**  
- **Tracks query execution trends over time** for optimization.  

🚀 **Use Case:** **Database performance tuning for large-scale applications.**  

---

## **4️⃣4️⃣ Scenario: Handling High Write Throughput Without Performance Degradation**  
🔹 **Problem:** The system performs **millions of writes per second**, and performance is **dropping**.  
🔹 **Requirements:**  
- Optimize **MongoDB for high-write workloads**.  
- Ensure **data integrity without write bottlenecks**.  

### **🔹 Solution: Use Write Concern & Reduce Index Overhead**  

✅ **Step 1: Adjust Write Concern for Faster Writes**  
```js
db.orders.insertOne(
    { orderId: 105, customerId: "C789", totalAmount: 450.00 },
    { writeConcern: { w: 1, j: false } }  // Acknowledges write after primary node receives it
);
```
✅ **Step 2: Reduce Indexing to Improve Write Speed**  
```js
db.orders.dropIndex("unnecessary_index");
```
✅ **Step 3: Enable Sharding for Scalability**  
```js
sh.enableSharding("ecommerceDB");
sh.shardCollection("ecommerceDB.orders", { orderId: "hashed" });
```
🔹 **Why?**  
- **Faster writes by avoiding unnecessary journaling & indexing**.  
- **Sharding distributes write load across multiple nodes**.  

🚀 **Use Case:** **Real-time analytics, IoT data ingestion, high-frequency trading.**  

---

## **4️⃣5️⃣ Scenario: Setting Up Automated Alerts for Database Performance Issues**  
🔹 **Problem:** The database experiences **performance issues**, but engineers don’t know until the system crashes.  
🔹 **Requirements:**  
- **Send alerts when performance drops**.  
- Monitor **CPU, memory, & slow queries** automatically.  

### **🔹 Solution: Use MongoDB Atlas Alerts & Monitoring Scripts**  

✅ **Step 1: Enable Alerts in MongoDB Atlas (For Cloud Deployments)**  
```yaml
alerts:
  - condition: "query execution time > 200ms"
    action: "send_email"
    recipients: ["admin@example.com"]
```
✅ **Step 2: Create a Custom Monitoring Script for On-Premise Deployments**  
```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(mongo --eval "db.serverStatus().mem.resident" --quiet)
if [ "$USAGE" -gt "$THRESHOLD" ]; then
  echo "ALERT: MongoDB Memory Usage High ($USAGE%)" | mail -s "MongoDB Alert" admin@example.com
fi
```
✅ **Step 3: Schedule Alerts Using a Cron Job**  
```bash
crontab -e
```
```bash
*/5 * * * * /path/to/mongo_monitor.sh
```
🔹 **Why?**  
- **Automates performance monitoring**.  
- **Ensures proactive issue resolution before failures occur**.  

🚀 **Use Case:** **Database reliability & proactive issue resolution in enterprise environments.**  

---

# **📌 Summary - MongoDB Logging & Monitoring Scenarios (41-45)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **41. Optimizing Log Storage** | Millions of logs increase storage costs | **Use capped collections & TTL indexes** |
| **42. Tracking Slow Queries** | Some queries take too long to execute | **Enable profiler & analyze slow queries** |
| **43. Monitoring Query Performance** | Need to track query execution over time | **Use `serverStatus()` & profile logs** |
| **44. Handling High Write Throughput** | Millions of writes cause slowdowns | **Adjust write concern, reduce indexes, enable sharding** |
| **45. Automated Performance Alerts** | Need alerts for database issues | **Use MongoDB Atlas alerts & monitoring scripts** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 46-50 (Miscellaneous Use Cases)**  

These scenarios focus on **offline data sync, multi-tenant database design, geospatial queries, MongoDB in microservices, and versioning of documents**.

---

## **4️⃣6️⃣ Scenario: Offline Mode with Data Sync in MongoDB**  
🔹 **Problem:** A mobile application needs to **allow users to access and update data offline**, and sync changes when they go online.  
🔹 **Requirements:**  
- Store **local data** on the device.  
- Sync data with the **MongoDB server** once online.  

### **🔹 Solution: Use MongoDB Realm for Offline-First Sync**  

✅ **Step 1: Store Data Locally Using Realm Database (Mobile SDK)**  
```js
const realm = new Realm({
    schema: [{ name: "orders", properties: { _id: "int", status: "string" } }]
});
```
✅ **Step 2: Enable Two-Way Sync with MongoDB Atlas**  
```js
const syncConfig = {
    user: app.currentUser,
    flexibleSync: true
};
```
✅ **Step 3: Merge Offline Changes with the Server When Online**  
```js
realm.write(() => {
    realm.create("orders", { _id: 101, status: "Pending" }, "modified");
});
```
🔹 **Why?**  
- **Ensures smooth user experience** without requiring a constant internet connection.  
- **Syncs data without conflicts** when the user reconnects.  

🚀 **Use Case:** **Mobile apps like note-taking apps, ride-sharing, and fieldwork applications.**  

---

## **4️⃣7️⃣ Scenario: Designing a Multi-Tenant SaaS Database in MongoDB**  
🔹 **Problem:** A SaaS application needs to **store multiple customers' data separately**, ensuring **data isolation and security**.  
🔹 **Requirements:**  
- Each **customer (tenant) must only access their data**.  
- Ensure **scalability** as more tenants join.  

### **🔹 Solution: Use Separate Databases for Each Tenant or Partition by Tenant ID**  

✅ **Approach 1: Separate Databases for Each Tenant (Best for Large-Scale Apps)**  
```js
db.getSiblingDB("tenant_123").users.insertOne({ name: "Alice" });
db.getSiblingDB("tenant_456").users.insertOne({ name: "Bob" });
```
✅ **Approach 2: Single Database with Tenant ID Filtering (Best for Small Apps)**  
```js
db.users.insertOne({ tenantId: "T123", name: "Alice", email: "alice@example.com" });
```
✅ **Ensure Queries are Scoped to the Tenant ID**  
```js
db.users.find({ tenantId: "T123" });
```
🔹 **Why?**  
- **Prevents tenants from accessing each other’s data**.  
- **Scales horizontally** as new tenants are added.  

🚀 **Use Case:** **CRM systems, SaaS accounting software, cloud-based HR platforms.**  

---

## **4️⃣8️⃣ Scenario: Performing Geospatial Queries in MongoDB**  
🔹 **Problem:** A food delivery app needs to **find nearby restaurants within a 5km radius** based on a user’s location.  
🔹 **Requirements:**  
- Store **latitude & longitude** coordinates.  
- Use **geospatial indexing** for fast lookups.  

### **🔹 Solution: Use 2dsphere Index for Geospatial Queries**  

✅ **Step 1: Store Locations with Coordinates**  
```js
db.restaurants.insertOne({
    name: "Pizza Hub",
    location: { type: "Point", coordinates: [-73.935242, 40.730610] }
});
```
✅ **Step 2: Create a Geospatial Index**  
```js
db.restaurants.createIndex({ location: "2dsphere" });
```
✅ **Step 3: Find Restaurants Near a User’s Location**  
```js
db.restaurants.find({
    location: {
        $near: {
            $geometry: { type: "Point", coordinates: [-73.935242, 40.730610] },
            $maxDistance: 5000  // 5km radius
        }
    }
});
```
🔹 **Why?**  
- **Fast lookup of nearby locations**.  
- **Efficient indexing for maps & navigation apps**.  

🚀 **Use Case:** **Uber Eats, Google Maps, location-based services.**  

---

## **4️⃣9️⃣ Scenario: Using MongoDB in a Microservices Architecture**  
🔹 **Problem:** A company is **building microservices**, and each service needs its own database while maintaining data consistency.  
🔹 **Requirements:**  
- Each microservice must **own its data** independently.  
- Services should **communicate asynchronously**.  

### **🔹 Solution: Use a Database per Microservice & Event-Driven Architecture**  

✅ **Step 1: Assign Each Microservice Its Own MongoDB Database**  
```js
// Order Service DB
db.orders.insertOne({ orderId: 201, userId: "U456", status: "Shipped" });

// User Service DB
db.users.insertOne({ userId: "U456", name: "Alice" });
```
✅ **Step 2: Use Change Streams for Event-Driven Communication**  
```js
const changeStream = db.orders.watch();
changeStream.on("change", (change) => {
    console.log("Order updated:", change);
});
```
✅ **Step 3: Publish Events to a Message Queue (Kafka, RabbitMQ)**  
```js
produceMessage("order.shipped", { orderId: 201, userId: "U456" });
```
🔹 **Why?**  
- **Prevents direct coupling between microservices**.  
- **Enables real-time updates across services**.  

🚀 **Use Case:** **E-commerce platforms, cloud-native applications, API-based architectures.**  

---

## **5️⃣0️⃣ Scenario: Implementing Versioning in MongoDB for Document History Tracking**  
🔹 **Problem:** A system needs to **track changes made to documents**, keeping a **history of previous versions**.  
🔹 **Requirements:**  
- Maintain **multiple versions** of documents.  
- Allow **rollback to previous versions** if needed.  

### **🔹 Solution: Store Versioned Documents in an Array or Separate Collection**  

✅ **Approach 1: Store Previous Versions in an Embedded Array**  
```js
db.articles.insertOne({
    _id: ObjectId("A101"),
    title: "MongoDB Best Practices",
    content: "Original content...",
    versions: [
        { version: 1, content: "Original content...", modifiedAt: ISODate("2024-07-01T10:00:00Z") }
    ]
});
```
✅ **Approach 2: Store Versions in a Separate Collection**  
```js
db.article_versions.insertOne({
    articleId: ObjectId("A101"),
    version: 2,
    content: "Updated content...",
    modifiedAt: new Date()
});
```
✅ **Step 3: Retrieve the Latest Version**  
```js
db.article_versions.find({ articleId: "A101" }).sort({ version: -1 }).limit(1);
```
🔹 **Why?**  
- **Allows restoring previous versions** in case of accidental changes.  
- **Efficiently tracks document modifications over time**.  

🚀 **Use Case:** **Wikis, document management systems, legal contract tracking.**  

---

# **📌 Summary - MongoDB Miscellaneous Use Cases (46-50)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **46. Offline Sync** | Users need to access & update data offline | **Use MongoDB Realm for automatic sync** |
| **47. Multi-Tenant SaaS** | Isolate customer data in a shared database | **Use separate DBs or partition by tenant ID** |
| **48. Geospatial Queries** | Find nearby locations within a certain radius | **Use `2dsphere` index for fast location lookups** |
| **49. MongoDB in Microservices** | Each service needs independent data management | **Use a DB per service & event-driven architecture** |
| **50. Document Versioning** | Track changes and restore previous document versions | **Store previous versions in an array or separate collection** |

<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 51-55 (Advanced Use Cases)**  

These scenarios focus on **full-text search, data deduplication, handling large aggregations, combining relational & NoSQL data, and implementing CQRS in MongoDB**.

---

## **5️⃣1️⃣ Scenario: Implementing Full-Text Search in MongoDB**  
🔹 **Problem:** A company needs a **search feature** to allow users to find products by name, description, and keywords.  
🔹 **Requirements:**  
- Support **fast, flexible keyword searches**.  
- Allow **fuzzy matching, stemming, and ranking**.  

### **🔹 Solution: Use MongoDB’s Full-Text Search Index**  

✅ **Step 1: Create a Text Index on Relevant Fields**  
```js
db.products.createIndex({ name: "text", description: "text" });
```
✅ **Step 2: Perform a Text Search Query**  
```js
db.products.find({ $text: { $search: "wireless headphones" } });
```
✅ **Step 3: Rank Results by Relevance Using `$meta`**  
```js
db.products.find(
    { $text: { $search: "wireless headphones" } },
    { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });
```
🔹 **Why?**  
- **Optimized for full-text search without external search engines**.  
- **Supports relevance ranking & natural language search**.  

🚀 **Use Case:** **E-commerce search, document search, knowledge bases.**  

---

## **5️⃣2️⃣ Scenario: Handling Data Deduplication in MongoDB**  
🔹 **Problem:** A system is storing **duplicate records**, leading to **redundant data and increased storage costs**.  
🔹 **Requirements:**  
- Identify and **remove duplicate records**.  
- Prevent **future duplicates**.  

### **🔹 Solution: Use Unique Indexing & Aggregation to Find Duplicates**  

✅ **Step 1: Prevent Duplicates by Enforcing Unique Indexes**  
```js
db.users.createIndex({ email: 1 }, { unique: true });
```
✅ **Step 2: Identify Duplicate Entries**  
```js
db.users.aggregate([
    { $group: { _id: "$email", count: { $sum: 1 }, docs: { $push: "$_id" } } },
    { $match: { count: { $gt: 1 } } }
]);
```
✅ **Step 3: Remove Duplicate Records (Keep One Copy)**  
```js
db.users.deleteMany({ _id: { $in: duplicateIds.slice(1) } });
```
🔹 **Why?**  
- **Reduces storage costs & ensures data consistency**.  
- **Prevents duplicate records from being inserted**.  

🚀 **Use Case:** **User account management, CRM systems, customer databases.**  

---

## **5️⃣3️⃣ Scenario: Optimizing Large Aggregations for Fast Analytics**  
🔹 **Problem:** A reporting system needs to **process millions of records** but aggregation queries are **slow**.  
🔹 **Requirements:**  
- Ensure **fast aggregations** without affecting live traffic.  

### **🔹 Solution: Use Aggregation Pipelines with Pre-Aggregated Data**  

✅ **Step 1: Pre-Aggregate Data into a Summary Collection**  
```js
db.orders.aggregate([
    { $match: { status: "Completed" } },
    { $group: { _id: "$productId", totalSales: { $sum: "$amount" } } },
    { $merge: { into: "order_summary" } }
]);
```
✅ **Step 2: Query Pre-Aggregated Data Instead of Raw Orders**  
```js
db.order_summary.find();
```
✅ **Step 3: Schedule Aggregation Jobs Periodically**  
```bash
crontab -e
```
```bash
0 * * * * mongo --eval "db.runCommand({ aggregate: 'orders', pipeline: [...], cursor: {} })"
```
🔹 **Why?**  
- **Reduces query execution time by precomputing results**.  
- **Improves performance for analytics dashboards**.  

🚀 **Use Case:** **Financial reporting, inventory analysis, business intelligence.**  

---

## **5️⃣4️⃣ Scenario: Combining Relational & NoSQL Data in MongoDB**  
🔹 **Problem:** A system needs to **store relational data (like SQL)** while benefiting from **MongoDB’s flexibility**.  
🔹 **Requirements:**  
- Use **references** instead of embedded documents for normalization.  

### **🔹 Solution: Use References (`$lookup`) for Relational-Like Queries**  

✅ **Step 1: Store Data in Separate Collections (Normalized Approach)**  
```js
db.customers.insertOne({ _id: "C101", name: "Alice" });
db.orders.insertOne({ _id: "O201", customerId: "C101", amount: 200 });
```
✅ **Step 2: Query Using `$lookup` to Join Data**  
```js
db.orders.aggregate([
    { $lookup: { from: "customers", localField: "customerId", foreignField: "_id", as: "customer" } }
]);
```
🔹 **Why?**  
- **Combines relational & NoSQL approaches** for flexibility.  
- **Maintains normalized structure while supporting JSON flexibility**.  

🚀 **Use Case:** **Migrations from SQL to MongoDB, hybrid database architectures.**  

---

## **5️⃣5️⃣ Scenario: Implementing CQRS (Command Query Responsibility Segregation) in MongoDB**  
🔹 **Problem:** A system needs to **separate read & write operations** for better performance and scalability.  
🔹 **Requirements:**  
- Use **separate databases for reads & writes**.  
- Ensure **data consistency across systems**.  

### **🔹 Solution: Use MongoDB Replication & Read Preference for CQRS**  

✅ **Step 1: Use a Primary Database for Writes**  
```js
db.orders.insertOne(
    { orderId: 105, customerId: "C789", totalAmount: 450.00 }
);
```
✅ **Step 2: Route Read Queries to Secondary Replicas**  
```js
db.getMongo().setReadPref("secondaryPreferred");
```
✅ **Step 3: Use Change Streams to Sync Read & Write Databases**  
```js
const changeStream = db.orders.watch();
changeStream.on("change", (change) => {
    db.order_reports.insertOne(change.fullDocument);
});
```
🔹 **Why?**  
- **Prevents read operations from slowing down writes**.  
- **Scales better for high-traffic applications**.  

🚀 **Use Case:** **E-commerce order management, real-time analytics, high-scale web apps.**  

---

# **📌 Summary - Advanced MongoDB Use Cases (51-55)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **51. Full-Text Search** | Users need a powerful search feature | **Use MongoDB text indexes** |
| **52. Data Deduplication** | Avoid storing duplicate data | **Use unique indexes & aggregation to detect duplicates** |
| **53. Large Aggregations** | Reporting queries are slow | **Pre-aggregate data in summary collections** |
| **54. Combining Relational & NoSQL Data** | Store normalized data in MongoDB | **Use `$lookup` for relational-like queries** |
| **55. CQRS for Scalability** | Separate read & write operations | **Use MongoDB replication & read preferences** |


<br/>
<br/>

# **📌 Scenario-Based MongoDB Questions 56-60 (Scaling & Performance Optimization)**  

These scenarios focus on **high-scale MongoDB deployments, hybrid indexing strategies, cache optimization, reducing network latency, and bulk processing for big data applications**.

---

## **5️⃣6️⃣ Scenario: Scaling MongoDB for High-Traffic Applications**  
🔹 **Problem:** A **high-traffic** application (e.g., a social media platform) is experiencing **performance issues** due to millions of concurrent users.  
🔹 **Requirements:**  
- Scale **horizontally** to handle increasing load.  
- Ensure **data availability & fault tolerance**.  

### **🔹 Solution: Implement Sharding & Load Balancing**  

✅ **Step 1: Enable Sharding on the Database**  
```js
sh.enableSharding("socialDB");
```
✅ **Step 2: Choose an Optimal Shard Key for Even Distribution**  
```js
sh.shardCollection("socialDB.posts", { userId: "hashed" });
```
✅ **Step 3: Use Multiple Query Routers (`mongos`) to Distribute Traffic**  
```bash
mongos --configdb configReplicaSet/host1:27019,host2:27019,host3:27019
```
🔹 **Why?**  
- **Sharding distributes traffic across multiple servers**.  
- **Ensures high availability & automatic failover**.  

🚀 **Use Case:** **Facebook, Twitter, LinkedIn-scale social media platforms.**  

---

## **5️⃣7️⃣ Scenario: Hybrid Indexing Strategy for Optimized Query Performance**  
🔹 **Problem:** Queries on **large collections** (millions of documents) are slow despite using **single-field indexes**.  
🔹 **Requirements:**  
- Improve query performance using **optimized indexing strategies**.  
- Reduce **index size** for efficiency.  

### **🔹 Solution: Use Compound, Partial, and Sparse Indexes**  

✅ **Step 1: Create a Compound Index for Multi-Field Queries**  
```js
db.orders.createIndex({ customerId: 1, orderDate: -1 });
```
✅ **Step 2: Use a Partial Index to Index Only Relevant Documents**  
```js
db.orders.createIndex({ status: 1 }, { partialFilterExpression: { status: "Completed" } });
```
✅ **Step 3: Use Sparse Indexes for Fields That Aren’t Always Present**  
```js
db.users.createIndex({ email: 1 }, { sparse: true });
```
🔹 **Why?**  
- **Compound indexes speed up complex queries**.  
- **Partial indexes reduce index size** by excluding unnecessary documents.  

🚀 **Use Case:** **E-commerce order history, banking transactions, data-heavy applications.**  

---

## **5️⃣8️⃣ Scenario: Using Redis with MongoDB for Caching & Performance Optimization**  
🔹 **Problem:** A **high-traffic web app** queries MongoDB frequently, causing **latency issues**.  
🔹 **Requirements:**  
- Reduce **database load** by caching frequent queries.  
- Improve **application response time**.  

### **🔹 Solution: Implement Redis as a Cache Layer Before MongoDB**  

✅ **Step 1: Check Cache Before Querying MongoDB**  
```js
const cachedData = redisClient.get("recentOrders");
if (cachedData) return JSON.parse(cachedData);
```
✅ **Step 2: Query MongoDB If Cache Misses & Store Result in Redis**  
```js
const orders = db.orders.find({ userId: "U123" }).toArray();
redisClient.setex("recentOrders", 3600, JSON.stringify(orders));  // Cache for 1 hour
```
🔹 **Why?**  
- **Speeds up repeated queries**.  
- **Reduces MongoDB read load** for better performance.  

🚀 **Use Case:** **Real-time leaderboards, social media feeds, personalized recommendations.**  

---

## **5️⃣9️⃣ Scenario: Reducing Network Latency in a Distributed MongoDB Deployment**  
🔹 **Problem:** A globally distributed MongoDB cluster is experiencing **high query response times** due to **network delays**.  
🔹 **Requirements:**  
- Reduce **latency for geographically dispersed users**.  
- Improve **data retrieval speed**.  

### **🔹 Solution: Use Read Preferences & Geographically Distributed Replica Sets**  

✅ **Step 1: Deploy Replica Sets Across Multiple Data Centers**  
```yaml
replication:
  replSetName: "globalCluster"
```
✅ **Step 2: Route Read Queries to the Nearest Replica**  
```js
db.getMongo().setReadPref("nearest");
```
✅ **Step 3: Use Read-Only Secondary Nodes for Fast Regional Access**  
```js
db.replicaset.status();
```
🔹 **Why?**  
- **Users in different regions query the nearest data center**.  
- **Read queries are faster without affecting the primary node**.  

🚀 **Use Case:** **Multi-region applications like Netflix, Airbnb, global SaaS platforms.**  

---

## **6️⃣0️⃣ Scenario: Handling Bulk Data Processing in MongoDB for Big Data Applications**  
🔹 **Problem:** A system processes **millions of records daily**, and bulk insertions/updating are **causing slow performance**.  
🔹 **Requirements:**  
- Efficiently **insert/update large datasets**.  
- Reduce **indexing overhead during batch operations**.  

### **🔹 Solution: Use `bulkWrite()` & Disable Indexing Temporarily**  

✅ **Step 1: Use Bulk Inserts Instead of Multiple `insertOne()` Calls**  
```js
db.orders.bulkWrite([
    { insertOne: { document: { orderId: 1, status: "Completed" } } },
    { insertOne: { document: { orderId: 2, status: "Processing" } } }
]);
```
✅ **Step 2: Disable Indexing Temporarily to Speed Up Inserts**  
```js
db.orders.dropIndex("status_1");
db.orders.bulkWrite([...]);  // Perform Bulk Insert
db.orders.createIndex({ status: 1 });
```
✅ **Step 3: Process Large Updates in Chunks**  
```js
let batchSize = 1000;
let cursor = db.orders.find().batchSize(batchSize);
cursor.forEach((doc) => {
    db.orders.updateOne({ _id: doc._id }, { $set: { processed: true } });
});
```
🔹 **Why?**  
- **Bulk operations optimize batch inserts/updates**.  
- **Temporarily disabling indexing speeds up writes**.  

🚀 **Use Case:** **Big data ETL pipelines, large-scale log processing, analytics dashboards.**  

---

# **📌 Summary - MongoDB Scaling & Performance Optimization Scenarios (56-60)**  

| **Scenario** | **Problem** | **Solution** |
|-------------|------------|-------------|
| **56. Scaling for High Traffic** | Millions of users causing slow performance | **Enable sharding, distribute traffic with `mongos`** |
| **57. Hybrid Indexing** | Queries are slow despite indexing | **Use compound, partial, and sparse indexes** |
| **58. MongoDB + Redis Caching** | High read load slowing down queries | **Cache frequent queries with Redis** |
| **59. Reducing Network Latency** | Global users experience high response times | **Use geographically distributed replica sets & read preferences** |
| **60. Bulk Data Processing** | Large datasets causing slow inserts & updates | **Use `bulkWrite()`, process data in chunks, disable indexing temporarily** |
