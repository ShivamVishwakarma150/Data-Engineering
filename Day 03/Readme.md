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

