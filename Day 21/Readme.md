# Understanding Slowly Changing Dimensions (SCD)

Slowly Changing Dimensions (SCD) are database design techniques that track historical changes in dimension data over time. Here's a comprehensive breakdown of the different SCD types:

## 1. Type 0: Retain Original
- **Description**: Never changes (static dimensions)
- **Use Case**: For attributes that should never change (e.g., birth date, original customer signup date)
- **Implementation**: No special handling needed
- **Example**: A customer's original credit score when first onboarded

## 2. Type 1: Overwrite
- **Description**: Overwrites old data with new data (no history kept)
- **Use Case**: When historical values are irrelevant (corrections to typos)
- **Implementation**: Simple UPDATE statement
- **Example**: Updating a customer's phone number

```
UPDATE customers
SET phone = '555-1234'
WHERE customer_id = 1001;
```

## 3. Type 2: Add New Row (Most Common)
- **Description**: Creates new record for each change (preserves history)
- **Use Case**: When tracking full history is required (most common)
- **Implementation**:
  - Add effective date columns
  - Add current flag column
  - Add surrogate key

```
customer_id | name | address | start_date | end_date | current_flag
-----------------------------------------------------------
1001        | John | 123 Main| 2020-01-01 | 2022-05-31 | N
1001        | John | 456 Elm | 2022-06-01 | NULL       | Y
```

## 4. Type 3: Add New Column
- **Description**: Stores limited history in additional columns
- **Use Case**: When only tracking a few previous values is needed
- **Implementation**:
  - Add previous value columns
  - Typically tracks only 1-2 prior values

```
customer_id | name | current_address | previous_address
-------------------------------------------------------
1001        | John | 456 Elm St      | 123 Main St
```

## 5. Type 4: Separate History Table
- **Description**: Uses separate table for historical values
- **Use Case**: When current values are queried frequently without history
- **Implementation**:
  - Main table contains current values only
  - History table tracks all changes

**Current Table:**
```
customer_id | name | current_address
------------------------------------
1001        | John | 456 Elm St
```

**History Table:**
```
customer_id | address | effective_date
--------------------------------------
1001        | 123 Main| 2020-01-01
1001        | 456 Elm | 2022-06-01
```

## 6. Type 6: Hybrid (1+2+3)
- **Description**: Combines Type 1, 2, and 3 approaches
- **Use Case**: When needing both current values and comprehensive history
- **Implementation**:
  - Current values overwritten (Type 1)
  - New rows for changes (Type 2)
  - Previous values in columns (Type 3)

```
customer_id | name | address | prev_address | start_date | end_date | current_flag
----------------------------------------------------------------------------
1001        | John | 456 Elm | 123 Main     | 2020-01-01 | 2022-05-31 | N
1001        | John | 789 Oak | 456 Elm      | 2022-06-01 | NULL       | Y
```

## Implementation Considerations

### For Type 2 (Most Common Implementation):

**Key Columns to Add:**
- `effective_date`/`start_date`: When the record became valid
- `expiration_date`/`end_date`: When the record stopped being valid
- `current_flag`: Boolean/Y-N indicator (current record)
- `version_number`: Optional sequential version counter

**SQL Example (Creating Type 2 Dimension):**

```sql
CREATE TABLE dim_customer (
    customer_sk BIGINT PRIMARY KEY, -- surrogate key
    customer_id INT,                -- natural/business key
    name VARCHAR(100),
    address VARCHAR(200),
    start_date DATE,
    end_date DATE,
    current_flag CHAR(1),
    version_number INT
);
```

**ETL Process Logic:**
1. Identify changed records
2. Expire current records (set end_date = change date, current_flag = 'N')
3. Insert new records (start_date = change date, end_date = NULL, current_flag = 'Y')
4. Increment version numbers

## Choosing the Right SCD Type

| Type | History Kept | Storage Impact | Query Complexity | Common Use Cases |
|------|-------------|----------------|------------------|------------------|
| 0    | None        | Low            | Low              | Immutable attributes |
| 1    | None        | Low            | Low              | Corrections, unimportant changes |
| 2    | Complete    | High           | Moderate         | Most dimension tables |
| 3    | Limited     | Moderate       | Moderate         | When only last change matters |
| 4    | Complete    | High           | High             | Large dimensions with heavy current queries |
| 6    | Complete    | Highest        | Highest          | When needing all tracking methods |

## Best Practices

1. **Type 2 is the most widely used** for data warehousing
2. **Consider query patterns** - Type 2 requires more complex joins
3. **Monitor storage growth** - Type 2 can expand quickly for volatile dimensions
4. **Use surrogate keys** for Type 2 to avoid business key duplication
5. **Consider partitioning** Type 2 tables by current_flag for performance
6. **Document SCD types** used for each dimension in your data dictionary

Each SCD type serves different business requirements for tracking dimensional changes over time in data warehousing and analytics systems.