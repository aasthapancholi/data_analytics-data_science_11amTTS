# Module 2: Working with Databases - Session 4: SELECT Statement Basics

---

## 1. Create a table named `MusicPlaylist` with columns: `id`, `song_name`, `artist`, `genre`, and `duration`. Insert at least 5 records representing songs from your favorite Spotify playlist, then write a SELECT statement to retrieve all columns for all songs.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique identifier for each song)
  * `song_name`: `VARCHAR(100) NOT NULL` (Track title)
  * `artist`: `VARCHAR(100) NOT NULL` (Performing artist/band)
  * `genre`: `VARCHAR(50) NOT NULL` (Music category)
  * `duration`: `INT NOT NULL` (Song length in seconds)

#### SQL Script:
```sql
-- 1. Create the MusicPlaylist table
CREATE TABLE MusicPlaylist (
    id INT PRIMARY KEY AUTO_INCREMENT,
    song_name VARCHAR(100) NOT NULL,
    artist VARCHAR(100) NOT NULL,
    genre VARCHAR(50) NOT NULL,
    duration INT NOT NULL -- duration stored in seconds
);

-- 2. Insert 5 records from Spotify playlist
INSERT INTO MusicPlaylist (song_name, artist, genre, duration) VALUES
('Blinding Lights', 'The Weeknd', 'Synthwave', 200),
('Shape of You', 'Ed Sheeran', 'Pop', 233),
('Kesariya', 'Arijit Singh', 'Bollywood', 268),
('Levitating', 'Dua Lipa', 'Pop', 203),
('Starboy', 'The Weeknd', 'R&B', 230);

-- 3. Retrieve all columns for all songs
SELECT * FROM MusicPlaylist;
```

#### Output (Table State):
| id | song_name | artist | genre | duration |
|:---|:---|:---|:---|:---|
| 1 | Blinding Lights | The Weeknd | Synthwave | 200 |
| 2 | Shape of You | Ed Sheeran | Pop | 233 |
| 3 | Kesariya | Arijit Singh | Bollywood | 268 |
| 4 | Levitating | Dua Lipa | Pop | 203 |
| 5 | Starboy | The Weeknd | R&B | 230 |

*(Screenshot Recommended: phpMyAdmin / MySQL Workbench query window showing the 5 inserted rows)*

---

## 2. Write a SQL query to display only the `song_name` and `artist` columns from the `MusicPlaylist` table, showing just the first 3 records using the `LIMIT` keyword.

### SOLUTION:
To retrieve only specific columns, we specify `song_name` and `artist` in the `SELECT` clause, and append `LIMIT 3` to restrict the result set to the first 3 rows.

#### SQL Query:
```sql
SELECT song_name, artist 
FROM MusicPlaylist 
LIMIT 3;
```

#### Output:
| song_name | artist |
|:---|:---|
| Blinding Lights | The Weeknd |
| Shape of You | Ed Sheeran |
| Kesariya | Arijit Singh |


---

## 3. Suppose you have a table named `FoodOrders` with columns: `id`, `restaurant`, `food_item`, and `order_date`. Write a SQL query to list all unique restaurant names where you have placed orders, using the `DISTINCT` keyword.

### SOLUTION:
The `DISTINCT` keyword eliminates duplicate values from the output, ensuring that each restaurant name appears only once even if multiple orders were placed there.

#### SQL Query:
```sql
SELECT DISTINCT restaurant 
FROM FoodOrders;
```

#### Optional Dummy Setup (to test in MySQL):
```sql
-- Creating dummy FoodOrders table for testing
CREATE TABLE FoodOrders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    restaurant VARCHAR(100) NOT NULL,
    food_item VARCHAR(100) NOT NULL,
    order_date DATE NOT NULL
);

INSERT INTO FoodOrders (restaurant, food_item, order_date) VALUES
('Dominos', 'Farmhouse Pizza', '2026-10-01'),
('Haldirams', 'Chole Bhature', '2026-10-02'),
('Dominos', 'Garlic Bread', '2026-10-05'),
('Subway', 'Paneer Tikka Sub', '2026-10-07'),
('Haldirams', 'Raj Kachori', '2026-10-10');
```

#### Output:
| restaurant |
|:---|
| Dominos |
| Haldirams |
| Subway |

---

## 4. Write a SQL query on the `FoodOrders` table to select `food_item` as 'Dish' and `order_date` as 'Date Ordered', displaying only these two columns with the column aliases in the output.

### SOLUTION:
Column aliases are created using the `AS` keyword. When alias names contain spaces (e.g., `'Date Ordered'`), they are enclosed in single or double quotes.

#### SQL Query:
```sql
SELECT 
    food_item AS 'Dish', 
    order_date AS 'Date Ordered'
FROM FoodOrders;
```

#### Output:
| Dish | Date Ordered |
|:---|:---|
| Farmhouse Pizza | 2026-10-01 |
| Chole Bhature | 2026-10-02 |
| Garlic Bread | 2026-10-05 |
| Paneer Tikka Sub | 2026-10-07 |
| Raj Kachori | 2026-10-10 |

---

## 5. You tried running this query: `SELECT DISTINCT food_item, restaurant FROM FoodOrders LIMIT 2`, but it returns an error or doesn't work as expected. Identify and fix the mistake in the query.

### SOLUTION & ANALYSIS:

Depending on the database environment and expectations, there are key reasons why this query fails or does not work as expected:

### 1. Database Dialect Incompatibility (The Primary Syntax Error)
* **SQL Server (T-SQL) & MS Access:**
  * The `LIMIT` keyword is **not supported** in SQL Server. Running this query in SQL Server or SQL Server Management Studio (SSMS) produces:
    `Msg 102, Level 15: Incorrect syntax near 'LIMIT'`.
  * **Fix for SQL Server:** Use `TOP` immediately after `SELECT DISTINCT`:
    ```sql
    SELECT DISTINCT TOP 2 food_item, restaurant 
    FROM FoodOrders;
    ```
* **Oracle Database:**
  * Oracle does not use `LIMIT`. In modern Oracle (12c+), use `FETCH FIRST`:
    ```sql
    SELECT DISTINCT food_item, restaurant 
    FROM FoodOrders 
    FETCH FIRST 2 ROWS ONLY;
    ```

---

### 2. Missing `ORDER BY` Clause (Why it "doesn't work as expected")
* In relational databases (including MySQL and PostgreSQL), records have **no inherent order**.
* When using `LIMIT 2` without an `ORDER BY` clause, the database returns an **arbitrary (unpredictable)** set of 2 rows. Every time data changes or an index is updated, different rows might be returned.
* **Fix (Adding deterministic ordering):** To get the first 2 ordered records reliably:
  ```sql
  SELECT DISTINCT food_item, restaurant 
  FROM FoodOrders 
  ORDER BY id ASC 
  LIMIT 2;
  ```

---

### 3. Misunderstanding of `DISTINCT` on Multiple Columns
* `DISTINCT` applies to the **combination** of `(food_item, restaurant)`, not just one column.
* If your intention was to get 2 distinct restaurants, this query might still return the same restaurant twice if the dishes differ (e.g. `['Pizza', 'Dominos']` and `['Burger', 'Dominos']`).
* **Fix (If distinct restaurants were desired):**
  ```sql
  SELECT DISTINCT restaurant 
  FROM FoodOrders 
  LIMIT 2;
  ```

---

### 4. Missing Statement Terminator
* SQL statements should end with a semicolon (`;`). Omitting it can cause syntax errors when running multiple statements together.
* **Standard MySQL / PostgreSQL Corrected Query:**
  ```sql
  SELECT DISTINCT food_item, restaurant 
  FROM FoodOrders 
  ORDER BY id ASC 
  LIMIT 2;
  ```
