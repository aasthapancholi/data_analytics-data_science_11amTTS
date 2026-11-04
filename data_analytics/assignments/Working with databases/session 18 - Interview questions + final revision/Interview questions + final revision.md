# Module 2: Working with Databases - Session 18: SQL Interview Masterclass & Final Revision

---

## 1. Write an SQL query to display the total number of songs uploaded by each artist from a table `songs` (columns: `song_id`, `artist_name`, `title`) and show only those artists who have uploaded more than 3 songs.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Interview Concept:** `GROUP BY` Aggregation vs. `HAVING` Clause Filtering
* **Table Design:**
  * `songs`:
    * `song_id`: `INT PRIMARY KEY AUTO_INCREMENT`
    * `artist_name`: `VARCHAR(100) NOT NULL`
    * `title`: `VARCHAR(150) NOT NULL`

#### SQL Setup Script:
```sql
-- 1. Create songs table
CREATE TABLE songs (
    song_id INT PRIMARY KEY AUTO_INCREMENT,
    artist_name VARCHAR(100) NOT NULL,
    title VARCHAR(150) NOT NULL
);

-- 2. Insert sample tracks across various artists
INSERT INTO songs (artist_name, title) VALUES
-- Arijit Singh: 4 songs (Should appear in result)
('Arijit Singh', 'Kesariya'),
('Arijit Singh', 'Tum Hi Ho'),
('Arijit Singh', 'Channa Mereya'),
('Arijit Singh', 'Apna Bana Le'),

-- The Weeknd: 4 songs (Should appear in result)
('The Weeknd', 'Blinding Lights'),
('The Weeknd', 'Starboy'),
('The Weeknd', 'Save Your Tears'),
('The Weeknd', 'Die For You'),

-- Diljit Dosanjh: 3 songs (Should NOT appear: exactly 3 is not > 3)
('Diljit Dosanjh', 'Lover'),
('Diljit Dosanjh', 'Born to Shine'),
('Diljit Dosanjh', 'GOAT'),

-- Dua Lipa: 2 songs (Should NOT appear)
('Dua Lipa', 'Levitating'),
('Dua Lipa', 'Don\'t Start Now');
```

---

### SQL Query:
```sql
SELECT 
    artist_name,
    COUNT(song_id) AS total_songs
FROM songs
GROUP BY artist_name
HAVING COUNT(song_id) > 3
ORDER BY total_songs DESC, artist_name ASC;
```

---

### Critical Interview Question: *Why `HAVING` and not `WHERE`?*
> **SQL Order of Execution:**
> 1. `FROM`
> 2. `WHERE` *(Row-level filter before grouping)*
> 3. `GROUP BY` *(Grouping rows together)*
> 4. `HAVING` *(Group-level filter on aggregated results)*
> 5. `SELECT`
>
> The `WHERE` clause filters individual records **before** groups are formed. Because `COUNT(song_id)` only exists **after** rows are grouped, filtering on an aggregate function must be placed in the **`HAVING`** clause. Attempting `WHERE COUNT(song_id) > 3` causes a fatal SQL syntax error.

#### Output:
| artist_name | total_songs |
|:---|:---|
| Arijit Singh | 4 |
| The Weeknd | 4 |

---

## 2. Given two tables, `orders` (`order_id`, `user_id`, `amount`) and `users` (`user_id`, `username`), write a SQL JOIN query to display each username along with their total order amount.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Interview Concept:** Multi-Table Relational JOINs with Aggregation (`LEFT JOIN` vs `INNER JOIN`)
* **Table Design:**
  * `users`: `user_id` (PK), `username`
  * `orders`: `order_id` (PK), `user_id` (FK), `amount`

#### SQL Setup Script:
```sql
-- 1. Create users table
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL
);

-- 2. Create orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- 3. Insert Users (Notice: Pooja has 0 orders)
INSERT INTO users (username) VALUES
('Aarav'),
('Diya'),
('Rohan'),
('Pooja');

-- 4. Insert Orders
INSERT INTO orders (user_id, amount) VALUES
(1, 850.00),
(1, 450.00),
(2, 1200.00),
(2, 650.00),
(2, 300.00),
(3, 950.00);
```

---

### SQL Queries:

#### **Method A: `LEFT JOIN` with `COALESCE()` (Senior Data Analyst Best Practice)**
> *In production analytics, we want to include all registered users even if they haven't made a purchase yet, displaying `0.00` rather than excluding them:*

```sql
SELECT 
    u.user_id,
    u.username,
    COALESCE(SUM(o.amount), 0.00) AS total_order_amount
FROM users u
LEFT JOIN orders o 
    ON u.user_id = o.user_id
GROUP BY u.user_id, u.username
ORDER BY total_order_amount DESC;
```

#### **Method B: Standard `INNER JOIN` (Excludes Users with Zero Orders)**
```sql
SELECT 
    u.user_id,
    u.username,
    SUM(o.amount) AS total_order_amount
FROM users u
INNER JOIN orders o 
    ON u.user_id = o.user_id
GROUP BY u.user_id, u.username
ORDER BY total_order_amount DESC;
```

---

### Interview Nuance:
* When using `LEFT JOIN`, users without orders produce `NULL` for `SUM(o.amount)`. Using `COALESCE(SUM(o.amount), 0.00)` replaces `NULL` with `0.00`, ensuring clean, production-grade output for BI dashboards.

#### Output (Method A):
| user_id | username | total_order_amount |
|:---|:---|:---|
| 2 | Diya | 2150.00 |
| 1 | Aarav | 1300.00 |
| 3 | Rohan | 950.00 |
| 4 | Pooja | 0.00 |

---

## 3. Write a SQL subquery to find the names of all restaurants from a `restaurants` table (`id`, `name`, `rating`) whose rating is higher than the average rating of all restaurants.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Interview Concept:** Non-Correlated Scalar Subquery in the `WHERE` Clause
* **Table Design:**
  * `restaurants`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT`
    * `name`: `VARCHAR(100) NOT NULL`
    * `rating`: `DECIMAL(3, 2) NOT NULL`

#### SQL Setup Script:
```sql
-- 1. Create restaurants table
CREATE TABLE restaurants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    rating DECIMAL(3, 2) NOT NULL
);

-- 2. Insert sample restaurants
INSERT INTO restaurants (name, rating) VALUES
('Swati Snacks', 4.50),
('Gordhan Thal', 4.10),
('Little Italy', 4.70),
('Sale & Pepe', 3.90),
('Manek Chowk Street Eats', 3.80),
('Agashiye', 4.60);
```

---

### SQL Query:
```sql
SELECT 
    id,
    name,
    rating
FROM restaurants
WHERE rating > (
    -- Inner Scalar Subquery: Computes the overall average rating of all restaurants
    SELECT AVG(rating)
    FROM restaurants
)
ORDER BY rating DESC;
```

---

### Step-by-Step Execution Breakdown:
1. **Subquery Execution:**
   * Computes the arithmetic mean:
     $$\text{Average} = \frac{4.50 + 4.10 + 4.70 + 3.90 + 3.80 + 4.60}{6} = \frac{25.60}{6} \approx \mathbf{4.27}$$
2. **Outer Query Filtering:**
   * Checks every row against the scalar threshold $4.27$:
     * *Little Italy* ($4.70 > 4.27$) $\rightarrow$ **Included**
     * *Agashiye* ($4.60 > 4.27$) $\rightarrow$ **Included**
     * *Swati Snacks* ($4.50 > 4.27$) $\rightarrow$ **Included**
     * *Gordhan Thal* ($4.10 \le 4.27$) $\rightarrow$ Excluded
     * *Sale & Pepe* ($3.90 \le 4.27$) $\rightarrow$ Excluded
     * *Manek Chowk Street Eats* ($3.80 \le 4.27$) $\rightarrow$ Excluded

#### Output:
| id | name | rating |
|:---|:---|:---|
| 3 | Little Italy | 4.70 |
| 6 | Agashiye | 4.60 |
| 1 | Swati Snacks | 4.50 |

---

## 4. Using a `transactions` table (`id`, `user_id`, `amount`, `transaction_date`), write a SQL query with a window function to display each user's transaction amount and their running total (cumulative sum) ordered by `transaction_date`.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Interview Concept:** Cumulative Window Aggregate Function with Partitioning and Framing
* **Table Design:**
  * `transactions`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT`
    * `user_id`: `INT NOT NULL`
    * `amount`: `DECIMAL(10, 2) NOT NULL`
    * `transaction_date`: `DATE NOT NULL`

#### SQL Setup Script:
```sql
-- 1. Create transactions table
CREATE TABLE transactions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    transaction_date DATE NOT NULL
);

-- 2. Insert sample transactions across multiple users
INSERT INTO transactions (user_id, amount, transaction_date) VALUES
-- User 101: 3 transactions in October
(101, 500.00, '2026-10-01'),
(101, 750.00, '2026-10-05'),
(101, 1200.00, '2026-10-12'),

-- User 102: 3 transactions in October
(102, 1000.00, '2026-10-02'),
(102, 400.00, '2026-10-07'),
(102, 600.00, '2026-10-15');
```

---

### SQL Query:
```sql
SELECT 
    user_id,
    id AS transaction_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id 
        ORDER BY transaction_date ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM transactions
ORDER BY user_id ASC, transaction_date ASC;
```

---

### Window Function Anatomy:
* **`PARTITION BY user_id`:** Isolates each user's financial ledger. The cumulative sum resets to the initial deposit when crossing over to the next user.
* **`ORDER BY transaction_date ASC`:** Enforces sequential chronological order.
* **`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`:** Instructs the database engine to sum from the very first transaction of that user up to the current row being scanned.

#### Output:
| user_id | transaction_id | transaction_date | amount | running_total |
|:---|:---|:---|:---|:---|
| 101 | 1 | 2026-10-01 | 500.00 | 500.00 |
| 101 | 2 | 2026-10-05 | 750.00 | 1250.00 |
| 101 | 3 | 2026-10-12 | 1200.00 | 2450.00 |
| 102 | 4 | 2026-10-02 | 1000.00 | 1000.00 |
| 102 | 5 | 2026-10-07 | 400.00 | 1400.00 |
| 102 | 6 | 2026-10-15 | 600.00 | 2000.00 |

---

## 5. List two optimizations you would apply to speed up a query that filters Flipkart products by category and price, and briefly explain how each helps.
*(Hint: Think about indexes and query structure.)*

### SOLUTION:

Suppose we are optimizing a high-traffic e-commerce query executed millions of times per day on Flipkart:

```sql
-- Target Query:
SELECT product_id, product_name, brand, price, rating
FROM products
WHERE category_id = 5 
  AND price BETWEEN 500 AND 2000
ORDER BY price ASC;
```

---

### Optimization 1: Create a Composite B-Tree Index on `(category_id, price)`

#### **SQL Implementation:**
```sql
CREATE INDEX idx_products_category_price ON products (category_id, price);
```

#### **How it helps (The Mechanics):**
1. **The Leftmost Prefix Rule:**
   * A B-Tree index sorts data first by the first column (`category_id`) and then by the second column (`price`).
2. **Instant Exact Matching + Range Scan:**
   * The database engine uses the first key (`category_id = 5`) to instantly jump via binary search to the exact branch in the index tree where Category 5 lives.
   * Within that category branch, products are **already pre-sorted by `price`**. The engine simply scans the range from `500` to `2000` linearly in index memory without examining a single row from any other category.
3. **Eliminates `filesort` for `ORDER BY price`:**
   * Because the index leaf nodes are already physically ordered by `price ASC`, the database completely skips an expensive in-memory or on-disk sort operation (`Using filesort`), returning sorted results instantly.

---

### Optimization 2: Query Structure Refinement & Covering Index (Avoid `SELECT *` & Eliminate Clustered Lookups)

#### **SQL Implementation (Covering Index):**
```sql
-- Extending the index to include projected columns:
CREATE INDEX idx_products_covering ON products (category_id, price, product_name, rating);

-- Query strictly projecting required columns:
SELECT product_id, product_name, price, rating
FROM products
WHERE category_id = 5 
  AND price BETWEEN 500.00 AND 2000.00
ORDER BY price ASC;
```

#### **How it helps (The Mechanics):**
1. **Elimination of Clustered Index Lookups (Bookmark Lookups):**
   * In standard secondary indexes, the index only stores the indexed keys and the Primary Key (`product_id`). To fetch remaining columns like `product_name` and `rating`, the engine must execute random disk I/O lookups into the main table for every matching row.
   * With a **Covering Index**, 100% of the columns requested in the `SELECT`, `WHERE`, and `ORDER BY` clauses reside directly in the index leaf pages (`Using index` in MySQL `EXPLAIN`). The query is satisfied purely in RAM cache without touching the physical disk table.
2. **Bandwidth & Buffer Cache Optimization:**
   * Avoiding wildcard `SELECT *` dramatically reduces network payload size and prevents polluting the database server's InnoDB buffer pool with unused text/blob fields.

---

### Bonus Optimization Summary Table (Interview Cheat Sheet):

| Optimization Technique | Database Mechanism | Bottleneck Eliminated |
|:---|:---|:---|
| **Composite Index `(category_id, price)`** | B-Tree point lookup + contiguous leaf range scan | Eliminates Full Table Scan (`ALL`) & `filesort` |
| **Covering Index (`Using index`)** | Direct RAM cache index resolution | Eliminates Random Disk I/O (Table Lookups) |
| **Table Partitioning (by `category_id`)** | Partition Pruning (reads only target partition file) | Drastically reduces table size examined |
