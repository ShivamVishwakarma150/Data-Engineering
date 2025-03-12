# **📌 Explanation of Python to Mongo DB in Detail**

This Python script uses **PyMongo** to connect to a **MongoDB database** and perform **queries and aggregations** on an `airbnb_property_reviews` collection. Below is a **line-by-line explanation** of the script.

---

## **1️⃣ Importing Required Libraries**
```python
from pymongo import MongoClient
```
- **`pymongo`**: This is the official **MongoDB driver for Python**.
- **`MongoClient`**: A class used to **establish a connection** to the MongoDB database.

---

## **2️⃣ MongoDB Connection String**
```python
conn_string = "mongodb+srv://<username>:<password>@mongo-db-cluster-new.6wvma1w.mongodb.net/?retryWrites=true&w=majority&appName=mongo-db-cluster-new"
```
- **`mongodb+srv://...`** → Connection string to a **MongoDB Atlas (cloud-based MongoDB cluster)**.
- **`<username>` and `<password>`** → Should be replaced with **actual credentials**.
- **`mongo-db-cluster-new.6wvma1w.mongodb.net`** → **Host address** of the MongoDB cluster.
- **`retryWrites=true&w=majority`** → Ensures **automatic retries** for failed write operations.

---

## **3️⃣ Connecting to MongoDB**
```python
client = MongoClient(conn_string)
```
- Creates a **MongoDB client instance** to establish a **connection** to the database.

---

## **4️⃣ Selecting a Database and Collection**
```python
db = client['airbnb_mart']
collection = db['airbnb_property_reviews']
```
- **`db = client['airbnb_mart']`** → Selects the database **`airbnb_mart`**.
- **`collection = db['airbnb_property_reviews']`** → Selects the collection **`airbnb_property_reviews`**.

---

## **5️⃣ Inserting a Document (Commented Out)**
```python
# insert_result = collection.insert_one({'name': 'John Doe', 'age': 30})
# print(f"Document inserted with id: {insert_result.inserted_id}")
```
- **`insert_one()`** → Inserts a **single document** into the collection.
- **`inserted_id`** → Returns the **ID of the inserted document**.
- **Commented out** to prevent inserting test data during execution.

✅ **Example Output (if enabled):**
```
Document inserted with id: 650c3c8e12f3f6a19f92e2a3
```

---

## **6️⃣ Querying the Collection (Commented Out)**
Several queries are defined to **filter documents** based on conditions.

### **🔹 Query 1: Find All Private Room Apartments**
```python
# find_query = {'property_type' : 'Apartment', 'room_type': 'Private room'}
```
- Retrieves all documents where:
  - **`property_type`** is `"Apartment"`.
  - **`room_type`** is `"Private room"`.

### **🔹 Query 2: Find Houses or Apartments**
```python
# find_query = { '$or' : [ {'property_type' : 'House'} , {'property_type' : 'Apartment'} ] }
```
- Uses **`$or`** to find properties that are **either "House" or "Apartment"**.

### **🔹 Query 3: Find Private Rooms in Houses or Apartments**
```python
# find_query = { 'room_type': 'Private room', 
#                '$or' : [ {'property_type' : 'House'} , {'property_type' : 'Apartment'} ] 
#             }
```
- Filters properties where:
  - **`room_type = "Private room"`**.
  - **Property type is either "House" or "Apartment"**.

### **🔹 Query 4: Find Properties That Accommodate More Than 2 People**
```python
# find_query = { 'accommodates': { '$gt' : 2} }
```
- Uses **`$gt` (greater than)** to find properties that **accommodate more than 2 people**.

---

## **7️⃣ Counting Documents (Commented Out)**
```python
# count_results = collection.count_documents(find_query)
# print("Total Documents Found : ", count_results)
```
- **`count_documents(find_query)`** → Counts the number of documents matching the query.

✅ **Example Output (if enabled):**
```
Total Documents Found : 450
```

---

## **8️⃣ Fetching & Printing Documents (Commented Out)**
```python
# find_results = collection.find(find_query)
# for doc in find_results:
#     print(doc)
```
- **`collection.find(find_query)`** → Fetches all matching documents.
- **Loop prints each document**.

✅ **Example Output (if enabled):**
```json
{
    "_id": ObjectId("650c3c8e12f3f6a19f92e2a3"),
    "property_type": "Apartment",
    "room_type": "Private room",
    "accommodates": 4,
    "price": 120,
    "address": { "city": "New York", "country": "USA" }
}
```

---

## **9️⃣ Aggregation Query: Average Price by Country & City**
```python
grp2 = [
    {
        "$group": {
            "_id": {
                "country": "$address.country",
                "city": "$address.suburb"
            },
            "avg_price": {"$avg": "$price"}
        }
    },
    {
        "$project": {
            "country": "$_id.country",
            "city": "$_id.city",
            "avg_price": {"$toDouble": "$avg_price"},
            "_id": 0
        }
    }
]
```
### **🔹 Explanation:**
1. **`$group` Stage**
   - Groups documents by **country** and **city**.
   - Computes the **average price (`$avg`)** of properties in each city.

2. **`$project` Stage**
   - Extracts fields:
     - **Country** → `$_id.country`
     - **City** → `$_id.city`
     - **Average Price** (converted to double) → `"$toDouble": "$avg_price"`
   - **Hides `_id` field** (`"_id": 0`).

---

## **🔟 Executing the Aggregation Query**
```python
results = collection.aggregate(grp2)
for result in results:
    print(result)
```
- **Runs the aggregation pipeline**.
- **Iterates through the results** and prints them.

✅ **Example Output:**
```json
{ "country": "USA", "city": "New York", "avg_price": 150.25 }
{ "country": "France", "city": "Paris", "avg_price": 180.75 }
{ "country": "India", "city": "Mumbai", "avg_price": 75.50 }
```

---

## **🔚 Closing the Connection**
```python
client.close()
```
- **Closes the connection** to free resources.

---

# **📌 Summary - What Does This Script Do?**
| **Step** | **Action** |
|----------|-----------|
| **1️⃣** | Connects to MongoDB Atlas using `MongoClient` |
| **2️⃣** | Selects the `airbnb_mart` database and `airbnb_property_reviews` collection |
| **3️⃣** | (Commented) Inserts a sample document |
| **4️⃣** | (Commented) Defines multiple **find queries** |
| **5️⃣** | (Commented) Counts matching documents |
| **6️⃣** | (Commented) Fetches and prints documents |
| **7️⃣** | **Runs an aggregation query** to compute **average price per country & city** |
| **8️⃣** | Prints **aggregated results** |
| **9️⃣** | Closes the MongoDB connection |

