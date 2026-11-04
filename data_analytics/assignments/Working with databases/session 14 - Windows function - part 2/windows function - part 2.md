# Module 2: Working with Databases - Session 14: Window Functions - Part 2 (LAG, LEAD, Window Framing & Moving Averages)

---

## 1. Create a table called `Orders` with columns: `order_id`, `user_id`, `order_date`, and `total_amount`. Insert at least 7 sample rows representing different users and dates, similar to how food orders appear in Zomato or Swiggy.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Relational Schema Design for Time-Series Transactional Data
* **Table Design:**
  * `Orders`:
    * `order_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique order identifier)
    * `user_id`: `INT NOT NULL` (Zomato/Swiggy customer ID)
    * `order_date`: `DATE NOT NULL` (Date the meal was ordered)
    * `total_amount`: `DECIMAL(10, 2) NOT NULL` (Order bill total in INR)

#### SQL Setup Script:
```sql
-- 1. Create Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

-- 2. Insert 9 sample orders across multiple users and dates (Swiggy / Zomato order history)
INSERT INTO Orders (user_id, order_date, total_amount) VALUES
-- User 101 (Aarav): Regular foodie ordering throughout October
(101, '2026-10-01', 350.00),
(101, '2026-10-05', 450.00),
(101, '2026-10-12', 600.00),
(101, '2026-10-20', 800.00),

-- User 102 (Diya): Steady spender with 3 orders
(102, '2026-10-02', 250.00),
(102, '2026-10-10', 500.00),
(102, '2026-10-18', 750.00),

-- User 103 (Rohan): High-value party orders
(103, '2026-10-04', 1200.00),
(103, '2026-10-15', 950.00);

-- 3. Retrieve all records to verify insertion
SELECT * FROM Orders ORDER BY user_id, order_date;
```

#### Output (Initial Table State):
| order_id | user_id | order_date | total_amount |
|:---|:---|:---|:---|
| 1 | 101 | 2026-10-01 | 350.00 |
| 2 | 101 | 2026-10-05 | 450.00 |
| 3 | 101 | 2026-10-12 | 600.00 |
| 4 | 101 | 2026-10-20 | 800.00 |
| 5 | 102 | 2026-10-02 | 250.00 |
| 6 | 102 | 2026-10-10 | 500.00 |
| 7 | 102 | 2026-10-18 | 750.00 |
| 8 | 103 | 2026-10-04 | 1200.00 |
| 9 | 103 | 2026-10-15 | 950.00 |

---

## 2. Write a SQL query using the `LAG()` function to show each user's `order_id`, `order_date`, and the `total_amount` of their previous order (if any), ordered by user and date.
*(Hint: Use PARTITION BY user_id and ORDER BY order_date in your window function.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Value Window Function `LAG()` with Partitioning

---

### SQL Query:
```sql
SELECT 
    user_id,
    order_id,
    order_date,
    total_amount AS current_order_amount,
    LAG(total_amount, 1) OVER (
        PARTITION BY user_id 
        ORDER BY order_date ASC
    ) AS previous_order_amount
FROM Orders
ORDER BY user_id ASC, order_date ASC;
```

---

### How `LAG()` Works:
1. **Window Definition:**
   * `PARTITION BY user_id` divides the table into isolated partitions per customer.
   * `ORDER BY order_date ASC` sequences orders chronologically.
2. **Accessing Prior Rows:**
   * `LAG(total_amount, 1)` reaches back **1 row** within the current user's partition and retrieves that row's `total_amount`.
3. **Handling the First Order (`NULL`):**
   * For the earliest order placed by each user, there is no preceding row. `LAG()` correctly returns `NULL`.
   * *(Optional Industry Tip: You can supply a third argument as a fallback default, e.g., `LAG(total_amount, 1, 0.00)` to return `0.00` instead of `NULL`.)*

#### Output:
| user_id | order_id | order_date | current_order_amount | previous_order_amount |
|:---|:---|:---|:---|:---|
| 101 | 1 | 2026-10-01 | 350.00 | *NULL* |
| 101 | 2 | 2026-10-05 | 450.00 | 350.00 |
| 101 | 3 | 2026-10-12 | 600.00 | 450.00 |
| 101 | 4 | 2026-10-20 | 800.00 | 600.00 |
| 102 | 5 | 2026-10-02 | 250.00 | *NULL* |
| 102 | 6 | 2026-10-10 | 500.00 | 250.00 |
| 102 | 7 | 2026-10-18 | 750.00 | 500.00 |
| 103 | 8 | 2026-10-04 | 1200.00 | *NULL* |
| 103 | 9 | 2026-10-15 | 950.00 | 1200.00 |

---

## 3. Using the same `Orders` table, write a SQL query with the `LEAD()` function to display each `order_id`, `order_date`, and the next order's `total_amount` for the same user.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Value Window Function `LEAD()` looking ahead chronologically

---

### SQL Query:
```sql
SELECT 
    user_id,
    order_id,
    order_date,
    total_amount AS current_order_amount,
    LEAD(total_amount, 1) OVER (
        PARTITION BY user_id 
        ORDER BY order_date ASC
    ) AS next_order_amount
FROM Orders
ORDER BY user_id ASC, order_date ASC;
```

---

### How `LEAD()` Works & Analytical Use Cases:
1. **Peeking Ahead in Time:**
   * While `LAG()` looks backward, `LEAD(total_amount, 1)` looks ahead **1 row** forward within the user's partition.
2. **Terminal Row (`NULL`):**
   * For each user's most recent (latest) order, there is no subsequent order, so `LEAD()` returns `NULL`.
3. **Real-World Analytics Applications on Swiggy / Zomato:**
   * **Churn Analysis:** If `LEAD(order_date)` is `NULL` for over 60 days, the user is flagged as at-risk of churning.
   * **Purchase Frequency:** Analysts use `DATEDIFF(LEAD(order_date) OVER(...), order_date)` to calculate the average number of days between repeat food deliveries.

#### Output:
| user_id | order_id | order_date | current_order_amount | next_order_amount |
|:---|:---|:---|:---|:---|
| 101 | 1 | 2026-10-01 | 350.00 | 450.00 |
| 101 | 2 | 2026-10-05 | 450.00 | 600.00 |
| 101 | 3 | 2026-10-12 | 600.00 | 800.00 |
| 101 | 4 | 2026-10-20 | 800.00 | *NULL* |
| 102 | 5 | 2026-10-02 | 250.00 | 500.00 |
| 102 | 6 | 2026-10-10 | 500.00 | 750.00 |
| 102 | 7 | 2026-10-18 | 750.00 | *NULL* |
| 103 | 8 | 2026-10-04 | 1200.00 | 950.00 |
| 103 | 9 | 2026-10-15 | 950.00 | *NULL* |

---

## 4. Write a SQL query to calculate the running total of `total_amount` for each user, showing `order_id`, `order_date`, `total_amount`, and a column `running_total` that accumulates the sum as you move through each user's orders.
*(Hint: Use SUM(total_amount) OVER (PARTITION BY user_id ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW).)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Window Aggregate Function with Window Frame Specification (`ROWS BETWEEN`)

---

### SQL Query:
```sql
SELECT 
    user_id,
    order_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY user_id 
        ORDER BY order_date ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM Orders
ORDER BY user_id ASC, order_date ASC;
```

---

### Anatomy of the Window Frame:
1. **`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`:**
   * **`UNBOUNDED PRECEDING`:** Start summing from the very first row of the partition.
   * **`CURRENT ROW`:** Stop summing at the row currently being processed.
   * **Result:** A true cumulative/running total that builds up row by row.
2. **Why specify `ROWS BETWEEN` explicitly?**
   * If omitted, standard ANSI SQL defaults to `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. If two orders occurred on the identical timestamp/date, `RANGE` sums tied rows together at once, whereas `ROWS` evaluates strictly row-by-row, ensuring accurate progressive accumulation.

#### Step-by-Step Cumulative Trace for User 101:
* Row 1 (2026-10-01): $350.00 \rightarrow \mathbf{350.00}$
* Row 2 (2026-10-05): $350.00 + 450.00 \rightarrow \mathbf{800.00}$
* Row 3 (2026-10-12): $800.00 + 600.00 \rightarrow \mathbf{1400.00}$
* Row 4 (2026-10-20): $1400.00 + 800.00 \rightarrow \mathbf{2200.00}$

#### Output:
| user_id | order_id | order_date | total_amount | running_total |
|:---|:---|:---|:---|:---|
| 101 | 1 | 2026-10-01 | 350.00 | 350.00 |
| 101 | 2 | 2026-10-05 | 450.00 | 800.00 |
| 101 | 3 | 2026-10-12 | 600.00 | 1400.00 |
| 101 | 4 | 2026-10-20 | 800.00 | 2200.00 |
| 102 | 5 | 2026-10-02 | 250.00 | 250.00 |
| 102 | 6 | 2026-10-10 | 500.00 | 750.00 |
| 102 | 7 | 2026-10-18 | 750.00 | 1500.00 |
| 103 | 8 | 2026-10-04 | 1200.00 | 1200.00 |
| 103 | 9 | 2026-10-15 | 950.00 | 2150.00 |

---

## 5. Write a SQL query to calculate a 3-order moving average of `total_amount` for each user, showing `order_id`, `order_date`, `total_amount`, and `moving_avg` columns.
*(Constraint: Use SUM() OVER() with ROWS BETWEEN 2 PRECEDING AND CURRENT ROW to compute the moving average.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Moving/Rolling Average over a Sliding Window Frame using `SUM() OVER()`

---

### SQL Queries:

#### **Primary Query: Satisfying the Constraint using `SUM() OVER()`**
> *To compute the true average dynamically across the initial window (handling 1st and 2nd orders where fewer than 3 orders exist), we divide `SUM()` by the window row count using `COUNT()`:*

```sql
SELECT 
    user_id,
    order_id,
    order_date,
    total_amount,
    ROUND(
        SUM(total_amount) OVER (
            PARTITION BY user_id 
            ORDER BY order_date ASC
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ) 
        / 
        COUNT(order_id) OVER (
            PARTITION BY user_id 
            ORDER BY order_date ASC
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS moving_avg
FROM Orders
ORDER BY user_id ASC, order_date ASC;
```

---

#### **Alternative Variant: Using Native `AVG()` Window Function**
> *In production systems where the `SUM()` constraint is not mandated, native `AVG()` directly implements the identical sliding frame:*

```sql
SELECT 
    user_id,
    order_id,
    order_date,
    total_amount,
    ROUND(
        AVG(total_amount) OVER (
            PARTITION BY user_id 
            ORDER BY order_date ASC
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS moving_avg
FROM Orders
ORDER BY user_id ASC, order_date ASC;
```

---

### Step-by-Step Sliding Window Trace (User 101):
1. **Order 1 (Amount: 350.00):**
   * Frame (Current): $[350.00]$
   * Calculation: $350.00 / 1 = \mathbf{350.00}$
2. **Order 2 (Amount: 450.00):**
   * Frame (1 Preceding + Current): $[350.00, 450.00]$
   * Calculation: $(350.00 + 450.00) / 2 = 800.00 / 2 = \mathbf{400.00}$
3. **Order 3 (Amount: 600.00):**
   * Frame (2 Preceding + Current): $[350.00, 450.00, 600.00]$
   * Calculation: $(350.00 + 450.00 + 600.00) / 3 = 1400.00 / 3 = \mathbf{466.67}$
4. **Order 4 (Amount: 800.00):**
   * Frame slides forward! Order 1 drops out of the frame: $[450.00, 600.00, 800.00]$
   * Calculation: $(450.00 + 600.00 + 800.00) / 3 = 1850.00 / 3 = \mathbf{616.67}$

#### Output:
| user_id | order_id | order_date | total_amount | moving_avg |
|:---|:---|:---|:---|:---|
| 101 | 1 | 2026-10-01 | 350.00 | 350.00 |
| 101 | 2 | 2026-10-05 | 450.00 | 400.00 |
| 101 | 3 | 2026-10-12 | 600.00 | 466.67 |
| 101 | 4 | 2026-10-20 | 800.00 | 616.67 |
| 102 | 5 | 2026-10-02 | 250.00 | 250.00 |
| 102 | 6 | 2026-10-10 | 500.00 | 375.00 |
| 102 | 7 | 2026-10-18 | 750.00 | 500.00 |
| 103 | 8 | 2026-10-04 | 1200.00 | 1200.00 |
| 103 | 9 | 2026-10-15 | 950.00 | 1075.00 |
