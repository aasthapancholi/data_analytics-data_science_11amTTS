# Module 2: Working with Databases - Session 16: SQL Capstone Project (Sales & Performance Analysis)

---

## 1. Import a CSV file of food delivery orders (with columns like `order_id`, `restaurant_name`, `customer_name`, `order_amount`, `order_date`) into a new SQL table named `FoodOrders` using your database tool of choice.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Relational Schema DDL, CSV Data Ingestion & Data Loading Workflows

---

### Step 1: SQL Table Creation (DDL)
```sql
-- 1. Create database (if not already existing)
CREATE DATABASE IF NOT EXISTS FoodieSalesDB;
USE FoodieSalesDB;

-- 2. Create FoodOrders table schema
CREATE TABLE FoodOrders (
    order_id INT PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    order_amount DECIMAL(10, 2) NOT NULL,
    order_date DATE NOT NULL
);
```

---

### Step 2: The Raw CSV Data (`FoodOrders.csv`)
If you are creating a CSV file to import, create a file named `FoodOrders.csv` with the following content:

```csv
order_id,restaurant_name,customer_name,order_amount,order_date
1001,Little Italy,Aarav Sharma,850.00,2026-10-01
1002,Swati Snacks,Diya Patel,450.00,2026-10-02
1003,La Pino'z Pizza,Aarav Sharma,600.00,2026-10-04
1004,Behrouz Biryani,Rohan Verma,1200.00,2026-10-05
1005,Swati Snacks,Pooja Mehta,350.00,2026-10-08
1006,Behrouz Biryani,Diya Patel,950.00,2026-10-10
1007,Theobroma,Aarav Sharma,1200.00,2026-10-12
1008,La Pino'z Pizza,Rohan Verma,800.00,2026-10-15
1009,Little Italy,Diya Patel,1400.00,2026-10-18
1010,La Pino'z Pizza,Pooja Mehta,400.00,2026-10-20
1011,Swati Snacks,Karan Singhania,300.00,2026-10-22
```

---

### Step 3: Two Methods to Ingest CSV into SQL

#### **Method A: Using phpMyAdmin GUI (Recommended for Beginners in XAMPP)**
1. Open **phpMyAdmin** in your browser (`http://localhost/phpmyadmin`).
2. Select the `FoodieSalesDB` database from the left sidebar and click on the `FoodOrders` table.
3. Click the **"Import"** tab in the top navigation bar.
4. Under **"File to import"**, click **Choose File** and select your `FoodOrders.csv`.
5. Under **"Format"**, choose **CSV** (or *CSV using LOAD DATA*).
6. Set **"Number of rows to skip"** to `1` (to skip the header line).
7. Under **"Columns separated with:"**, verify comma (`,`) is set.
8. Click **Import / Go**. phpMyAdmin will insert all records into the table.

#### **Method B: Direct SQL Setup Script (Immediate Execution in SQL Query Box)**
> *If you want to quickly populate the table without navigating file menus, execute this `INSERT` statement:*

```sql
INSERT INTO FoodOrders (order_id, restaurant_name, customer_name, order_amount, order_date) VALUES
(1001, 'Little Italy', 'Aarav Sharma', 850.00, '2026-10-01'),
(1002, 'Swati Snacks', 'Diya Patel', 450.00, '2026-10-02'),
(1003, 'La Pino\'z Pizza', 'Aarav Sharma', 600.00, '2026-10-04'),
(1004, 'Behrouz Biryani', 'Rohan Verma', 1200.00, '2026-10-05'),
(1005, 'Swati Snacks', 'Pooja Mehta', 350.00, '2026-10-08'),
(1006, 'Behrouz Biryani', 'Diya Patel', 950.00, '2026-10-10'),
(1007, 'Theobroma', 'Aarav Sharma', 1200.00, '2026-10-12'),
(1008, 'La Pino\'z Pizza', 'Rohan Verma', 800.00, '2026-10-15'),
(1009, 'Little Italy', 'Diya Patel', 1400.00, '2026-10-18'),
(1010, 'La Pino\'z Pizza', 'Pooja Mehta', 400.00, '2026-10-20'),
(1011, 'Swati Snacks', 'Karan Singhania', 300.00, '2026-10-22');

-- Verify insertion
SELECT * FROM FoodOrders;
```

---

## 2. Write SQL statements to create a table called `TopSongs` with columns: `song_id`, `song_title`, `artist`, `streams`, and `release_date`, then insert at least 5 records representing popular tracks from Spotify.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Handling High-Volume Numeric Data (`BIGINT` for Billions of Streams)

---

### SQL Table Schema & Insert Script:
```sql
-- 1. Create TopSongs table
CREATE TABLE TopSongs (
    song_id INT PRIMARY KEY AUTO_INCREMENT,
    song_title VARCHAR(150) NOT NULL,
    artist VARCHAR(100) NOT NULL,
    streams BIGINT NOT NULL, -- Streams exceed 2-4 billion, requiring BIGINT instead of INT
    release_date DATE NOT NULL
);

-- 2. Insert 5 popular Spotify tracks
INSERT INTO TopSongs (song_title, artist, streams, release_date) VALUES
('Blinding Lights', 'The Weeknd', 4250000000, '2019-11-29'),
('Shape of You', 'Ed Sheeran', 3890000000, '2017-01-06'),
('Starboy', 'The Weeknd ft. Daft Punk', 3200000000, '2016-09-21'),
('Kesariya', 'Arijit Singh & Pritam', 680000000, '2022-07-17'),
('Chaleya', 'Anirudh Ravichander & Arijit Singh', 520000000, '2023-08-14');

-- 3. Verify Table
SELECT * FROM TopSongs ORDER BY streams DESC;
```

---

### Data Engineering Best Practice:
* **Why `BIGINT`?** A standard MySQL `INT` (signed 4-byte integer) can only store numbers up to $2,147,483,647$ ($\approx 2.14$ billion). Global streaming hits like *Blinding Lights* and *Shape of You* exceed 3 to 4 billion streams, which will trigger an **Integer Overflow Error** if defined as `INT`. Using `BIGINT` (8-byte integer supporting up to 9 quintillion) is mandatory.

#### Output:
| song_id | song_title | artist | streams | release_date |
|:---|:---|:---|:---|:---|
| 1 | Blinding Lights | The Weeknd | 4250000000 | 2019-11-29 |
| 2 | Shape of You | Ed Sheeran | 3890000000 | 2017-01-06 |
| 3 | Starboy | The Weeknd ft. Daft Punk | 3200000000 | 2016-09-21 |
| 4 | Kesariya | Arijit Singh & Pritam | 680000000 | 2022-07-17 |
| 5 | Chaleya | Anirudh Ravichander & Arijit Singh | 520000000 | 2023-08-14 |

---

## 3. Write an SQL query to find the top 3 customers who ordered the most from the `FoodOrders` table based on total `order_amount`, and display their names and total spent.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Aggregation with `GROUP BY`, Descending Sort (`ORDER BY`), and `LIMIT`

---

### SQL Query:
```sql
SELECT 
    customer_name,
    COUNT(order_id) AS total_orders,
    SUM(order_amount) AS total_spent
FROM FoodOrders
GROUP BY customer_name
ORDER BY total_spent DESC
LIMIT 3;
```

---

### Execution Breakdown:
1. `GROUP BY customer_name`: Aggregates the 11 transactions into unique customer accounts.
2. `SUM(order_amount)`: Calculates the cumulative amount spent by each user:
   * **Diya Patel:** $450 + 950 + 1400 = \mathbf{2,800.00}$ (3 orders)
   * **Aarav Sharma:** $850 + 600 + 1200 = \mathbf{2,650.00}$ (3 orders)
   * **Rohan Verma:** $1200 + 800 = \mathbf{2,000.00}$ (2 orders)
   * **Pooja Mehta:** $350 + 400 = 750.00$ (2 orders)
   * **Karan Singhania:** $300.00$ (1 order)
3. `ORDER BY total_spent DESC LIMIT 3`: Selects the top 3 high-value spenders.

#### Output:
| customer_name | total_orders | total_spent |
|:---|:---|:---|
| Diya Patel | 3 | 2800.00 |
| Aarav Sharma | 3 | 2650.00 |
| Rohan Verma | 2 | 2000.00 |

---

## 4. Generate a product performance report by writing an SQL query that lists each `restaurant_name` from `FoodOrders`, the number of orders, and the total `order_amount`, ordered by total `order_amount` descending.
*(Hint: Use GROUP BY and ORDER BY clauses.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Multi-Metric Restaurant Sales Performance Report

---

### SQL Query:
```sql
SELECT 
    restaurant_name,
    COUNT(order_id) AS total_orders,
    SUM(order_amount) AS total_revenue,
    ROUND(AVG(order_amount), 2) AS average_order_value
FROM FoodOrders
GROUP BY restaurant_name
ORDER BY total_revenue DESC;
```

---

### Business Intelligence Insights from this Report:
* **Top Grossing Merchant:** *Little Italy* generated the highest gross revenue (**₹2,250.00**) across just 2 orders, delivering the highest Average Order Value (**₹1,125.00**).
* **Highest Volume Delivery Partner:** *Swati Snacks* and *La Pino'z Pizza* handled the most orders (**3 orders each**), serving budget and quick-service diners.

#### Output:
| restaurant_name | total_orders | total_revenue | average_order_value |
|:---|:---|:---|:---|
| Little Italy | 2 | 2250.00 | 1125.00 |
| Behrouz Biryani | 2 | 2150.00 | 1075.00 |
| La Pino'z Pizza | 3 | 1800.00 | 600.00 |
| Theobroma | 1 | 1200.00 | 1200.00 |
| Swati Snacks | 3 | 1100.00 | 366.67 |

---

## 5. Create an SQL query that calculates two KPIs for the `FoodOrders` table: (1) average `order_amount` and (2) total number of unique customers, and format the output for dashboard display (two columns: `kpi_name`, `kpi_value`).

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Key Performance Indicator (KPI) Metric Card Formatting via `UNION ALL`

---

### SQL Queries:

#### **Method A: Vertical Executive Dashboard Card (Two Columns: `kpi_name`, `kpi_value`)**
> *This structure directly satisfies the requirement and formats metrics as key-value pairs ready for mobile card widgets or KPI reporting tiles:*

```sql
SELECT 
    'Average Order Amount' AS kpi_name,
    CONCAT('₹', ROUND(AVG(order_amount), 2)) AS kpi_value
FROM FoodOrders

UNION ALL

SELECT 
    'Total Unique Customers' AS kpi_name,
    CAST(COUNT(DISTINCT customer_name) AS CHAR) AS kpi_value
FROM FoodOrders;
```

#### **Method B: Raw Numeric KPI Card (Ideal for Chart Ingestion & BI Feeds)**
> *If the BI tool requires unformatted raw numeric values:*

```sql
SELECT 
    'Average Order Amount (INR)' AS kpi_name,
    ROUND(AVG(order_amount), 2) AS kpi_value
FROM FoodOrders

UNION ALL

SELECT 
    'Total Unique Customers' AS kpi_name,
    COUNT(DISTINCT customer_name) AS kpi_value
FROM FoodOrders;
```

---

### Technical Highlights:
1. **Vertical Schema Transformation:**
   * Executive summaries prefer stacked key-value metrics (`kpi_name`, `kpi_value`) rather than wide single-row tables.
2. **`UNION ALL` Operator:**
   * Combines the independent calculations of `AVG(order_amount)` and `COUNT(DISTINCT customer_name)`.
3. **Data Type Harmonization (`CAST` / `CONCAT`):**
   * `AVG()` produces a decimal, while `COUNT(DISTINCT)` produces an integer. By converting or prefixing them as string characters (`CONCAT('₹', ...)`, `CAST(... AS CHAR)`), both rows align under a single `kpi_value` column without type mismatch warnings.

#### Output (Method A):
| kpi_name | kpi_value |
|:---|:---|
| Average Order Amount | ₹772.73 |
| Total Unique Customers | 5 |
