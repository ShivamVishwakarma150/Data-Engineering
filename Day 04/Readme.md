# **Day 4 - Advanced Topics in SQL**  

## **1. Window Functions in SQL**  
Window functions allow SQL operations to be performed **across a set of related rows** without aggregating them into a single result.  

### **How Window Functions Work**  
✅ Instead of operating on **individual rows**, they operate on a **window of rows** related to the current row.  
✅ The **window** is defined using different criteria:  
   - **Partitions**: The `PARTITION BY` clause groups data into smaller subsets.  
   - **Order of Rows**: The `ORDER BY` clause determines the sequence of rows in each partition.  
   - **Frames**: The `ROWS` or `RANGE` clause defines a **subset** of rows within a partition.  

### **Window Function Syntax**  
```sql
function_name (column) OVER (
    [PARTITION BY column_name]
    [ORDER BY column_name ASC|DESC]
)
```
✅ `function_name` → The window function to apply (e.g., `ROW_NUMBER()`, `RANK()`, `SUM()`).  
✅ `PARTITION BY` → Divides the result set into partitions.  
✅ `ORDER BY` → Defines the sorting within each partition.  

---

## **2. Types of Window Functions**  

### **2.1 Ranking Functions**  
Used to **assign ranks to rows** within each partition.  

#### **2.1.1 ROW_NUMBER()**
Assigns a **unique row number** starting from 1 for each row in a partition.  
```sql
SELECT StudentName, Subject, Marks,
       ROW_NUMBER() OVER(ORDER BY Marks DESC) AS RowNumber
FROM ExamResult;
```
- **Each row gets a unique row number**, even if values are the same.  

#### **2.1.2 RANK()**  
Assigns a rank **with gaps** if duplicate values exist.  
```sql
SELECT StudentName, Subject, Marks,
       RANK() OVER(ORDER BY Marks DESC) AS Rank
FROM ExamResult;
```
- **Duplicate values get the same rank, but the next rank is skipped**.  

#### **2.1.3 DENSE_RANK()**  
Similar to `RANK()`, but **without gaps in ranking**.  
```sql
SELECT StudentName, Subject, Marks,
       DENSE_RANK() OVER(ORDER BY Marks DESC) AS Rank
FROM ExamResult;
```
- **If two students have the same marks, they get the same rank, and the next rank is not skipped**.  

---

### **2.2 Value Functions**  
Used to **retrieve specific row values** from the window.  

#### **2.2.1 FIRST_VALUE()**  
Returns the **first value** in the window.  
```sql
SELECT employee_name, department, hours,
       FIRST_VALUE(employee_name) OVER (
           PARTITION BY department
           ORDER BY hours
       ) AS least_over_time
FROM overtime;
```
- **Finds the employee with the lowest hours in each department**.  

#### **2.2.2 LAST_VALUE()**  
Returns the **last value** in the window.  
```sql
SELECT employee_name, department, salary,
       LAST_VALUE(employee_name) OVER (
           PARTITION BY department
           ORDER BY salary
       ) AS max_salary
FROM Employee;
```
- **Finds the employee with the highest salary in each department**.  

#### **2.2.3 LAG()**  
Returns the **value of the previous row**.  
```sql
SELECT Year, Quarter, Sales,
       LAG(Sales, 1, 0) OVER (
           PARTITION BY Year
           ORDER BY Year, Quarter ASC
       ) AS PreviousQuarterSales
FROM ProductSales;
```
- **Finds sales for the previous quarter** for each year.  

#### **2.2.4 LEAD()**  
Returns the **value of the next row**.  
```sql
SELECT Year, Quarter, Sales,
       LEAD(Sales, 1, 0) OVER (
           PARTITION BY Year
           ORDER BY Year, Quarter ASC
       ) AS NextQuarterSales
FROM ProductSales;
```
- **Finds sales for the next quarter** for each year.  

---

### **2.3 Aggregate Functions in Windowing**  
Used to **perform calculations** over a set of rows in the window.  

✅ **SUM()** → Calculates the total sum.  
✅ **MIN()** → Finds the minimum value.  
✅ **MAX()** → Finds the maximum value.  
✅ **AVG()** → Computes the average value.  

---

## **3. Frame Clause in Window Functions**  
The **Frame Clause** defines the specific **subset of rows** used in window calculations.  

### **3.1 Syntax of the Frame Clause**  
```sql
function_name(expression) OVER (
  [PARTITION BY column]
  [ORDER BY column ASC|DESC]
  [ROWS|RANGE frame_start TO frame_end]
)
```
✅ `UNBOUNDED PRECEDING` → Starts at the **first row** of the partition.  
✅ `N PRECEDING` → Starts **N rows before** the current row.  
✅ `CURRENT ROW` → Starts at the **current row**.  
✅ `UNBOUNDED FOLLOWING` → Ends at the **last row** of the partition.  
✅ `N FOLLOWING` → Ends **N rows after** the current row.  

### **3.2 Example - Running Total**
```sql
SELECT date, revenue,
       SUM(revenue) OVER (
           ORDER BY date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM sales;
```
- **Calculates a running total of revenue over time**.  

---

## **4. Common Table Expressions (CTEs)**  
A **Common Table Expression (CTE)** is a temporary result set that simplifies complex queries.  

### **4.1 Syntax of CTEs**  
```sql
WITH sales_cte AS (
    SELECT sales_person, SUM(sales_amount) AS total_sales
    FROM sales_table
    GROUP BY sales_person
)
SELECT sales_person, total_sales
FROM sales_cte
WHERE total_sales > 1000;
```
✅ Makes **queries more readable**.  
✅ Can be used in **SELECT, INSERT, UPDATE, or DELETE** queries.  

### **4.2 Recursive CTEs**  
Allows for hierarchical (tree-structured) data.  
```sql
WITH RECURSIVE number_sequence AS (
    SELECT 1 AS number
    UNION ALL
    SELECT number + 1 
    FROM number_sequence 
    WHERE number < 10
)
SELECT * FROM number_sequence;
```
- **Generates a sequence of numbers from 1 to 10**.  

---

## **5. Subqueries in SQL**  
A **subquery** is a query **nested inside another query**.  

✅ **`IN` Operator** → Matches values within a list.  
```sql
SELECT * FROM Orders WHERE ProductName IN ('Apple', 'Banana');
```
✅ **`NOT IN` Operator** → Excludes values.  
```sql
SELECT * FROM Orders WHERE ProductName NOT IN ('Apple', 'Banana');
```
✅ **`ANY` Operator** → Returns true if **any value** meets the condition.  
✅ **`ALL` Operator** → Returns true if **all values** meet the condition.  
✅ **`EXISTS` Operator** → Checks if a subquery **returns results**.  

---

## **6. Views in SQL**  
A **View** is a virtual table based on a query.  

✅ **Encapsulates complex queries**.  
✅ **Restricts access to sensitive data**.  
✅ **Always shows up-to-date results**.  

### **6.1 Creating a View**  
```sql
CREATE VIEW HighPricedProducts AS
SELECT ProductName, Price
FROM Products
WHERE Price > 30;
```
- **Creates a virtual table** for products priced above $30.  

---

## **Key Takeaways**  
- **Window Functions** allow row-by-row calculations over a defined window.  
- **Ranking Functions** provide row numbers and rankings (`ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`).  
- **Value Functions** retrieve specific row values (`FIRST_VALUE()`, `LAG()`, `LEAD()`).  
- **CTEs** simplify complex queries and can be recursive.  
- **Subqueries** allow nested queries for advanced filtering.  
- **Views** create virtual tables for easier data access.  

---

This detailed breakdown explains all major topics from your **SQL Class 4 Notes**. Let me know if you need further explanations! 🚀