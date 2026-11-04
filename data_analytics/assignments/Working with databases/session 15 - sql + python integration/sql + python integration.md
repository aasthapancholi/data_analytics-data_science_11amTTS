# Module 2: Working with Databases - Session 15: SQL & Python Integration (Pure SQL Solutions & Concept Mapping)

> **Student Note:** As Python has not yet been introduced in class, this document provides the **complete Pure SQL implementation** for every task (runnable directly in XAMPP, phpMyAdmin, MySQL, and SQLite). For each task, we also provide a **SQL $\leftrightarrow$ Python/Pandas Concept Bridge** and an optional reference Python snippet so you understand how these exact SQL commands connect to Python when you learn Module 3!

---

## 1. Install the `sqlite3` module in Python and write a script to create a new database called `foodie.db` with a table `Restaurants (id, name, cuisine, rating)`.

### SOLUTION:

#### **Pure SQL Implementation (XAMPP / MySQL / SQLite CLI):**
```sql
-- 1. Create the database (in MySQL / XAMPP)
CREATE DATABASE IF NOT EXISTS foodie;
USE foodie;

-- 2. Create the Restaurants table
CREATE TABLE Restaurants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    rating DECIMAL(3, 1) NOT NULL
);
```

> **For SQLite Engine (`foodie.db`):**
> In SQLite, files ending in `.db` represent the entire database. The SQL DDL is:
> ```sql
> CREATE TABLE IF NOT EXISTS Restaurants (
>     id INTEGER PRIMARY KEY AUTOINCREMENT,
>     name TEXT NOT NULL,
>     cuisine TEXT NOT NULL,
>     rating REAL NOT NULL
> );
> ```

---

### SQL $\leftrightarrow$ Python Concept Bridge:
* **Do we need to install `sqlite3`?**
  * **No!** `sqlite3` is a built-in module in Python's standard library. You do **not** need to run `pip install sqlite3`. It is ready to use immediately with `import sqlite3`.
* **How Python executes this SQL:**
  * In Python, `sqlite3.connect('foodie.db')` creates the database file automatically if it doesn't exist.
  * A `cursor` object is used to pass and execute the SQL `CREATE TABLE` statement.

#### Python Code (For Future Reference):
```python
import sqlite3

# Connect to database (creates 'foodie.db' file in the current directory)
conn = sqlite3.connect('foodie.db')
cursor = conn.cursor()

# Execute SQL DDL
cursor.execute('''
CREATE TABLE IF NOT EXISTS Restaurants (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    cuisine TEXT NOT NULL,
    rating REAL NOT NULL
);
''')

# Commit changes and close connection
conn.commit()
conn.close()
print("Database 'foodie.db' and table 'Restaurants' created successfully!")
```

---

## 2. Using `sqlite3` in Python, insert three sample restaurants into the `Restaurants` table in `foodie.db` and write a query to fetch all restaurants with a rating above 4.0, then print their names.

### SOLUTION:

#### **Pure SQL Implementation:**

```sql
-- 1. Insert 3 sample restaurants
INSERT INTO Restaurants (name, cuisine, rating) VALUES
('Little Italy', 'Italian', 4.5),
('Swati Snacks', 'Street Food', 4.3),
('Burger King', 'Fast Food', 3.8);

-- 2. Query to fetch names of restaurants with a rating above 4.0
SELECT 
    name,
    rating
FROM Restaurants
WHERE rating > 4.0;
```

---

### Step-by-Step Execution Breakdown:
1. `WHERE rating > 4.0` filters each row:
   * Little Italy: $4.5 > 4.0 \rightarrow$ **TRUE** (Selected)
   * Swati Snacks: $4.3 > 4.0 \rightarrow$ **TRUE** (Selected)
   * Burger King: $3.8 > 4.0 \rightarrow$ **FALSE** (Filtered out)
2. `SELECT name, rating` projects the qualified rows.

#### Output:
| name | rating |
|:---|:---|
| Little Italy | 4.5 |
| Swati Snacks | 4.3 |

---

### Python Code (For Future Reference):
```python
import sqlite3

conn = sqlite3.connect('foodie.db')
cursor = conn.cursor()

# Insert 3 sample rows
cursor.executemany('''
INSERT INTO Restaurants (name, cuisine, rating) VALUES (?, ?, ?);
''', [
    ('Little Italy', 'Italian', 4.5),
    ('Swati Snacks', 'Street Food', 4.3),
    ('Burger King', 'Fast Food', 3.8)
])
conn.commit()

# Fetch and print restaurants with rating > 4.0
cursor.execute("SELECT name FROM Restaurants WHERE rating > 4.0;")
results = cursor.fetchall()

print("Restaurants with rating > 4.0:")
for row in results:
    print("-", row[0])

conn.close()
```

---

## 3. Write Python code to load all rows from the `Restaurants` table in `foodie.db` into a Pandas DataFrame and display the top 2 rows using `DataFrame.head()`.

### SOLUTION:

#### **Pure SQL Equivalent:**

In pure SQL, "loading all rows" is a standard `SELECT *`, and "displaying the top 2 rows" (`head(2)`) is achieved using the **`LIMIT 2`** clause:

```sql
-- SQL equivalent of df.head(2):
SELECT 
    id,
    name,
    cuisine,
    rating
FROM Restaurants
LIMIT 2;
```

---

### SQL $\leftrightarrow$ Pandas Concept Mapping:

| SQL Concept | Pandas (Python) Equivalent | Description |
|:---|:---|:---|
| **Table** | **DataFrame (`df`)** | Two-dimensional tabular data structure with rows and columns. |
| **Row / Record** | **Observation / Series** | A single record in the dataset. |
| **Column / Field** | **Series / Feature** | A named vertical slice of data with a specific datatype. |
| **`LIMIT 2`** | **`df.head(2)`** | Displays the top 2 rows of the dataset. |
| **`LIMIT 2 OFFSET 2`** | **`df.iloc[2:4]`** | Displays a specific slice of rows. |
| **`SELECT * FROM table;`** | **`pd.read_sql_query("SELECT * ...", conn)`** | Loads the entire table into memory as a DataFrame. |

#### Output:
| id | name | cuisine | rating |
|:---|:---|:---|:---|
| 1 | Little Italy | Italian | 4.5 |
| 2 | Swati Snacks | Street Food | 4.3 |

---

### Python Code (For Future Reference):
```python
import sqlite3
import pandas as pd

# Connect to database
conn = sqlite3.connect('foodie.db')

# Load entire table into Pandas DataFrame
df = pd.read_sql_query("SELECT * FROM Restaurants;", conn)

# Display top 2 rows
print(df.head(2))

conn.close()
```

---

## 4. Add a new column `'delivery_charge'` to your DataFrame, setting it to 50 for all restaurants, and then calculate a new column `'final_rating'` as `rating + (0.1 if cuisine is 'Italian')`.
*(Hint: Use DataFrame.apply() or a lambda function for the conditional logic.)*

### SOLUTION:

#### **Pure SQL Implementation:**

There are two standard ways to achieve this in pure SQL:

#### **Method A: Dynamic Query Projection (Best Practice - Computes Columns on the Fly)**
> *In SQL data analytics, instead of permanently mutating the physical table, analysts project calculated columns using literals and `CASE WHEN`:*

```sql
SELECT 
    id,
    name,
    cuisine,
    rating,
    50.00 AS delivery_charge,
    ROUND(
        rating + (CASE WHEN cuisine = 'Italian' THEN 0.1 ELSE 0.0 END), 
        2
    ) AS final_rating
FROM Restaurants;
```

#### **Method B: Schema Alteration via `ALTER TABLE` and `UPDATE`**
> *If permanently adding the columns to the physical database schema:*

```sql
-- Step 1: Add delivery_charge column with default 50.00
ALTER TABLE Restaurants ADD COLUMN delivery_charge DECIMAL(5, 2) DEFAULT 50.00;
UPDATE Restaurants SET delivery_charge = 50.00;

-- Step 2: Add final_rating column
ALTER TABLE Restaurants ADD COLUMN final_rating DECIMAL(3, 2);

-- Step 3: Compute final_rating using conditional CASE WHEN logic
UPDATE Restaurants
SET final_rating = rating + (
    CASE 
        WHEN cuisine = 'Italian' THEN 0.1 
        ELSE 0.0 
    END
);

-- Step 4: Verify the table updates
SELECT * FROM Restaurants;
```

---

### SQL $\leftrightarrow$ Pandas Logic Comparison:

* **Adding a Constant Column:**
  * **Pandas:** `df['delivery_charge'] = 50`
  * **SQL:** `50.00 AS delivery_charge` (or `ALTER TABLE ... DEFAULT 50.00`)
* **Conditional Logic:**
  * **Pandas:** `df.apply(lambda row: row['rating'] + 0.1 if row['cuisine'] == 'Italian' else row['rating'], axis=1)`
  * **SQL:** `rating + (CASE WHEN cuisine = 'Italian' THEN 0.1 ELSE 0.0 END)`
  * *`CASE WHEN ... THEN ... ELSE ... END` in SQL is the direct equivalent of Python's `if ... else` and Pandas' `apply(lambda ...)`.*

#### Output:
| id | name | cuisine | rating | delivery_charge | final_rating |
|:---|:---|:---|:---|:---|:---|
| 1 | Little Italy | Italian | 4.5 | 50.00 | 4.60 |
| 2 | Swati Snacks | Street Food | 4.3 | 50.00 | 4.30 |
| 3 | Burger King | Fast Food | 3.8 | 50.00 | 3.80 |

---

### Python Code (For Future Reference):
```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('foodie.db')
df = pd.read_sql_query("SELECT * FROM Restaurants;", conn)

# 1. Add delivery_charge column set to 50
df['delivery_charge'] = 50

# 2. Add final_rating using apply with lambda conditional logic
df['final_rating'] = df.apply(
    lambda row: round(row['rating'] + 0.1, 2) if row['cuisine'] == 'Italian' else row['rating'], 
    axis=1
)

print(df)
conn.close()
```

---

## 5. Automate a daily summary: Write a Python script that connects to `foodie.db`, fetches all restaurants with rating above 4.5, loads them into a DataFrame, and saves the result as a CSV file named `top_rated_restaurants.csv`.

### SOLUTION:

#### **Pure SQL Implementation:**

#### **1. The SQL Query to Extract the Target Data:**
```sql
SELECT 
    id,
    name,
    cuisine,
    rating
FROM Restaurants
WHERE rating >= 4.5
ORDER BY rating DESC;
```

#### **2. Automating with a SQL View (`CREATE VIEW`):**
> *In pure SQL, an automated daily summary is created as a **View**. Whenever a user, dashboard, or Power BI connects to the database, the View automatically delivers the updated list without needing any Python script:*

```sql
CREATE OR REPLACE VIEW Daily_Top_Rated_Summary AS
SELECT 
    id,
    name,
    cuisine,
    rating
FROM Restaurants
WHERE rating >= 4.5;

-- Test the daily automated summary view
SELECT * FROM Daily_Top_Rated_Summary;
```

---

#### **3. How to Export to CSV using Database Tools (Without Python):**

* **Method 1: Using phpMyAdmin / XAMPP (GUI One-Click Export):**
  1. Open **phpMyAdmin** (`http://localhost/phpmyadmin`).
  2. Select database `foodie` and run: `SELECT * FROM Restaurants WHERE rating >= 4.5;`.
  3. Scroll to the bottom of the results and click **Export**.
  4. Choose **CSV** as the format and click **Go** $\rightarrow$ File downloads as `top_rated_restaurants.csv`!
* **Method 2: Using Pure MySQL SQL (`INTO OUTFILE`):**
  ```sql
  SELECT id, name, cuisine, rating
  INTO OUTFILE 'C:/xampp/mysql/data/top_rated_restaurants.csv'
  FIELDS TERMINATED BY ','
  ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  FROM Restaurants
  WHERE rating >= 4.5;
  ```
* **Method 3: Using SQLite Command Line:**
  ```sql
  .headers on
  .mode csv
  .output top_rated_restaurants.csv
  SELECT * FROM Restaurants WHERE rating >= 4.5;
  .output stdout
  ```

---

### Python Automation Code (For Future Reference):
```python
import sqlite3
import pandas as pd

# 1. Connect to database
conn = sqlite3.connect('foodie.db')

# 2. Query top-rated restaurants directly into a DataFrame
query = "SELECT * FROM Restaurants WHERE rating >= 4.5;"
df_top_rated = pd.read_sql_query(query, conn)

# 3. Export DataFrame directly to CSV file
df_top_rated.to_csv('top_rated_restaurants.csv', index=False)

print("Automated daily summary saved successfully to 'top_rated_restaurants.csv'!")
print(df_top_rated)

conn.close()
```

#### Output (`top_rated_restaurants.csv`):
```csv
id,name,cuisine,rating
1,Little Italy,Italian,4.5
```
