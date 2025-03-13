# Cassandra_Demo.ipynb
Here's a detailed explanation of the code from the **Cassandra_Demo.ipynb** file:

---

### **1. Basic Print Statement**
```python
print("Hello World !!")
```
- This simply prints `"Hello World !!"` to the console to ensure the script is running.

---

### **2. Installing the Cassandra Driver**
```python
pip install cassandra-driver
```
- This command installs the `cassandra-driver`, which is the official Python client for Apache Cassandra.

---

### **3. Importing the Cassandra Library**
```python
import cassandra
print (cassandra.__version__)
```
- The `cassandra` module is imported, and its version is printed to verify that the driver is installed correctly.

---

### **4. Connecting to Cassandra Cluster**
```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

cloud_config= {
  'secure_connect_bundle': 'secure-connect-cassandra-demo.zip'
}
auth_provider = PlainTextAuthProvider('PumerferfDjZnBWbbCLxpPZw', 'QS4mrxNshcjGJGHDDGJSikjsiksaegNDgqTIuhYUeB-L,pNtR1m7tj9uTdMx+vOdXO0.7xzyD23BK+NXftsmeh-ofLFXrx6siefdsD-cdfdsfNZIrl,sdfsP9xcYe_')
cluster = Cluster(cloud=cloud_config, auth_provider=auth_provider)
session = cluster.connect()
```
- **`Cluster` and `PlainTextAuthProvider`** are imported from the `cassandra.cluster` module.
- **`secure_connect_bundle`**: This is a special bundle for connecting securely to a cloud-based Cassandra database.
- **`PlainTextAuthProvider`**: This authenticates the connection using a username and password.
- **`Cluster(cloud=cloud_config, auth_provider=auth_provider)`**: Creates a connection to the Cassandra cluster.
- **`session = cluster.connect()`**: Establishes a session for executing queries.

---

### **5. Checking Cassandra Version**
```python
row = session.execute("select release_version from system.local").one()
if row:
    print(row[0])
else:
    print("An error occurred.")
```
- This query retrieves the **Cassandra version** from the system.
- If the result exists, it prints the release version; otherwise, it prints an error message.

---

### **6. Using a Keyspace**
```python
try:
    query = "use employee_keyspace"
    session.execute(query)
    print("Inside the employee_keyspace")
except Exception as err:
    print("Exception Occurred while using Keyspace : ", err)
```
- **Keyspace**: A namespace that defines data replication and placement strategies.
- The script tries to switch to the keyspace named **`employee_keyspace`**.
- If the keyspace does not exist or an error occurs, it prints an error message.

---


### **7. Creating a Keyspace**
```python
try:
    query = """CREATE KEYSPACE IF NOT EXISTS employee_keyspace
               WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3}"""
    session.execute(query)
    print("Keyspace employee_keyspace created successfully")
except Exception as err:
    print("Exception Occurred while creating Keyspace : ", err)
```
#### **Explanation:**
- **`CREATE KEYSPACE IF NOT EXISTS employee_keyspace`**: Creates a keyspace named `employee_keyspace` if it doesn't already exist.
- **`WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3}`**:
  - `SimpleStrategy`: A basic replication strategy suitable for a single data center.
  - `replication_factor = 3`: Each piece of data is stored in **three different nodes**.
- If successful, it prints `"Keyspace employee_keyspace created successfully"`, otherwise, it prints the error.

---

### **8. Switching to the Keyspace**
```python
try:
    query = "USE employee_keyspace"
    session.execute(query)
    print("Inside the employee_keyspace")
except Exception as err:
    print("Exception Occurred while using Keyspace : ", err)
```
#### **Explanation:**
- The `USE employee_keyspace` statement switches the session to use `employee_keyspace` for further operations.
- If the keyspace does not exist or an error occurs, it prints an error message.

---

### **9. Creating an Employee Table**
```python
try:
    query = """CREATE TABLE IF NOT EXISTS employees (
               department_id int,
               office_id int,
               employee_id int,
               first_name text,
               last_name text,
               email text,
               PRIMARY KEY ((department_id, office_id), employee_id, last_name)
               )"""
    session.execute(query)
    print("Table employees created successfully")
except Exception as err:
    print("Exception Occurred while creating Table : ", err)
```
#### **Explanation:**
- **`CREATE TABLE IF NOT EXISTS employees`**: Creates a table named `employees` if it doesn't already exist.
- **Columns**:
  - `department_id`, `office_id`, `employee_id`: Integer type.
  - `first_name`, `last_name`, `email`: Text type.
- **Primary Key Definition**:
  - **Partition Key**: `(department_id, office_id)` - This determines how data is distributed across nodes.
  - **Clustering Keys**: `employee_id, last_name` - These help in sorting the data within a partition.
- If successful, it prints `"Table employees created successfully"`, otherwise, it prints the error.

---

### **10. Inserting Data into the Employees Table**
```python
try:
    query = """INSERT INTO employees (department_id, office_id, employee_id, first_name, last_name, email)
               VALUES (1, 101, 5001, 'John', 'Doe', 'john.doe@example.com')"""
    session.execute(query)
    print("Data inserted successfully")
except Exception as err:
    print("Exception Occurred while inserting Data : ", err)
```
#### **Explanation:**
- **`INSERT INTO employees`**: Inserts a row into the `employees` table.
- **Values**:
  - `department_id = 1`
  - `office_id = 101`
  - `employee_id = 5001`
  - `first_name = 'John'`
  - `last_name = 'Doe'`
  - `email = 'john.doe@example.com'`
- If successful, it prints `"Data inserted successfully"`.

---

### **11. Querying Data from Employees Table**
```python
try:
    query = "SELECT * FROM employees"
    rows = session.execute(query)
    for row in rows:
        print(row)
except Exception as err:
    print("Exception Occurred while fetching Data : ", err)
```
#### **Explanation:**
- **`SELECT * FROM employees`**: Retrieves all rows from the `employees` table.
- **Iterating Over Rows**:
  - `for row in rows:`: Iterates through each row in the result.
  - `print(row)`: Prints the row data.
- If an error occurs, it prints an error message.

---

### **12. Updating Data in the Employees Table**
```python
try:
    query = """UPDATE employees
               SET email = 'johndoe@example.com'
               WHERE department_id = 1 AND office_id = 101 AND employee_id = 5001 AND last_name = 'Doe'"""
    session.execute(query)
    print("Data updated successfully")
except Exception as err:
    print("Exception Occurred while updating Data : ", err)
```
#### **Explanation:**
- **`UPDATE employees`**: Modifies an existing row.
- **Setting New Value**:
  - Changes `email` to `'johndoe@example.com'`.
- **WHERE Clause**:
  - Filters for `department_id = 1`, `office_id = 101`, `employee_id = 5001`, and `last_name = 'Doe'` to identify the row to update.
- If successful, it prints `"Data updated successfully"`.

---

### **13. Deleting Data from Employees Table**
```python
try:
    query = """DELETE FROM employees
               WHERE department_id = 1 AND office_id = 101 AND employee_id = 5001 AND last_name = 'Doe'"""
    session.execute(query)
    print("Data deleted successfully")
except Exception as err:
    print("Exception Occurred while deleting Data : ", err)
```
#### **Explanation:**
- **`DELETE FROM employees`**: Deletes a specific row.
- **WHERE Clause**:
  - Ensures that only the record matching `department_id = 1`, `office_id = 101`, `employee_id = 5001`, and `last_name = 'Doe'` is deleted.
- If successful, it prints `"Data deleted successfully"`.

---

### **14. Closing the Cassandra Session**
```python
session.shutdown()
cluster.shutdown()
print("Cassandra session closed successfully")
```
#### **Explanation:**
- **`session.shutdown()`**: Closes the current session.
- **`cluster.shutdown()`**: Closes the connection to the cluster.
- Prints `"Cassandra session closed successfully"` to indicate that the script has completed execution.

---

### **Summary of the Cassandra Script**
1. **Connects to a Cassandra cluster** using secure authentication.
2. **Creates a keyspace** (`employee_keyspace`) if it doesn’t exist.
3. **Switches to the keyspace** for further operations.
4. **Creates a table** (`employees`) with a composite primary key.
5. **Performs CRUD operations**:
   - Inserts a row.
   - Retrieves all rows.
   - Updates a row.
   - Deletes a row.
6. **Closes the session** after execution.

This script follows a structured approach for managing employee data in Cassandra.

---


✅ **Connecting to Cassandra**  
✅ **Creating a Keyspace**  
✅ **Using the Keyspace**  
✅ **Creating a Table**  
✅ **Inserting Data**  
✅ **Querying Data**  
✅ **Updating Data**  
✅ **Deleting Data**  
✅ **Closing the Session**  

However, if you want to **extend** this, you can explore:  

### **1. Batch Operations**  
If you want to perform multiple operations atomically:  
```python
try:
    batch = BatchStatement()
    batch.add("INSERT INTO employees (department_id, office_id, employee_id, first_name, last_name, email) VALUES (2, 102, 5002, 'Alice', 'Smith', 'alice.smith@example.com')")
    batch.add("INSERT INTO employees (department_id, office_id, employee_id, first_name, last_name, email) VALUES (3, 103, 5003, 'Bob', 'Brown', 'bob.brown@example.com')")
    
    session.execute(batch)
    print("Batch Insert Successful")
except Exception as err:
    print("Error in Batch Operation:", err)
```
This ensures **atomic execution** of multiple queries.

---

### **2. Indexing for Faster Queries**  
If you frequently query by `email`, you can create a secondary index:  
```python
session.execute("CREATE INDEX ON employees(email)")
```
But **Cassandra indexes are not always efficient**, so use them wisely!

---

### **3. Materialized Views (For Query Optimization)**  
If you often **query by last_name**, you can create a **Materialized View** for faster lookups:  
```python
session.execute("""
    CREATE MATERIALIZED VIEW IF NOT EXISTS employees_by_lastname AS
    SELECT department_id, office_id, employee_id, first_name, email
    FROM employees
    WHERE last_name IS NOT NULL
    PRIMARY KEY (last_name, department_id, office_id, employee_id)
""")
```
This improves **read performance** for `last_name`-based queries.

---

### **4. Asynchronous Queries (Performance Optimization)**
If you want to **improve performance**, use async queries:  
```python
from cassandra.concurrent import execute_concurrent_with_args

query = "INSERT INTO employees (department_id, office_id, employee_id, first_name, last_name, email) VALUES (?, ?, ?, ?, ?, ?)"
data = [
    (4, 104, 5004, 'Charlie', 'Davis', 'charlie.davis@example.com'),
    (5, 105, 5005, 'Eve', 'Wilson', 'eve.wilson@example.com')
]

execute_concurrent_with_args(session, query, data)
print("Asynchronous insert completed")
```
This **improves speed** by executing multiple queries in parallel.

---
