# **SQL Detailed Explanation of Key Topics**  

## **1. Primary Key**  
A **Primary Key** is a column (or a set of columns) that **uniquely identifies each row** in a table.  

### **Characteristics of a Primary Key**  
✅ **Uniqueness** → Each primary key value must be **unique** for each row.  
✅ **Immutable** → The value **should not change** once assigned.  
✅ **Simplicity** → It should be **as simple as possible** (usually a single column).  
✅ **Non-Intelligent** → It should not have any meaningful business information (e.g., avoid using phone numbers, email IDs, etc.).  
✅ **Indexed** → Primary keys are automatically indexed, improving **query performance**.  
✅ **Referential Integrity** → Used as a reference in **Foreign Keys** in other tables.  
✅ **Data Type** → Commonly used types include **INTEGER, BIGINT, and UUIDs**.  

### **Example of Creating a Primary Key**
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);
```
- `emp_id` is the **Primary Key** and ensures that each employee has a **unique** identifier.  

---

## **2. Foreign Key**  
A **Foreign Key** is a column that **establishes a relationship between two tables** by referencing a **Primary Key** in another table.  

### **Characteristics of a Foreign Key**  
✅ **Ensures Referential Integrity** → Prevents invalid data entry by enforcing relationships.  
✅ **Can Be NULL** → If not restricted, Foreign Keys **can store NULL values**.  
✅ **Must Match a Primary Key** → The Foreign Key **must reference an existing Primary Key** in another table.  
✅ **No Uniqueness Required** → Foreign Key values can be **duplicated across multiple rows**.  

### **Example of a Foreign Key**
```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```
- `dept_id` in the `employees` table references `dept_id` in `departments`, ensuring that employees can only be assigned to **valid departments**.  

---

## **3. DELETE Command**  
The `DELETE` statement is used to **remove specific records** from a table.  

### **Syntax**  
```sql
DELETE FROM table_name WHERE condition;
```

### **Example**  
```sql
DELETE FROM Students WHERE ID = 5;
```
- Deletes the record where `ID = 5` in the `Students` table.  

⚠ **Warning**: Running `DELETE` without a `WHERE` clause will **delete all rows** from the table!  

---

## **4. DROP vs TRUNCATE vs DELETE**  

| Feature       | DELETE | TRUNCATE | DROP |
|--------------|--------|----------|------|
| Removes Data | ✅ Yes | ✅ Yes | ✅ Yes |
| Removes Table Structure | ❌ No | ❌ No | ✅ Yes |
| Can Be Rolled Back | ✅ Yes | ❌ No | ❌ No |
| Uses WHERE Clause | ✅ Yes | ❌ No | ❌ No |
| Resets Auto-Increment | ❌ No | ✅ Yes | ✅ Yes |

### **Example Usage**  
- **DELETE:**  
  ```sql
  DELETE FROM employees WHERE age > 40;
  ```
  - Removes employees older than 40.  

- **TRUNCATE:**  
  ```sql
  TRUNCATE TABLE employees;
  ```
  - Removes all records **but keeps the table structure**.  

- **DROP:**  
  ```sql
  DROP TABLE employees;
  ```
  - Deletes the table **completely**.  

---

## **5. SQL Joins**  
SQL **Joins** are used to **combine data from multiple tables** based on a related column.  

### **5.1 INNER JOIN**  
✅ Returns **only matching records** from both tables.  
```sql
SELECT Customers.customer_id, Customers.first_name, Orders.amount 
FROM Customers
INNER JOIN Orders ON Orders.customer = Customers.customer_id;
```
- Fetches customers **only if they have placed an order**.  

### **5.2 LEFT JOIN**  
✅ Returns **all records from the left table** and **matching records from the right table**.  
✅ If no match is found, NULL is returned.  
```sql
SELECT Customers.customer_id, Customers.first_name, Orders.amount 
FROM Customers
LEFT JOIN Orders ON Orders.customer = Customers.customer_id;
```
- Fetches **all customers**, even if they haven't placed an order.  

### **5.3 RIGHT JOIN**  
✅ Returns **all records from the right table** and **matching records from the left table**.  
```sql
SELECT Customers.customer_id, Customers.first_name, Orders.amount 
FROM Customers
RIGHT JOIN Orders ON Orders.customer = Customers.customer_id;
```
- Fetches **all orders**, even if the customer does not exist.  

### **5.4 FULL JOIN**  
✅ Returns **all records from both tables**, with NULLs where there is no match.  
```sql
SELECT Customers.customer_id, Customers.first_name, Orders.amount 
FROM Customers
FULL OUTER JOIN Orders ON Orders.customer = Customers.customer_id;
```

### **5.5 CROSS JOIN**  
✅ Returns **a Cartesian product** of two tables (**matches every row in the first table with every row in the second table**).  
```sql
SELECT Model.car_model, Color.color_name
FROM Model
CROSS JOIN Color;
```

---

## **6. Self Join**  
A **Self Join** is a join where a table is **joined with itself**.  

### **Example: Finding Employee Managers**  
```sql
SELECT e1.emp_id, e1.name, e2.name AS ManagerName
FROM Employees e1
JOIN Employees e2 ON e1.manager_id = e2.emp_id;
```
- Retrieves **each employee along with their manager's name**.  

---

## **7. GROUP BY with ROLLUP**  
✅ `GROUP BY` is used to **aggregate data**.  
✅ `ROLLUP` adds **subtotals and grand totals**.  

### **Example Usage**  
```sql
SELECT
    SUM(payment_amount),
    YEAR(payment_date) AS 'Payment Year',
    store_id AS 'Store'
FROM payment
GROUP BY YEAR(payment_date), store_id WITH ROLLUP
ORDER BY YEAR(payment_date), store_id;
```
- **Adds subtotals and grand totals** for each group.  

---

## **8. Views in SQL**  
✅ A **View** is a virtual table based on a query.  
✅ It simplifies complex queries by storing frequently used SELECT statements.  

### **Example of Creating a View**  
```sql
CREATE VIEW HighPricedProducts AS
SELECT ProductName, Price
FROM Products
WHERE Price > 30;
```
- **Fetches all products with a price greater than 30** and stores it as a **virtual table**.  

### **Advantages of Views**  
✔ **Security** → Restricts access to certain data.  
✔ **Simplifies Queries** → Encapsulates complex logic.  
✔ **Performance Optimization** → Speeds up frequently used queries.  

---

## **Key Takeaways**  

### **🔹 Primary Key**  
- **Ensures uniqueness** and is **automatically indexed**.  

### **🔹 Foreign Key**  
- **Maintains relationships** between tables.  

### **🔹 DELETE vs TRUNCATE vs DROP**  
- `DELETE` removes specific records, `TRUNCATE` clears the table, `DROP` removes the table.  

### **🔹 Joins**  
- `INNER JOIN` → Only matching records.  
- `LEFT JOIN` → All records from the left table, matching from right.  
- `RIGHT JOIN` → All records from the right table, matching from left.  
- `FULL JOIN` → All records from both tables.  
- `CROSS JOIN` → Cartesian product.  
- `SELF JOIN` → Table joins itself.  

### **🔹 GROUP BY & ROLLUP**  
- Aggregates data and adds **subtotals** and **totals**.  

### **🔹 Views**  
- **Virtual tables** that simplify complex queries and **improve security**.  

---

<br/>

# **SQL Advanced Concepts - Detailed Definitions**  

## **1. `IS NULL` and `IS NOT NULL`**  
- `IS NULL` and `IS NOT NULL` are used to **filter NULL values** in SQL queries.  
- **`NULL`** represents a missing or unknown value in a database.  
- **Example Usage:**  
  ```sql
  SELECT * FROM employee WHERE age IS NULL;
  ```
  - Retrieves employees where `age` is NULL.  

  ```sql
  SELECT * FROM employee WHERE salary IS NOT NULL;
  ```
  - Retrieves employees where `salary` is NOT NULL.  

---

## **2. `GROUP BY` Clause**  
- `GROUP BY` is used to **group records with the same values in specified columns**.  
- It is often used with aggregate functions like **SUM, COUNT, AVG, MIN, and MAX**.  
- **Example Usage:**  
  ```sql
  SELECT country, COUNT(*) AS order_count FROM orders_data GROUP BY country;
  ```
  - Groups orders by country and counts total orders for each country.  

---

## **3. Aggregate Functions**  
Aggregate functions perform calculations on multiple rows and return a **single value**.  

### **3.1 `SUM()`**  
- Returns the **total sum** of a column.  
- **Example:**  
  ```sql
  SELECT SUM(salary) FROM employee;
  ```
  - Returns the **total salary** of all employees.  

### **3.2 `COUNT()`**  
- Returns the **number of records**.  
- **Example:**  
  ```sql
  SELECT COUNT(*) FROM employee;
  ```
  - Returns the **total number of employees**.  

### **3.3 `AVG()`**  
- Returns the **average value** of a numeric column.  
- **Example:**  
  ```sql
  SELECT AVG(salary) FROM employee;
  ```
  - Returns the **average salary** of employees.  

### **3.4 `MIN()` & `MAX()`**  
- `MIN()` → Finds the **lowest value**.  
- `MAX()` → Finds the **highest value**.  
- **Example:**  
  ```sql
  SELECT MIN(salary) FROM employee;
  SELECT MAX(salary) FROM employee;
  ```
  - Returns **minimum and maximum salary** from `employee` table.  

---

## **4. `HAVING` Clause**  
- `HAVING` is used to **filter aggregated data** after applying `GROUP BY`.  
- **Example Usage:**  
  ```sql
  SELECT country FROM orders_data GROUP BY country HAVING COUNT(*) = 1;
  ```
  - Retrieves **countries where only one order was placed**.  

---

## **5. `GROUP_CONCAT()`**  
- `GROUP_CONCAT()` combines values from multiple rows **into a single string**.  
- **Example Usage:**  
  ```sql
  SELECT country, GROUP_CONCAT(state) FROM orders_data GROUP BY country;
  ```
  - Lists **all states** for each country in a single column.  

- **Using DISTINCT & Custom Separator:**  
  ```sql
  SELECT country, GROUP_CONCAT(DISTINCT state ORDER BY state DESC SEPARATOR ' | ') FROM orders_data GROUP BY country;
  ```
  - **Removes duplicates**, orders states **descendingly**, and separates values with `' | '` instead of `,`.  

---

## **6. Subqueries**  
- A **subquery** is a query inside another SQL query.  
- Used to fetch data **dynamically based on another query result**.  
- **Example:**  
  ```sql
  SELECT * FROM employees WHERE salary > (SELECT salary FROM employees WHERE name = 'Rohit');
  ```
  - Fetches employees **earning more than Rohit**.  

---

## **7. `IN` and `NOT IN` Operators**  
- `IN` checks if a value **exists in a given list or subquery result**.  
- `NOT IN` excludes values from the given list.  
- **Example Usage:**  
  ```sql
  SELECT * FROM orders_data WHERE state IN ('Seattle', 'Goa');
  ```
  - Fetches orders placed **only in Seattle or Goa**.  

  ```sql
  SELECT * FROM orders_data WHERE state NOT IN ('Seattle', 'Goa');
  ```
  - Fetches orders **not placed in Seattle or Goa**.  

---

## **8. SQL Joins**  
Joins are used to **combine data from multiple tables** based on a related column.  

### **8.1 INNER JOIN**  
- Returns **only matching rows** from both tables.  
- **Example:**  
  ```sql
  SELECT o.*, c.*
  FROM orders o
  INNER JOIN customers c ON o.cust_id = c.cust_id;
  ```
  - Retrieves **only orders with matching customers**.  

### **8.2 LEFT JOIN**  
- Returns **all rows from the left table** and **matching rows from the right table**.  
- **Example:**  
  ```sql
  SELECT o.*, c.*
  FROM orders o
  LEFT JOIN customers c ON o.cust_id = c.cust_id;
  ```
  - If a customer **doesn’t exist**, their details will be NULL.  

### **8.3 RIGHT JOIN**  
- Returns **all rows from the right table** and **matching rows from the left table**.  
- **Example:**  
  ```sql
  SELECT o.*, c.*
  FROM orders o
  RIGHT JOIN customers c ON o.cust_id = c.cust_id;
  ```
  - If an order **doesn’t exist**, it returns NULL.  

### **8.4 Joining More Than Two Tables**
```sql
SELECT o.*, c.*, s.*
FROM orders o
INNER JOIN customers c ON o.cust_id = c.cust_id
INNER JOIN shippers s ON o.shipper_id = s.ship_id;
```
- Fetches **order details, customer information, and shipper details** in a single query.  

---

## **9. Recursive Queries (Hierarchical Data - Uber SQL Question Example)**  
- Used to **identify hierarchical relationships** like **parent-child** relationships.  

### **9.1 Creating a Tree Structure**
```sql
CREATE TABLE tree (
    node INT,
    parent INT
);
```
### **9.2 Inserting Data**
```sql
INSERT INTO tree VALUES (5, 8), (9, 8), (4, 5), (2, 9), (1, 5), (3, 9), (8, NULL);
```
### **9.3 Categorizing Nodes**
```sql
SELECT node,
       CASE
           WHEN node NOT IN (SELECT DISTINCT parent FROM tree WHERE parent IS NOT NULL) THEN 'LEAF'
           WHEN parent IS NULL THEN 'ROOT'
           ELSE 'INNER'
       END AS node_type
FROM tree;
```
- **ROOT** → Parent is NULL.  
- **INNER** → Exists as a parent and a child.  
- **LEAF** → Never appears as a parent.  

---

## **10. Self Join (Finding Managers from Employee Data)**  

### **10.1 Creating Employee Hierarchy Table**
```sql
CREATE TABLE employees_full_data (
    emp_id INT,
    name VARCHAR(50),
    mgr_id INT
);
```
### **10.2 Inserting Data**
```sql
INSERT INTO employees_full_data VALUES (1, 'Shivam', 3), 
                                       (2, 'Amit', 3), 
                                       (3, 'Rajesh', 4), 
                                       (4, 'Ankit', 6), 
                                       (6, 'Nikhil', NULL);
```
### **10.3 Finding Distinct Managers**
```sql
SELECT emp.name AS manager
FROM employees_full_data emp
INNER JOIN (SELECT DISTINCT mgr_id FROM employees_full_data) mgr ON mgr.mgr_id = emp.emp_id;
```
- Fetches **distinct managers** from employee records.  

---

# **Key Takeaways**
✅ `GROUP BY` → Groups data for aggregation.  
✅ `HAVING` → Filters grouped data.  
✅ `GROUP_CONCAT` → Combines multiple values into a string.  
✅ `IN` & `NOT IN` → Filters based on a list.  
✅ `JOINS` → Merges multiple tables.  
✅ `CASE WHEN` → Categorizes data dynamically.  
✅ `SELF JOIN` → Used for hierarchical data.  


Here are **detailed notes** covering all the SQL concepts from your queries.  

