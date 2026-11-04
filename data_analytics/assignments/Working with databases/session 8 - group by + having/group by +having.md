# Module 2: Working with Databases - Session 8: GROUP BY & HAVING Clause

---

## 1. Create a table called `Orders` with columns: `order_id`, `user_id`, `payment_method`, and `amount`. Insert at least 8 sample records representing different users and payment methods (like UPI, Card, Wallet, COD).

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `order_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique order identifier)
  * `user_id`: `INT NOT NULL` (Identifier for the customer placing the order)
  * `payment_method`: `VARCHAR(50) NOT NULL` (e.g., 'UPI', 'Card', 'Wallet', 'COD')
  * `amount`: `DECIMAL(10, 2) NOT NULL` (Transaction value)

#### SQL Script:
```sql
-- 1. Create the Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
);

-- 2. Insert 9 sample records across diverse users and payment channels
INSERT INTO Orders (user_id, payment_method, amount) VALUES
(101, 'UPI', 250.00),
(102, 'Card', 850.00),
(103, 'UPI', 450.00),
(104, 'Wallet', 150.00),
(101, 'COD', 220.00),
(102, 'Wallet', 180.00),
(103, 'COD', 280.00),
(104, 'Card', 650.00),
(101, 'UPI', 500.00);

-- 3. Retrieve all records to verify insertion
SELECT * FROM Orders;
```

#### Output (Initial Table State):
| order_id | user_id | payment_method | amount |
|:---|:---|:---|:---|
| 1 | 101 | UPI | 250.00 |
| 2 | 102 | Card | 850.00 |
| 3 | 103 | UPI | 450.00 |
| 4 | 104 | Wallet | 150.00 |
| 5 | 101 | COD | 220.00 |
| 6 | 102 | Wallet | 180.00 |
| 7 | 103 | COD | 280.00 |
| 8 | 104 | Card | 650.00 |
| 9 | 101 | UPI | 500.00 |

*(Screenshot Recommended: phpMyAdmin / MySQL Workbench query window showing the inserted sample records)*

---

## 2. Write an SQL query to count how many orders were placed using each `payment_method` in the `Orders` table, similar to how Zomato shows payment breakdown in analytics.

### SOLUTION:
* **Concept:** 
  * `GROUP BY payment_method` gathers records with the same payment channel into aggregated buckets.
  * `COUNT(order_id)` computes the total number of orders belonging to each payment method.
  * `ORDER BY order_count DESC` sorts channels from most popular to least popular.

#### SQL Query:
```sql
SELECT 
    payment_method, 
    COUNT(order_id) AS order_count 
FROM Orders 
GROUP BY payment_method 
ORDER BY order_count DESC;
```

#### Output (Payment Breakdown):
| payment_method | order_count |
|:---|:---|
| UPI | 3 |
| Card | 2 |
| Wallet | 2 |
| COD | 2 |

---

## 3. Write an SQL query to find the total amount spent by each `user_id` in the `Orders` table. Display `user_id` and their total spend.

### SOLUTION:
* **Concept:** 
  * `GROUP BY user_id` groups transactions for each individual customer.
  * `SUM(amount)` aggregates all order amounts for that user.
  * `AS total_spend` labels the result for clean financial reporting.

#### SQL Query:
```sql
SELECT 
    user_id, 
    SUM(amount) AS total_spend 
FROM Orders 
GROUP BY user_id 
ORDER BY total_spend DESC;
```

#### Output (Total Spend per User):
| user_id | total_spend |
|:---|:---|
| 102 | 1030.00 |
| 101 | 970.00 |
| 104 | 800.00 |
| 103 | 730.00 |

*(Calculation Verification: User 101 = 250 + 220 + 500 = 970.00; User 102 = 850 + 180 = 1030.00).*

---

## 4. Write an SQL query to show only those payment methods where the average order amount is greater than 300, using `GROUP BY` and `HAVING`.
*(Hint: Use AVG(amount) in your HAVING clause.)*

### SOLUTION:
* **Concept:** 
  * The `WHERE` clause cannot filter aggregated results (like averages) because `WHERE` executes *before* grouping occurs.
  * The `HAVING` clause evaluates conditions *after* groups are created. `HAVING AVG(amount) > 300` filters out any payment group whose average order ticket is below or equal to 300.

#### Average Calculations by Channel:
* **Card:** $\frac{850 + 650}{2} = 750.00$ (> 300) &rarr; **Included**
* **UPI:** $\frac{250 + 450 + 500}{3} = 400.00$ (> 300) &rarr; **Included**
* **COD:** $\frac{220 + 280}{2} = 250.00$ (&le; 300) &rarr; *Excluded*
* **Wallet:** $\frac{150 + 180}{2} = 165.00$ (&le; 300) &rarr; *Excluded*

#### SQL Query:
```sql
SELECT 
    payment_method, 
    ROUND(AVG(amount), 2) AS average_amount 
FROM Orders 
GROUP BY payment_method 
HAVING AVG(amount) > 300;
```

#### Output:
| payment_method | average_amount |
|:---|:---|
| Card | 750.00 |
| UPI | 400.00 |

---

## 5. Explain the difference between `WHERE` and `HAVING` by giving one example query for each, using the `Orders` table. Your examples should show a scenario where `WHERE` and `HAVING` filter different things.

### SOLUTION:

### Key Differences Between `WHERE` and `HAVING`:

| Feature | `WHERE` Clause | `HAVING` Clause |
|:---|:---|:---|
| **What it filters** | Filters individual **rows** before grouping | Filters **groups/aggregates** after grouping |
| **Use with Aggregates** | **Cannot** use aggregate functions (`SUM`, `AVG`, `COUNT`) | **Can** evaluate aggregate functions (`AVG(amount) > 300`) |
| **Execution Order** | Executes **before** `GROUP BY` | Executes **after** `GROUP BY` |
| **Dependency** | Can be used without `GROUP BY` | Used with `GROUP BY` (or table-level aggregation) |

---

### Example Queries Using `Orders`:

#### **Scenario 1: Using `WHERE` to Filter Individual Orders (Row-Level)**
Suppose we want to find all individual orders with high order value (`amount >= 500`). `WHERE` inspects each row before any grouping:

```sql
-- Filters individual transaction rows where amount is 500 or higher
SELECT order_id, user_id, payment_method, amount 
FROM Orders 
WHERE amount >= 500;
```

**Output:**
| order_id | user_id | payment_method | amount |
|:---|:---|:---|:---|
| 2 | 102 | Card | 850.00 |
| 8 | 104 | Card | 650.00 |
| 9 | 101 | UPI | 500.00 |

---

#### **Scenario 2: Using `HAVING` to Filter Grouped Categories (Aggregate-Level)**
Suppose we want to find payment methods that generated a total transaction volume (`SUM(amount)`) exceeding 1000. `HAVING` filters the summary groups after `GROUP BY` sums them:

```sql
-- Filters summarized groups based on total sales volume
SELECT 
    payment_method, 
    SUM(amount) AS total_revenue 
FROM Orders 
GROUP BY payment_method 
HAVING SUM(amount) > 1000;
```

**Output:**
| payment_method | total_revenue |
|:---|:---|
| Card | 1500.00 |
| UPI | 1200.00 |

*(Note: COD ($500.00) and Wallet ($330.00) are excluded because their aggregated sum does not exceed 1000).*

---

#### **Combined Best-Practice Query (Demonstrating Both Together):**
```sql
SELECT 
    payment_method, 
    COUNT(order_id) AS total_orders, 
    SUM(amount) AS total_amount 
FROM Orders 
WHERE amount > 200            -- Row filter: Only consider orders above 200
GROUP BY payment_method 
HAVING SUM(amount) >= 700;    -- Group filter: Only keep channels generating at least 700
```
