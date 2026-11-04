# Module 2: Working with Databases - Session 17: SQL Capstone Project (Zomato Restaurant & Reviews Analytics)

---

## 1. Create a SQL table called `Restaurant` with columns: `id`, `name`, `cuisine`, `location`, and `average_rating`. Insert at least 5 sample rows representing popular restaurants from Zomato.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Relational Schema DDL & Seed Data Insertion
* **Table Design:**
  * `Restaurant`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique restaurant identifier)
    * `name`: `VARCHAR(100) NOT NULL` (Restaurant name)
    * `cuisine`: `VARCHAR(50) NOT NULL` (Primary food category, e.g., 'Street Food', 'Italian', 'Gujarati')
    * `location`: `VARCHAR(100) NOT NULL` (City / area landmark)
    * `average_rating`: `DECIMAL(3, 2) NOT NULL` (Overall platform listed rating)

#### SQL Setup Script:
```sql
-- 1. Create Database
CREATE DATABASE IF NOT EXISTS ZomatoAnalyticsDB;
USE ZomatoAnalyticsDB;

-- 2. Create Restaurant table
CREATE TABLE Restaurant (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    location VARCHAR(100) NOT NULL,
    average_rating DECIMAL(3, 2) NOT NULL
);

-- 3. Insert 7 sample restaurants across multiple cuisines (to enable multi-item grouping & ranking)
INSERT INTO Restaurant (name, cuisine, location, average_rating) VALUES
('Swati Snacks', 'Street Food', 'Law Garden, Ahmedabad', 4.50),
('Manek Chowk Treats', 'Street Food', 'Manek Chowk, Ahmedabad', 4.10),
('Little Italy', 'Italian', 'Bodakdev, Ahmedabad', 4.60),
('Sale & Pepe', 'Italian', 'Prahlad Nagar, Ahmedabad', 4.20),
('Gordhan Thal', 'Gujarati', 'SG Highway, Ahmedabad', 4.40),
('Sasumaa Gujarati Thali', 'Gujarati', 'Ring Road, Surat', 4.70),
('Behrouz Biryani', 'North Indian', 'Vastrapur, Ahmedabad', 4.30);

-- Verify records
SELECT * FROM Restaurant;
```

#### Output (Initial Table State):
| id | name | cuisine | location | average_rating |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Street Food | Law Garden, Ahmedabad | 4.50 |
| 2 | Manek Chowk Treats | Street Food | Manek Chowk, Ahmedabad | 4.10 |
| 3 | Little Italy | Italian | Bodakdev, Ahmedabad | 4.60 |
| 4 | Sale & Pepe | Italian | Prahlad Nagar, Ahmedabad | 4.20 |
| 5 | Gordhan Thal | Gujarati | SG Highway, Ahmedabad | 4.40 |
| 6 | Sasumaa Gujarati Thali | Gujarati | Ring Road, Surat | 4.70 |
| 7 | Behrouz Biryani | North Indian | Vastrapur, Ahmedabad | 4.30 |

---

## 2. Write a SQL query to generate a report showing the number of restaurants for each cuisine type from your `Restaurant` table, ordered by the count in descending order.
*(Hint: Use GROUP BY and ORDER BY.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Category Frequency Aggregation using `GROUP BY` and `COUNT()`

---

### SQL Query:
```sql
SELECT 
    cuisine,
    COUNT(id) AS total_restaurants
FROM Restaurant
GROUP BY cuisine
ORDER BY total_restaurants DESC, cuisine ASC;
```

---

### Step-by-Step Execution Breakdown:
1. `FROM Restaurant`: Scans all 7 restaurant rows.
2. `GROUP BY cuisine`: Forms distinct categorical buckets: `Street Food`, `Italian`, `Gujarati`, and `North Indian`.
3. `COUNT(id)`: Counts the total restaurants mapped to each cuisine bucket.
4. `ORDER BY total_restaurants DESC`: Sorts the cuisines with the highest restaurant density at the top.

#### Output:
| cuisine | total_restaurants |
|:---|:---|
| Gujarati | 2 |
| Italian | 2 |
| Street Food | 2 |
| North Indian | 1 |

---

## 3. Add a new table called `Review` with columns: `id`, `restaurant_id`, `user_name`, `rating`, and `review_date`. Insert at least 10 sample reviews, linking them to restaurants using `restaurant_id`.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Normalized $1:N$ Foreign Key Relationship Design & Sample Review Transactions
* **Table Design:**
  * `Review`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique review ID)
    * `restaurant_id`: `INT NOT NULL` (Foreign key referencing `Restaurant.id`)
    * `user_name`: `VARCHAR(100) NOT NULL` (Customer name)
    * `rating`: `DECIMAL(2, 1) NOT NULL` (Star rating given by customer, e.g. 5.0, 4.0, 3.5)
    * `review_date`: `DATE NOT NULL` (Review publication date)
    * Constraint: `FOREIGN KEY (restaurant_id) REFERENCES Restaurant(id) ON DELETE CASCADE`

#### SQL Setup Script:
```sql
-- 1. Create Review table with Foreign Key constraint
CREATE TABLE Review (
    id INT PRIMARY KEY AUTO_INCREMENT,
    restaurant_id INT NOT NULL,
    user_name VARCHAR(100) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL,
    review_date DATE NOT NULL,
    FOREIGN KEY (restaurant_id) REFERENCES Restaurant(id) ON DELETE CASCADE
);

-- 2. Insert 12 sample customer reviews linked to restaurants 1 through 7
INSERT INTO Review (restaurant_id, user_name, rating, review_date) VALUES
-- Swati Snacks (restaurant_id: 1)
(1, 'Aarav Sharma', 5.0, '2026-10-01'),
(1, 'Diya Patel', 4.5, '2026-10-03'),

-- Manek Chowk Treats (restaurant_id: 2)
(2, 'Rohan Verma', 4.0, '2026-10-02'),
(2, 'Pooja Mehta', 3.8, '2026-10-06'),

-- Little Italy (restaurant_id: 3)
(3, 'Aarav Sharma', 4.8, '2026-10-04'),
(3, 'Karan Singhania', 4.6, '2026-10-08'),

-- Sale & Pepe (restaurant_id: 4)
(4, 'Sneha Shah', 4.0, '2026-10-05'),
(4, 'Vikas Gupta', 4.2, '2026-10-10'),

-- Gordhan Thal (restaurant_id: 5)
(5, 'Diya Patel', 4.3, '2026-10-07'),
(5, 'Ankit Dave', 4.5, '2026-10-12'),

-- Sasumaa Gujarati Thali (restaurant_id: 6)
(6, 'Rohan Verma', 4.9, '2026-10-09'),
(6, 'Pooja Mehta', 4.7, '2026-10-14'),

-- Behrouz Biryani (restaurant_id: 7)
(7, 'Karan Singhania', 4.4, '2026-10-11'),
(7, 'Sneha Shah', 4.2, '2026-10-15');

-- Verify insertion
SELECT * FROM Review;
```

---

## 4. Write a SQL query using a `JOIN` to display each restaurant's name, cuisine, and its average review rating (from the `Review` table), ordered by highest average rating first.
*(Hint: Use JOIN and GROUP BY with aggregate functions.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Relational `INNER JOIN` with Aggregation (`AVG`) & Formatting (`ROUND`)

---

### SQL Query:
```sql
SELECT 
    r.id AS restaurant_id,
    r.name AS restaurant_name,
    r.cuisine,
    COUNT(rev.id) AS total_reviews,
    ROUND(AVG(rev.rating), 2) AS calculated_avg_rating
FROM Restaurant r
INNER JOIN Review rev 
    ON r.id = rev.restaurant_id
GROUP BY r.id, r.name, r.cuisine
ORDER BY calculated_avg_rating DESC;
```

---

### Step-by-Step Calculation Trace:
* **Sasumaa Gujarati Thali:** $(4.9 + 4.7) / 2 = \mathbf{4.80}$
* **Swati Snacks:** $(5.0 + 4.5) / 2 = \mathbf{4.75}$
* **Little Italy:** $(4.8 + 4.6) / 2 = \mathbf{4.70}$
* **Gordhan Thal:** $(4.3 + 4.5) / 2 = \mathbf{4.40}$
* **Behrouz Biryani:** $(4.4 + 4.2) / 2 = \mathbf{4.30}$
* **Sale & Pepe:** $(4.0 + 4.2) / 2 = \mathbf{4.10}$
* **Manek Chowk Treats:** $(4.0 + 3.8) / 2 = \mathbf{3.90}$

#### Output:
| restaurant_id | restaurant_name | cuisine | total_reviews | calculated_avg_rating |
|:---|:---|:---|:---|:---|
| 6 | Sasumaa Gujarati Thali | Gujarati | 2 | 4.80 |
| 1 | Swati Snacks | Street Food | 2 | 4.75 |
| 3 | Little Italy | Italian | 2 | 4.70 |
| 5 | Gordhan Thal | Gujarati | 2 | 4.40 |
| 7 | Behrouz Biryani | North Indian | 2 | 4.30 |
| 4 | Sale & Pepe | Italian | 2 | 4.10 |
| 2 | Manek Chowk Treats | Street Food | 2 | 3.90 |

---

## 5. Use a window function to rank restaurants by their average review rating within each cuisine type, showing the restaurant name, cuisine, average rating, and rank.
*(Hint: Use the RANK() or DENSE_RANK() window function partitioned by cuisine.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Window Partitioning over Aggregated Metrics (`DENSE_RANK() OVER (PARTITION BY cuisine ...)`)

---

### SQL Queries:

#### **Method A: Clean Two-Stage CTE Pipeline (Recommended for Clarity & Industry Dashboards)**
> *Stage 1 aggregates the reviews per restaurant; Stage 2 applies the window function partitioned by cuisine:*

```sql
WITH RestaurantReviewStats AS (
    -- Step 1: Calculate the average review rating per restaurant
    SELECT 
        r.id AS restaurant_id,
        r.name AS restaurant_name,
        r.cuisine,
        ROUND(AVG(rev.rating), 2) AS avg_rating
    FROM Restaurant r
    INNER JOIN Review rev 
        ON r.id = rev.restaurant_id
    GROUP BY r.id, r.name, r.cuisine
)
-- Step 2: Rank restaurants within each individual cuisine category
SELECT 
    cuisine,
    DENSE_RANK() OVER (
        PARTITION BY cuisine 
        ORDER BY avg_rating DESC
    ) AS cuisine_rank,
    restaurant_name,
    avg_rating
FROM RestaurantReviewStats
ORDER BY cuisine ASC, cuisine_rank ASC;
```

---

#### **Method B: Single-Query Aggregation with Inline Window Function**
> *In MySQL 8.0+, a window function can be computed directly over an aggregate function `AVG()` in the same query block:*

```sql
SELECT 
    r.cuisine,
    DENSE_RANK() OVER (
        PARTITION BY r.cuisine 
        ORDER BY ROUND(AVG(rev.rating), 2) DESC
    ) AS cuisine_rank,
    r.name AS restaurant_name,
    ROUND(AVG(rev.rating), 2) AS avg_rating
FROM Restaurant r
INNER JOIN Review rev 
    ON r.id = rev.restaurant_id
GROUP BY r.id, r.name, r.cuisine
ORDER BY r.cuisine ASC, cuisine_rank ASC;
```

---

### Key Analytical Takeaways:
1. **`PARTITION BY cuisine`:**
   * Isolates the ranking to each distinct cuisine category. The rank resets to **1** for the top restaurant in every food category.
2. **Category Champions Identified:**
   * **Gujarati Top Pick:** *Sasumaa Gujarati Thali* (4.80) ranks #1; *Gordhan Thal* (4.40) ranks #2.
   * **Italian Top Pick:** *Little Italy* (4.70) ranks #1; *Sale & Pepe* (4.10) ranks #2.
   * **Street Food Top Pick:** *Swati Snacks* (4.75) ranks #1; *Manek Chowk Treats* (3.90) ranks #2.
   * **North Indian Top Pick:** *Behrouz Biryani* (4.30) ranks #1.

#### Output:
| cuisine | cuisine_rank | restaurant_name | avg_rating |
|:---|:---|:---|:---|
| Gujarati | 1 | Sasumaa Gujarati Thali | 4.80 |
| Gujarati | 2 | Gordhan Thal | 4.40 |
| Italian | 1 | Little Italy | 4.70 |
| Italian | 2 | Sale & Pepe | 4.10 |
| North Indian | 1 | Behrouz Biryani | 4.30 |
| Street Food | 1 | Swati Snacks | 4.75 |
| Street Food | 2 | Manek Chowk Treats | 3.90 |
