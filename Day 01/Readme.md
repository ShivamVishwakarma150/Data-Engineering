# **Day 1 - Detailed Notes**  

## **1. Installing MySQL**  
To install MySQL, visit the official MySQL website: [MySQL Downloads](https://www.mysql.com/downloads/).  

---

## **2. Database Management System (DBMS)**  
A **Database Management System (DBMS)** is software that allows users to store, retrieve, manage, and manipulate data in databases.  

### **Most Used DBMS:**
- **MySQL**  
- **Oracle**  
- **Microsoft SQL Server**  
- **PostgreSQL**  
- **MongoDB**  

### **Advantages of DBMS:**
- Efficient data management  
- Data security and integrity  
- Multi-user access control  
- Backup and recovery  
- Data consistency  

---

## **3. Relational Database Management System (RDBMS)**  
- In **RDBMS**, data is stored in **tables** (database objects).  
- A **table** consists of **columns** (fields) and **rows** (records).  
- **Row (Record)**: A single data entry in a table.  
- **Column (Field)**: A vertical entity containing data for a specific attribute.  

---

## **4. What is SQL?**  
- **SQL (Structured Query Language)** is used to communicate with databases.  
- SQL became an **ANSI standard in 1986** and **ISO standard in 1987**.  

### **What Can SQL Do?**
- Execute queries on a database  
- Retrieve, insert, update, and delete records  
- Create new databases and tables  
- Create stored procedures and views  
- Manage permissions on database objects  

SQL is **very powerful** and widely used in database management.  

---

## **5. MySQL Data Types**  
Data types define what kind of values a column can store.  

### **Main Data Types in MySQL:**  
1. **String Data Types** (e.g., `CHAR`, `VARCHAR`, `TEXT`)  
2. **Numeric Data Types** (e.g., `INT`, `FLOAT`, `DECIMAL`)  
3. **Date and Time Data Types** (e.g., `DATE`, `DATETIME`, `TIMESTAMP`)  

---

## **6. Types of SQL Commands**  
SQL commands are categorized into four types:  

### **A. DDL (Data Definition Language)**
Used for defining the structure of a database.  

- **CREATE** – Creates a database/table/index/view.  
- **DROP** – Deletes a database/table/index/view.  
- **ALTER** – Modifies database structures.  
- **TRUNCATE** – Deletes all records from a table but retains the structure.  
- **RENAME** – Renames database objects.  

🔗 References:  
- [SQL CREATE](https://www.geeksforgeeks.org/sql-create/)  
- [SQL DROP & TRUNCATE](https://www.geeksforgeeks.org/sql-drop-truncate/)  
- [SQL ALTER](https://www.geeksforgeeks.org/sql-alter-add-drop-modify/)  

---

### **B. DML (Data Manipulation Language)**  
Used to manipulate data inside tables.  

- **INSERT** – Adds records to a table.  
- **UPDATE** – Modifies existing records.  
- **DELETE** – Removes records from a table.  

🔗 References:  
- [SQL INSERT](https://www.geeksforgeeks.org/sql-insert-statement/)  
- [SQL UPDATE](https://www.geeksforgeeks.org/sql-update-statement/)  
- [SQL DELETE](https://www.geeksforgeeks.org/sql-delete-statement/)  

---

### **C. DQL (Data Query Language)**  
Used to query data from a database.  

- **SELECT** – Retrieves data from a table.  
- It processes queries and returns a temporary result set.  

---

### **D. DCL (Data Control Language)**  
Used to control user access and permissions.  

- **GRANT** – Provides specific privileges to users.  
- **REVOKE** – Removes previously granted privileges.  

---

## **7. SQL Constraints**  
Constraints are rules applied to table columns to ensure **data integrity and reliability**.  

### **Common SQL Constraints:**  
- **NOT NULL** – Prevents NULL values in a column.  
- **UNIQUE** – Ensures all values in a column are distinct.  
- **PRIMARY KEY** – A combination of `NOT NULL` and `UNIQUE` to uniquely identify each row.  
- **FOREIGN KEY** – Links two tables to maintain referential integrity.  
- **CHECK** – Ensures column values satisfy a specific condition.  
- **DEFAULT** – Sets a default value for a column if no value is provided.  

🔗 References:  
- [NOT NULL](https://www.w3schools.com/sql/sql_notnull.asp)  
- [UNIQUE](https://www.w3schools.com/sql/sql_unique.asp)  
- [PRIMARY KEY](https://www.w3schools.com/sql/sql_primarykey.asp)  
- [FOREIGN KEY](https://www.w3schools.com/sql/sql_foreignkey.asp)  
- [CHECK](https://www.w3schools.com/sql/sql_check.asp)  
- [DEFAULT](https://www.w3schools.com/sql/sql_default.asp)  

---

## **8. SQL Commands Overview**  

### **SQL CREATE, DROP, TRUNCATE**  
- **TRUNCATE TABLE** – Removes all records but keeps the table structure.  
- **CREATE DATABASE** – Creates a new database.  
- **DROP DATABASE** – Deletes a database.  
- **CREATE TABLE** – Creates a new table.  
- **DROP TABLE** – Deletes a table.  

---

### **SQL ALTER TABLE**  
Used to modify the structure of an existing table.  

- **ALTER TABLE – MODIFY DATATYPE**  
- **ALTER TABLE – RENAME COLUMN**  
- **ALTER TABLE – DROP COLUMN**  
- **ALTER TABLE – ADD COLUMN**  

---

## **9. Important SQL Notes**  
- **SQL keywords are NOT case-sensitive:**  
  `SELECT` is the same as `select`.  
- **Semicolon (`;`) is used to separate SQL statements**, especially in systems where multiple SQL statements can be executed in one command.  

---

Here are detailed notes based on the SQL commands you provided:  

---

# **SQL Commands and Notes**  

## **1. Viewing and Managing Databases**  

### **View List of Databases**  
```sql
SHOW DATABASES;
```
- This command displays all available databases in the MySQL server.  

### **Create a New Database**  
```sql
CREATE DATABASE noob_db;
```
- Creates a new database named `noob_db`.  

### **Delete a Database**  
```sql
DROP DATABASE first_demo;
```
- Deletes the `first_demo` database permanently.  

### **Select a Database**  
```sql
USE noob_db;
```
- Switches to `noob_db` for executing further SQL commands.  

---

## **2. Working with Tables**  

### **Create a Table**  
```sql
CREATE TABLE IF NOT EXISTS employee (
    id INT,
    emp_name VARCHAR(20)
);
```
- Creates an `employee` table with columns `id` and `emp_name`.  
- `IF NOT EXISTS` ensures the table is created only if it does not already exist.  

### **View List of Tables**  
```sql
SHOW TABLES;
```
- Displays all tables in the selected database.  

### **View Table Structure**  
```sql
SHOW CREATE TABLE employee;
```
- Shows the SQL statement used to create the `employee` table.  

---

## **3. Creating Tables with Additional Columns**  

### **Create a Table with More Columns**  
```sql
CREATE TABLE IF NOT EXISTS employee_v1 (
    id INT,
    name VARCHAR(50),
    salary DOUBLE, 
    hiring_date DATE
);
```
- Adds more attributes: `salary` (DOUBLE) and `hiring_date` (DATE).  

---

## **4. Inserting Data into a Table**  

### **Method 1: Insert Data Without Specifying Column Names**  
```sql
INSERT INTO employee_v1 VALUES (1, 'Shivam', 1000, '2021-09-15');
```
- Inserts values in the order defined in the table structure.  

### **Invalid Insert Statement (Incorrect Data Type)**  
```sql
INSERT INTO employee_v1 VALUES (1, 'Shivam', '2021-09-15');
```
- ❌ This will fail because `salary` expects a **DOUBLE**, but a **DATE** is provided.  

### **Method 2: Insert Data with Column Names**  
```sql
INSERT INTO employee_v1 (salary, name, id) 
VALUES (2000, 'Rahul', 2);
```
- Columns are specified, so values can be inserted in any order.  

### **Insert Multiple Records**  
```sql
INSERT INTO employee_v1 VALUES 
(3, 'Amit', 5000, '2021-10-28'),
(4, 'Nitin', 3500, '2021-09-16'),
(5, 'Kajal', 4000, '2021-09-20');
```
- Multiple rows can be inserted in one command.  

---

## **5. Querying Data from a Table**  

### **Retrieve All Records**  
```sql
SELECT * FROM employee_v1;
```
- Fetches all records from `employee_v1`.  

---

## **6. Integrity Constraints (IC) in SQL**  

### **Create a Table with Constraints**  
```sql
CREATE TABLE IF NOT EXISTS employee_with_constraints (
    id INT,
    name VARCHAR(50) NOT NULL,
    salary DOUBLE, 
    hiring_date DATE DEFAULT '2021-01-01',
    UNIQUE (id),
    CHECK (salary > 1000)
);
```
#### **Key Constraints:**
- **NOT NULL** → `name` column **cannot** have `NULL` values.  
- **DEFAULT** → `hiring_date` defaults to `'2021-01-01'` if not provided.  
- **UNIQUE** → `id` must have unique values (but can be NULL).  
- **CHECK** → `salary` must be greater than **1000**.  

### **Example 1: Integrity Constraint Failure (NOT NULL Violation)**  
```sql
INSERT INTO employee_with_constraints VALUES (1, NULL, 3000, '2021-11-20');
```
❌ **Fails:** `name` cannot be `NULL`.  

### **Correct Insert Statement**  
```sql
INSERT INTO employee_with_constraints VALUES (1, 'Shivam', 3000, '2021-11-20');
```
✅ **Success:** All constraints are satisfied.  

### **Example 2: Integrity Constraint Failure (UNIQUE Violation)**  
```sql
INSERT INTO employee_with_constraints VALUES (1, 'Rahul', 5000, '2021-10-23');
```
❌ **Fails:** Duplicate `id` is not allowed.  

### **Example 3: Allowing NULL in UNIQUE Constraint**  
```sql
INSERT INTO employee_with_constraints VALUES (NULL, 'Rahul', 5000, '2021-10-23');
```
✅ **Success:** `NULL` values are allowed in UNIQUE constraints.  

### **Example 4: Integrity Constraint Failure (CHECK Violation)**  
```sql
INSERT INTO employee_with_constraints VALUES (5, 'Amit', 500, '2023-10-24');
```
❌ **Fails:** Salary **must be greater than** 1000.  

### **Testing DEFAULT Value for Date**  
```sql
INSERT INTO employee_with_constraints (id, name, salary) 
VALUES (7, 'Neeraj', 3000);
```
✅ **Success:** `hiring_date` defaults to `'2021-01-01'`.  

### **Retrieve Data**  
```sql
SELECT * FROM employee_with_constraints;
```
---

## **7. Creating Named Constraints**  

### **Define Constraints with Explicit Names**  
```sql
CREATE TABLE IF NOT EXISTS employee_with_constraints_tmp (
    id INT,
    name VARCHAR(50) NOT NULL,
    salary DOUBLE, 
    hiring_date DATE DEFAULT '2021-01-01',
    CONSTRAINT unique_emp_id UNIQUE (id),
    CONSTRAINT salary_check CHECK (salary > 1000)
);
```
- **Named Constraints:**  
  - `unique_emp_id` → Ensures `id` is unique.  
  - `salary_check` → Ensures `salary > 1000`.  

### **Integrity Constraint Failure (Named Constraint Violation)**  
```sql
INSERT INTO employee_with_constraints_tmp VALUES (5, 'Amit', 500, '2023-10-24');
```
❌ **Fails:**  
- **Error Message:** `"Check constraint 'salary_check' is violated"`  
- Ensures `salary > 1000`.  

---

# **Key Takeaways**  
1. **Database Operations:**  
   - `SHOW DATABASES;` → List all databases.  
   - `CREATE DATABASE db_name;` → Create a new database.  
   - `DROP DATABASE db_name;` → Delete a database.  
   - `USE db_name;` → Switch to a database.  

2. **Table Operations:**  
   - `SHOW TABLES;` → List all tables.  
   - `SHOW CREATE TABLE table_name;` → View table structure.  

3. **Inserting Data:**  
   - Use `INSERT INTO table_name VALUES (...);`  
   - Use `INSERT INTO table_name (column1, column2) VALUES (...);`  

4. **Querying Data:**  
   - `SELECT * FROM table_name;` → Retrieve all records.  

5. **Integrity Constraints:**  
   - **NOT NULL** → Prevents NULL values.  
   - **DEFAULT** → Sets a default value.  
   - **UNIQUE** → Ensures uniqueness.  
   - **CHECK** → Validates conditions on data.  

---
