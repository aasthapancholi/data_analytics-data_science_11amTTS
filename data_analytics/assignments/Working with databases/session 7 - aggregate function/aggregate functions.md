# Module 2: Working with Databases - Session 7: Aggregate Functions (COUNT, AVG, MAX, MIN, SUM)

---

## 1. Create a table called `Orders` with columns: `order_id`, `user_name`, `total_amount`, and `order_date`. Insert 5 sample rows with different users and order amounts, including at least one NULL value for `total_amount`.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `order_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique identifier for each transaction)
  * `user_name`: `VARCHAR(100) NOT NULL` (Customer name)
  * `total_amount`: `DECIMAL(10, 2) NULL` (Order billing amount; nullable to handle pending/cancelled orders)
  * `order_date`: `DATE NOT NULL` (Date when order was placed)

#### SQL Script:
```sql
-- 1. Create the Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_name VARCHAR(100) NOT NULL,
    total_amount DECIMAL(10, 2) NULL,
    order_date DATE NOT NULL
);

-- 2. Insert 5 sample records (including at least one NULL total_amount)
INSERT INTO Orders (user_name, total_amount, order_date) VALUES
('Aarav', 1250.00, '2026-10-01'),
('Diya', 2499.50, '2026-10-03'),
('Aarav', 750.00, '2026-10-05'),
('Rohan', NULL, '2026-10-07'), -- Pending payment / cancelled order
('Sneha', 3200.00, '2026-10-10');

-- 3. Retrieve all records to verify insertion
SELECT * FROM Orders;
```

#### Output (Initial Table State):
| order_id | user_name | total_amount | order_date |
|:---|:---|:---|:---|
| 1 | Aarav | 1250.00 | 2026-10-01 |
| 2 | Diya | 2499.50 | 2026-10-03 |
| 3 | Aarav | 750.00 | 2026-10-05 |
| 4 | Rohan | *NULL* | 2026-10-07 |
| 5 | Sneha | 3200.00 | 2026-10-10 |

*(Screenshot Recommended: phpMyAdmin / MySQL Workbench query window showing the 5 inserted records including the NULL value)*

---

## 2. Write a SQL query to count how many orders were placed by each user in the `Orders` table, displaying `user_name` and the number of orders as `order_count`.

### SOLUTION:
* **Concept:** 
  * The `GROUP BY` clause groups rows that have the same value in specified columns (`user_name`).
  * `COUNT(order_id)` or `COUNT(*)` counts the total number of orders placed by each distinct user.
  * `AS order_count` assigns a readable alias to the aggregated output column.

#### SQL Query:
```sql
SELECT 
    user_name, 
    COUNT(order_id) AS order_count 
FROM Orders 
GROUP BY user_name;
```

#### Output:
| user_name | order_count |
|:---|:---|
| Aarav | 2 |
| Diya | 1 |
| Rohan | 1 |
| Sneha | 1 |

*(Note: Even though Rohan's order amount was `NULL`, `COUNT(order_id)` properly records that 1 order transaction exists for Rohan).*

---

## 3. Write a SQL query to calculate the average `total_amount` of all orders in the `Orders` table, making sure to ignore any NULL values.

### SOLUTION:
* **Concept:**
  * By SQL standard definition, aggregate functions like `AVG()` **automatically ignore `NULL` values** during calculation.
  * In this dataset, `AVG(total_amount)` sums the non-null values ($1250.00 + 2499.50 + 750.00 + 3200.00 = 7699.50$) and divides by $4$ (not $5$), giving $1924.875$.
  * We can also explicitly filter with `WHERE total_amount IS NOT NULL` and use `ROUND()` for currency formatting.

#### SQL Query:
```sql
SELECT 
    ROUND(AVG(total_amount), 2) AS average_order_amount 
FROM Orders 
WHERE total_amount IS NOT NULL;
```

#### Output:
| average_order_amount |
|:---|
| 1924.88 |

---

## 4. Suppose you are building a Flipkart-style dashboard: Write a SQL query to find the highest and lowest order amounts (`MAX` and `MIN`) from the `Orders` table, and display both values in a single result row.

### SOLUTION:
* **Concept:** Multiple aggregate functions can be executed together in the same `SELECT` statement to produce consolidated KPI metrics on a single row.
  * `MAX(total_amount)` finds the highest amount.
  * `MIN(total_amount)` finds the lowest amount (automatically ignoring `NULL`).

#### SQL Query:
```sql
SELECT 
    MAX(total_amount) AS highest_order_amount, 
    MIN(total_amount) AS lowest_order_amount 
FROM Orders;
```

#### Output:
| highest_order_amount | lowest_order_amount |
|:---|:---|
| 3200.00 | 750.00 |

---

## 5. Write a SQL query to calculate the total sales (`SUM` of `total_amount`) for all orders, but only include orders where `total_amount` is not NULL.
*(Hint: Use a WHERE clause to filter out NULL values before applying the SUM function.)*

### SOLUTION:
* **Concept:**
  * While `SUM()` natively ignores `NULL` values, applying `WHERE total_amount IS NOT NULL` explicitly filters out invalid or pending records before aggregation occurs.
  * In SQL, we use `IS NOT NULL` (never `!= NULL` or `<> NULL` because comparisons with `NULL` evaluate to `UNKNOWN`).

#### SQL Query:
```sql
SELECT 
    SUM(total_amount) AS total_sales 
FROM Orders 
WHERE total_amount IS NOT NULL;
```

#### Output:
| total_sales |
|:---|
| 7699.50 |
