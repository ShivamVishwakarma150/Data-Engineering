# **Day 2 - Detailed Notes**  

## **1. Database Management**  

### **1.1 Creating and Using a Database**  
```sql
CREATE DATABASE class2_db;
USE class2_db;
```
- **`CREATE DATABASE`** → Creates a new database named `class2_db`.  
- **`USE class2_db`** → Selects `class2_db` as the active database.  

---

## **2. Table Operations**  

### **2.1 Creating a Table**  
```sql
CREATE TABLE IF NOT EXISTS employee (
    id INT,
    name VARCHAR(50),
    address VARCHAR(50),
    city VARCHAR(50)
);
```
- **Creates a table `employee`** with `id`, `name`, `address`, and `city` columns.  
- **`IF NOT EXISTS`** prevents errors if the table already exists.  

### **2.2 Inserting Data into a Table**  
```sql
INSERT INTO employee VALUES (1, 'Shivam', 'RJPM', 'Lucknow');
SELECT * FROM employee;
```
- Inserts a record into `employee` and retrieves all records.  

### **2.3 Altering a Table**  

#### **Adding a Column**  
```sql
ALTER TABLE employee ADD DOB DATE;
```
- Adds a new column `DOB` of type `DATE`.  

#### **Modifying a Column (Change Data Type or Length)**  
```sql
ALTER TABLE employee MODIFY COLUMN name VARCHAR(100);
```
- Increases `name` column length from `50` to `100`.  

#### **Deleting a Column**  
```sql
ALTER TABLE employee DROP COLUMN city;
```
- Removes the `city` column from the `employee` table.  

#### **Renaming a Column**  
```sql
ALTER TABLE employee RENAME COLUMN name TO full_name;
```
- Renames `name` column to `full_name`.  

---

## **3. Primary Key and Unique Constraints**  

### **3.1 Creating a Table with a Primary Key**  
```sql
CREATE TABLE persons (
    id INT,
    name VARCHAR(50),
    age INT,
    CONSTRAINT pk PRIMARY KEY (id)
);
```
- Defines `id` as the **primary key**, ensuring:  
  - It is **unique**.  
  - It **cannot be NULL**.  

#### **Inserting Data into `persons` Table**
```sql
INSERT INTO persons VALUES (1, 'Shivam', 29);
```
- ✅ Success: Unique `id` is provided.  

```sql
INSERT INTO persons VALUES (1, 'Rahul', 28);
```
❌ **Fails:** Duplicate entry for primary key (`id`).  

```sql
INSERT INTO persons VALUES (NULL, 'Rahul', 28);
```
❌ **Fails:** Primary key cannot be `NULL`.  

---

## **4. Foreign Key Constraints**  

### **4.1 Creating Tables with a Foreign Key**  
```sql
CREATE TABLE customer (
    cust_id INT,
    name VARCHAR(50), 
    age INT,
    CONSTRAINT pk PRIMARY KEY (cust_id)
);
```
```sql
CREATE TABLE orders (
    order_id INT,
    order_num INT,
    customer_id INT,
    CONSTRAINT pk PRIMARY KEY (order_id),
    CONSTRAINT fk FOREIGN KEY (customer_id) REFERENCES customer(cust_id)
);
```
- `customer_id` in `orders` is a **foreign key** referencing `cust_id` in `customer`.  
- Ensures **referential integrity** – a record in `orders` **must** have a corresponding `customer_id` in `customer`.  

#### **Inserting Valid Records**  
```sql
INSERT INTO customer VALUES (1, "Shivam", 29);
INSERT INTO customer VALUES (2, "Rahul", 30);
INSERT INTO orders VALUES (1001, 20, 1);
INSERT INTO orders VALUES (1002, 30, 2);
```
✅ **Success:** Customers exist in the `customer` table.  

#### **Inserting Invalid Record (Foreign Key Violation)**  
```sql
INSERT INTO orders VALUES (1004, 35, 5);
```
❌ **Fails:** `customer_id = 5` does not exist in `customer`.  

---

## **5. Drop vs Truncate Commands**  

### **Drop Command**
```sql
DROP TABLE persons;
```
- **Deletes** the table **completely** (structure + data).  

### **Truncate Command**
```sql
TRUNCATE TABLE persons;
```
- **Deletes all records**, but keeps the table structure.  

---

## **6. SELECT Statement Operations**  

### **6.1 Display All Records**
```sql
SELECT * FROM employee;
```
- Fetches **all** records from `employee`.  

### **6.2 Display Specific Columns**
```sql
SELECT name, salary FROM employee;
```
- Fetches only `name` and `salary` columns.  

### **6.3 Using Aliases**
```sql
SELECT name AS employee_name, salary AS employee_salary FROM employee;
```
- **Aliases (`AS`)** rename column names in output.  

### **6.4 Using DISTINCT**
```sql
SELECT DISTINCT (hiring_date) AS distinct_hiring_dates FROM employee;
```
- Fetches **unique hiring dates**.  

---

## **7. Filtering Data with WHERE Clause**  

### **7.1 Basic Comparisons**
```sql
SELECT * FROM employee WHERE salary > 20000;
```
- Lists employees **earning more than 20,000**.  

### **7.2 Using BETWEEN**
```sql
SELECT * FROM employee WHERE salary BETWEEN 10000 AND 28000;
```
- Fetches employees earning **between 10,000 and 28,000**.  

### **7.3 Using LIKE for Pattern Matching**  
```sql
SELECT * FROM employee WHERE name LIKE 'S%';
```
- **Finds names starting with "S"**.  

```sql
SELECT * FROM employee WHERE name LIKE '_h%';
```
- **Finds names where the second letter is "h"**.  

```sql
SELECT * FROM employee WHERE name LIKE '_____' ;
```
- **Finds names with exactly 5 characters**.  

---

## **8. Updating and Deleting Records**  

### **8.1 Updating Records**
```sql
UPDATE employee SET salary = salary + salary * 0.2;
```
- **Increases salary by 20%** for all employees.  

```sql
UPDATE employee SET salary = 80000 WHERE hiring_date = '2021-08-10';
```
- Updates salary **only for employees hired on `2021-08-10`**.  

### **8.2 Deleting Records**
```sql
DELETE FROM employee WHERE hiring_date = '2021-08-10';
```
- **Deletes employees hired on `2021-08-10`**.  

---

## **9. Sorting Data using ORDER BY**  

### **9.1 Sorting in Ascending Order**
```sql
SELECT * FROM employee ORDER BY name;
```
- Orders employees **alphabetically** by name.  

### **9.2 Sorting in Descending Order**
```sql
SELECT * FROM employee ORDER BY salary DESC;
```
- Orders employees **by highest salary first**.  

### **9.3 Multi-Level Sorting**
```sql
SELECT * FROM employee ORDER BY salary DESC, name ASC;
```
- Orders by **salary (highest first)**.  
- If two employees have the same salary, they are **sorted by name (A → Z)**.  

---

## **10. Finding Maximum and Minimum Salary**  
```sql
SELECT * FROM employee ORDER BY salary DESC LIMIT 1;
```
- **Finds the employee with the highest salary**.  

```sql
SELECT * FROM employee ORDER BY salary ASC LIMIT 1;
```
- **Finds the employee with the lowest salary**.  

---

## **11. Auto-Increment and Limiting Results**  

### **11.1 Using Auto-Increment**
```sql
CREATE TABLE auto_inc_exmp (
    id INT AUTO_INCREMENT,
    name VARCHAR(20),
    PRIMARY KEY (id)
);
```
- **`AUTO_INCREMENT`** automatically generates unique values for `id`.  

### **11.2 Using LIMIT**
```sql
SELECT * FROM employee LIMIT 2;
```
- **Fetches only the first two records**.  

---

## **Key Takeaways**
- `PRIMARY KEY` = **Unique + Not Null**  
- `FOREIGN KEY` = **References another table**  
- `ALTER TABLE` = **Modify structure**  
- `ORDER BY` = **Sort results**  
- `LIMIT` = **Restrict output**  

Here are **detailed notes** based on the SQL queries you provided:  

---

# **SQL Query Notes - Sorting, Filtering, and Searching**  

---

## **1. Auto-Increment and Inserting Data**  

### **1.1 Creating and Using an Auto-Increment Column**  
```sql
CREATE TABLE auto_inc_exmp (
    id INT AUTO_INCREMENT,
    name VARCHAR(20),
    PRIMARY KEY (id)
);
```
- **AUTO_INCREMENT** automatically assigns unique values to `id`.  
- **PRIMARY KEY** ensures uniqueness.  

### **1.2 Inserting Data**
```sql
INSERT INTO auto_inc_exmp(name) VALUES ('Shivam');
INSERT INTO auto_inc_exmp(name) VALUES ('Rahul');
```
- Automatically generates **id values** (1, 2, 3, etc.).  

### **1.3 Fetching Data**
```sql
SELECT * FROM auto_inc_exmp;
```
- Retrieves all records from `auto_inc_exmp`.  

---

## **2. Limiting Query Results**  

### **2.1 Retrieve All Employees**
```sql
SELECT * FROM employee;
```
- Returns **all records** from `employee`.  

### **2.2 Retrieve Only 2 Employees**
```sql
SELECT * FROM employee LIMIT 2;
```
- Fetches only **the first 2 rows**.  

---

## **3. Sorting Data Using `ORDER BY`**  

### **3.1 Sorting in Ascending Order (Default)**
```sql
SELECT * FROM employee ORDER BY name;
```
- Sorts employees **alphabetically (A → Z)**.  

### **3.2 Sorting in Descending Order**
```sql
SELECT * FROM employee ORDER BY name DESC;
```
- Sorts employees **in reverse order (Z → A)**.  

### **3.3 Sorting by Salary (Descending)**
```sql
SELECT * FROM employee ORDER BY salary DESC;
```
- Orders employees **by highest salary first**.  

### **3.4 Sorting by Salary, then by Name**
```sql
SELECT * FROM employee ORDER BY salary DESC, name ASC;
```
- **First sorts by salary (highest first)**.  
- If two employees have the same salary, they are **sorted by name (A → Z)**.  

---

## **4. Finding Maximum and Minimum Salary**  

### **4.1 Find the Employee with the Highest Salary**
```sql
SELECT * FROM employee ORDER BY salary DESC LIMIT 1;
```
- **Fetches the employee with the highest salary**.  

### **4.2 Find the Employee with the Lowest Salary**
```sql
SELECT * FROM employee ORDER BY salary LIMIT 1;
```
- **Fetches the employee with the lowest salary**.  

---

## **5. Using WHERE Clause for Filtering Data**  

### **5.1 Filtering Employees by Salary**  
```sql
SELECT * FROM employee WHERE salary > 20000;
```
- **Lists employees earning more than 20,000**.  

```sql
SELECT * FROM employee WHERE salary >= 20000;
```
- **Lists employees earning 20,000 or more**.  

```sql
SELECT * FROM employee WHERE salary < 20000;
```
- **Lists employees earning less than 20,000**.  

```sql
SELECT * FROM employee WHERE salary <= 20000;
```
- **Lists employees earning 20,000 or less**.  

### **5.2 Filtering by Age**  
```sql
SELECT * FROM employee WHERE age = 20;
```
- **Finds employees who are exactly 20 years old**.  

```sql
SELECT * FROM employee WHERE age != 20;
SELECT * FROM employee WHERE age <> 20;
```
- **Finds employees whose age is NOT 20**.  

### **5.3 Using `AND` and `OR` Operators**  

#### **Find Employees Who Joined on `2021-08-11` AND Have Salary < 11,500**  
```sql
SELECT * FROM employee WHERE hiring_date = '2021-08-11' AND salary < 11500;
```
- **Both conditions must be true**.  

#### **Find Employees Who Joined After `2021-08-11` OR Have Salary < 20,000**  
```sql
SELECT * FROM employee WHERE hiring_date > '2021-08-11' OR salary < 20000;
```
- **At least one condition must be true**.  

---

## **6. Using `BETWEEN` for Ranges**  

### **6.1 Find Employees Hired Between `2021-08-05` and `2021-08-11`**  
```sql
SELECT * FROM employee WHERE hiring_date BETWEEN '2021-08-05' AND '2021-08-11';
```
- **Finds employees who joined within the given date range**.  

### **6.2 Find Employees Earning Between 10,000 and 28,000**  
```sql
SELECT * FROM employee WHERE salary BETWEEN 10000 AND 28000;
```
- **Finds employees whose salary falls within the specified range**.  

---

## **7. Using `LIKE` for Pattern Matching**  

### **7.1 Find Employees Whose Name Starts with "S"**  
```sql
SELECT * FROM employee WHERE name LIKE 'S%';
```
- **Finds all names that start with "S"**.  

### **7.2 Find Employees Whose Name Starts with "Sh"**  
```sql
SELECT * FROM employee WHERE name LIKE 'Sh%';
```
- **Finds names that start with "Sh"**.  

### **7.3 Find Employees Whose Name Ends with "l"**  
```sql
SELECT * FROM employee WHERE name LIKE '%l';
```
- **Finds names that end with "l"**.  

### **7.4 Find Employees Whose Name Starts with "S" and Ends with "m"**  
```sql
SELECT * FROM employee WHERE name LIKE 'S%m';
```
- **Finds names that start with "S" and end with "m"**.  

### **7.5 Find Employees Whose Name Has Exactly 5 Characters**  
```sql
SELECT * FROM employee WHERE name LIKE '_____';
```
- **Finds names with exactly 5 characters (each underscore `_` represents one character).**  

### **7.6 Find Employees Whose Name Has At Least 5 Characters**  
```sql
SELECT * FROM employee WHERE name LIKE '%_____%';
```
- **Finds names with at least 5 characters**.  

---

## **Key Takeaways**
1. **Sorting**
   - `ORDER BY column ASC|DESC` → Sort data in ascending or descending order.  
   - Multi-level sorting: `ORDER BY column1 DESC, column2 ASC`.  

2. **Filtering**
   - `WHERE` is used to filter records.  
   - Use `BETWEEN` for range-based filtering.  
   - Use `LIKE` for pattern matching.  

3. **Finding Min/Max**
   - `ORDER BY salary DESC LIMIT 1` → Highest salary.  
   - `ORDER BY salary ASC LIMIT 1` → Lowest salary.  

4. **Pattern Matching (`LIKE`)**
   - `%` → Matches **zero or more** characters.  
   - `_` → Matches **exactly one** character.  

5. **Logical Operators**
   - `AND` → Both conditions must be true.  
   - `OR` → At least one condition must be true.  
   - `NOT` → Negates a condition.  

---

These notes summarize all the **sorting, filtering, and searching** operations in SQL. Let me know if you need any modifications! 🚀