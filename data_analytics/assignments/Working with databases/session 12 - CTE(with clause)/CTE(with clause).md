# Module 2: Working with Databases - Session 12: CTEs (Common Table Expressions & WITH Clause)

---

## 1. Create a CTE using the `WITH` clause to select all products with a rating above 4.5 from a `Products` table, similar to how Flipkart or Myntra might highlight top-rated items.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Common Table Expression (CTE) using `WITH` clause
* **Table Design:**
  * `Products`:
    * `product_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique product identifier)
    * `product_name`: `VARCHAR(150) NOT NULL` (Item title)
    * `brand`: `VARCHAR(100) NOT NULL` (Brand name, e.g., 'Nike', 'Boat', 'Puma')
    * `category`: `VARCHAR(50) NOT NULL` (e.g., 'Footwear', 'Audio', 'Apparel')
    * `price`: `DECIMAL(10, 2) NOT NULL` (Retail price in INR)
    * `rating`: `DECIMAL(2, 1) NOT NULL` (Average customer star rating)
    * `total_reviews`: `INT NOT NULL` (Total customer review count)

#### SQL Setup Script:
```sql
-- 1. Create Products table
CREATE TABLE Products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(150) NOT NULL,
    brand VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL,
    total_reviews INT NOT NULL
);

-- 2. Insert sample catalog items (Flipkart / Myntra style)
INSERT INTO Products (product_name, brand, category, price, rating, total_reviews) VALUES
('Air Jordan 1 Retro', 'Nike', 'Footwear', 12999.00, 4.8, 1250),
('Airdopes 141 Wireless Earbuds', 'boAt', 'Audio', 1299.00, 4.2, 45000),
('Noise ColorFit Pro Smartwatch', 'Noise', 'Wearables', 1999.00, 4.6, 18200),
('Running Sport Shoes', 'Puma', 'Footwear', 2499.00, 4.1, 3100),
('Sony WH-1000XM5 Noise Cancelling', 'Sony', 'Audio', 29990.00, 4.9, 4200),
('Classic Cotton Casual Shirt', 'Roadster', 'Apparel', 799.00, 3.9, 850),
('Slim Fit Denim Jeans', 'Levis', 'Apparel', 2799.00, 4.7, 6100);

-- Verify table data
SELECT * FROM Products;
```

---

### SQL Query:

#### **Top-Rated Products CTE:**
```sql
WITH TopRatedProducts AS (
    -- Define CTE to isolate products with rating > 4.5
    SELECT 
        product_id,
        product_name,
        brand,
        category,
        price,
        rating,
        total_reviews
    FROM Products
    WHERE rating > 4.5
)
-- Main query consuming the CTE
SELECT 
    product_id,
    product_name,
    brand,
    category,
    price,
    rating,
    total_reviews,
    'Top Rated Pick' AS badge
FROM TopRatedProducts
ORDER BY rating DESC, total_reviews DESC;
```

---

### Key Concepts Explained:
1. **What is a CTE?**
   * A **Common Table Expression (CTE)** is a temporary named result set defined using the `WITH` keyword. It exists only within the execution scope of that single statement (`SELECT`, `INSERT`, `UPDATE`, or `DELETE`).
2. **Why use CTEs over Subqueries?**
   * **Modularity:** E-commerce catalogs frequently filter items into segments (e.g. "Top Rated", "Deals of the Day"). A CTE defines this business segment at the very top of the query, making the final presentation layer clean and adaptable.

#### Output:
| product_id | product_name | brand | category | price | rating | total_reviews | badge |
|:---|:---|:---|:---|:---|:---|:---|:---|
| 5 | Sony WH-1000XM5 Noise Cancelling | Sony | Audio | 29990.00 | 4.9 | 4200 | Top Rated Pick |
| 1 | Air Jordan 1 Retro | Nike | Footwear | 12999.00 | 4.8 | 1250 | Top Rated Pick |
| 7 | Slim Fit Denim Jeans | Levis | Apparel | 2799.00 | 4.7 | 6100 | Top Rated Pick |
| 3 | Noise ColorFit Pro Smartwatch | Noise | Wearables | 1999.00 | 4.6 | 18200 | Top Rated Pick |

---

## 2. Rewrite a query that finds all restaurants in 'Ahmedabad' with delivery charges under 50 from a `Restaurants` table, first using a subquery and then using a CTE. Compare both queries for readability.
*(Hint: Focus on making the CTE version cleaner and easier to understand.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Subquery vs. Common Table Expression (CTE) Readability & Structure
* **Table Design:**
  * `Restaurants`:
    * `restaurant_id`: `INT PRIMARY KEY AUTO_INCREMENT`
    * `restaurant_name`: `VARCHAR(100) NOT NULL`
    * `cuisine`: `VARCHAR(50) NOT NULL`
    * `city`: `VARCHAR(50) NOT NULL`
    * `delivery_charge`: `DECIMAL(5, 2) NOT NULL`
    * `rating`: `DECIMAL(2, 1) NOT NULL`

#### SQL Setup Script:
```sql
-- 1. Create Restaurants table
CREATE TABLE Restaurants (
    restaurant_id INT PRIMARY KEY AUTO_INCREMENT,
    restaurant_name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    city VARCHAR(50) NOT NULL,
    delivery_charge DECIMAL(5, 2) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL
);

-- 2. Insert sample restaurants
INSERT INTO Restaurants (restaurant_name, cuisine, city, delivery_charge, rating) VALUES
('Gwalia Sweets', 'Mithai / Fast Food', 'Ahmedabad', 30.00, 4.4),
('Gordhan Thal', 'Gujarati Thali', 'Ahmedabad', 70.00, 4.5),
('Swati Snacks', 'Street Food / Snacks', 'Ahmedabad', 40.00, 4.6),
('Theobroma', 'Bakery / Desserts', 'Ahmedabad', 25.00, 4.7),
('Bikanervala', 'North Indian', 'Ahmedabad', 60.00, 4.1),
('Bademiya', 'Mughlai', 'Mumbai', 35.00, 4.2),
('Bastian', 'Seafood', 'Mumbai', 90.00, 4.8);
```

---

### SQL Queries:

#### **Version 1: Using a Subquery (Derived Table Approach)**
```sql
SELECT 
    sub.restaurant_name,
    sub.cuisine,
    sub.city,
    sub.delivery_charge,
    sub.rating
FROM (
    -- Subquery: Extracting restaurants in Ahmedabad
    SELECT * 
    FROM Restaurants 
    WHERE city = 'Ahmedabad'
) AS sub
WHERE sub.delivery_charge < 50.00
ORDER BY sub.delivery_charge ASC;
```

#### **Version 2: Using a CTE (`WITH` Clause Approach - Cleaner & Modular)**
```sql
WITH AhmedabadRestaurants AS (
    -- Step 1: Filter all restaurants located in Ahmedabad
    SELECT 
        restaurant_id,
        restaurant_name,
        cuisine,
        city,
        delivery_charge,
        rating
    FROM Restaurants
    WHERE city = 'Ahmedabad'
)
-- Step 2: Main query filtering for budget delivery
SELECT 
    restaurant_name,
    cuisine,
    city,
    delivery_charge,
    rating
FROM AhmedabadRestaurants
WHERE delivery_charge < 50.00
ORDER BY delivery_charge ASC;
```

---

### Readability & Architectural Comparison:

| Criteria | Subquery Approach | CTE Approach (`WITH` Clause) |
|:---|:---|:---|
| **Reading Direction** | **Inside-Out (Reverse):** Requires jumping down into the `FROM (...)` clause first before understanding the main query. | **Top-to-Bottom (Natural Linear):** Reads like sequential logic in Python or standard storytelling. |
| **Naming & Intent** | Employs generic alias names like `sub` or `t1` at the end of the query block. | Clearly named descriptive identifier (`AhmedabadRestaurants`) at the very top. |
| **Separation of Concerns** | Mixed together; inner filtering and outer projection are wrapped into a single block. | Step 1 (Locality Filter) is physically separated from Step 2 (Price Condition). |
| **Ease of Debugging** | Hard to test sub-parts individually without selecting and isolating inner code. | Easy to debug: simply run `SELECT * FROM AhmedabadRestaurants` to verify intermediate results. |
| **Reusability** | Cannot be reused in the same query without copying and pasting the whole subquery. | Can be referenced multiple times (e.g. self-joined or combined with another CTE). |

#### Output:
| restaurant_name | cuisine | city | delivery_charge | rating |
|:---|:---|:---|:---|:---|
| Theobroma | Bakery / Desserts | Ahmedabad | 25.00 | 4.7 |
| Gwalia Sweets | Mithai / Fast Food | Ahmedabad | 30.00 | 4.4 |
| Swati Snacks | Street Food / Snacks | Ahmedabad | 40.00 | 4.6 |

---

## 3. Using two CTEs in a single query, find the top 3 most-followed users and the top 3 most-liked posts from a `Users` and `Posts` table (think Instagram-style data). Output both lists in the same result set.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Multiple Chained CTEs + `UNION ALL` for unified reporting
* **Table Design:**
  * `Users`:
    * `user_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique creator ID)
    * `username`: `VARCHAR(50) NOT NULL UNIQUE` (Instagram handle)
    * `followers_count`: `INT NOT NULL` (Total follower tally)
  * `Posts`:
    * `post_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique post ID)
    * `user_id`: `INT NOT NULL` (Post creator foreign key)
    * `caption`: `VARCHAR(255) NOT NULL` (Post caption/title)
    * `likes_count`: `INT NOT NULL` (Total like tally)

#### SQL Setup Script:
```sql
-- 1. Create Users table
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    followers_count INT NOT NULL
);

-- 2. Create Posts table
CREATE TABLE Posts (
    post_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    caption VARCHAR(255) NOT NULL,
    likes_count INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- 3. Insert sample Users (Celebrity / Creator handles)
INSERT INTO Users (username, followers_count) VALUES
('virat.kohli', 270000000),
('shraddhakapoor', 93000000),
('aliaabhatt', 85000000),
('ranveersingh', 47000000),
('bhuvan.bam22', 19000000);

-- 4. Insert sample Posts
INSERT INTO Posts (user_id, caption, likes_count) VALUES
(1, 'Historic victory with team India at Wankhede! 🇮🇳🏆', 8500000),
(1, 'Morning gym routine done right 💪', 4200000),
(2, 'Sunday smiles & rainy day coffee ☕🌸', 5100000),
(3, 'Moments from the set of our next film ✨🎬', 6400000),
(4, 'Unapologetic energy on the red carpet 🔥', 3100000),
(5, 'New Dhindora episode dropping tomorrow! 🍿', 2800000);
```

---

### SQL Query:

#### **Dual CTEs Combined via `UNION ALL`:**
```sql
-- CTE 1: Top 3 Most Followed Users
WITH TopFollowedUsers AS (
    SELECT 
        'Top Followed User' AS entity_category,
        user_id AS reference_id,
        CONCAT('@', username) AS entity_name,
        followers_count AS metric_score
    FROM Users
    ORDER BY followers_count DESC
    LIMIT 3
),
-- CTE 2: Top 3 Most Liked Posts
TopLikedPosts AS (
    SELECT 
        'Top Liked Post' AS entity_category,
        post_id AS reference_id,
        caption AS entity_name,
        likes_count AS metric_score
    FROM Posts
    ORDER BY likes_count DESC
    LIMIT 3
)
-- Unified Final Result Set
SELECT 
    entity_category,
    reference_id,
    entity_name,
    metric_score
FROM TopFollowedUsers

UNION ALL

SELECT 
    entity_category,
    reference_id,
    entity_name,
    metric_score
FROM TopLikedPosts;
```

---

### Key Technical Insights:
1. **Chaining Multiple CTEs:**
   * Notice that the `WITH` keyword is declared **only once** at the beginning. Subsequent CTEs are separated by commas (`,`).
2. **Schema Standardization for `UNION ALL`:**
   * `Users` and `Posts` have completely different schemas. By standardizing the column aliases in both CTEs (`entity_category`, `reference_id`, `entity_name`, `metric_score`), we can seamlessly stitch both datasets into a single dashboard payload.
3. **Performance Benefit:**
   * Each CTE performs its own sorted `LIMIT 3` query using indexes, avoiding expensive Cartesian products or unnecessary full table joins.

#### Output:
| entity_category | reference_id | entity_name | metric_score |
|:---|:---|:---|:---|
| Top Followed User | 1 | @virat.kohli | 270000000 |
| Top Followed User | 2 | @shraddhakapoor | 93000000 |
| Top Followed User | 3 | @aliaabhatt | 85000000 |
| Top Liked Post | 1 | Historic victory with team India at Wankhede! 🇮🇳🏆 | 8500000 |
| Top Liked Post | 4 | Moments from the set of our next film ✨🎬 | 6400000 |
| Top Liked Post | 3 | Sunday smiles & rainy day coffee ☕🌸 | 5100000 |

---

## 4. Write a recursive CTE that generates a list of dates for the next 7 days starting from today, similar to how BookMyShow shows available dates for movie bookings.
*(Hint: Use a base case for today and recursion to add one day at a time.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Recursive CTE (`WITH RECURSIVE`), Anchor Member, Recursive Member & Termination Condition

---

### SQL Query:

#### **Recursive Date Generator for BookMyShow Movie Booking:**
```sql
WITH RECURSIVE MovieBookingDates AS (
    -- Anchor Member (Base Case): Today's date (Day 1)
    SELECT 
        CURDATE() AS booking_date,
        DAYNAME(CURDATE()) AS day_name,
        1 AS day_number
    
    UNION ALL
    
    -- Recursive Member: Increment date by 1 day until day_number reaches 7
    SELECT 
        DATE_ADD(booking_date, INTERVAL 1 DAY),
        DAYNAME(DATE_ADD(booking_date, INTERVAL 1 DAY)),
        day_number + 1
    FROM MovieBookingDates
    WHERE day_number < 7
)
-- Main query with BookMyShow style UI labels
SELECT 
    day_number,
    booking_date,
    day_name,
    CASE 
        WHEN day_number = 1 THEN 'TODAY'
        WHEN day_number = 2 THEN 'TOMORROW'
        ELSE UPPER(CONCAT(LEFT(day_name, 3), ', ', DATE_FORMAT(booking_date, '%d %b')))
    END AS booking_tab_label
FROM MovieBookingDates;
```

---

### How Recursive CTEs Work (Step-by-Step Anatomy):

1. **`WITH RECURSIVE` Declaration:**
   * Signals the SQL query engine that the expression will reference its own output name (`MovieBookingDates`).
2. **Anchor Member (Base Case):**
   * The first `SELECT` statement initializes the sequence. It executes exactly once and generates row 1 (`CURDATE()`, Day 1).
3. **`UNION ALL` Operator:**
   * Mandatory connector between the anchor member and recursive member.
4. **Recursive Member:**
   * References the CTE name (`FROM MovieBookingDates`).
   * Takes the previous row's `booking_date` and advances it: `DATE_ADD(booking_date, INTERVAL 1 DAY)`.
   * Increments `day_number + 1`.
5. **Termination Condition (`WHERE day_number < 7`):**
   * **Crucial Safety Rule:** Prevents infinite recursive loops. As soon as `day_number` reaches 7, the `WHERE` condition evaluates to false, terminating the recursion gracefully.

#### Output (Assuming today is 2026-11-04):
| day_number | booking_date | day_name | booking_tab_label |
|:---|:---|:---|:---|
| 1 | 2026-11-04 | Wednesday | TODAY |
| 2 | 2026-11-05 | Thursday | TOMORROW |
| 3 | 2026-11-06 | Friday | FRI, 06 NOV |
| 4 | 2026-11-07 | Saturday | SAT, 07 NOV |
| 5 | 2026-11-08 | Sunday | SUN, 08 NOV |
| 6 | 2026-11-09 | Monday | MON, 09 NOV |
| 7 | 2026-11-10 | Tuesday | TUE, 10 NOV |

---

## 5. Given a messy SQL query that finds all users with more than 1000 followers from a `Users` table, refactor it to use a CTE for better clarity and maintainability.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Refactoring Nested Anti-Patterns into Clean, Pipeline-Oriented CTEs

---

### The Problem: A Typical Messy Legacy Query

In production systems, ad-hoc nested queries often accumulate confusing aliases, redundant filters, and deeply embedded conditional statements:

```sql
-- ❌ MESSY / HARD-TO-MAINTAIN LEGACY QUERY
SELECT 
    t1.u_id,
    t1.u_name,
    t1.f_count,
    t1.status
FROM (
    SELECT 
        u.user_id AS u_id,
        UPPER(TRIM(u.username)) AS u_name,
        u.followers_count AS f_count,
        CASE 
            WHEN u.followers_count >= 100000 THEN 'Mega Creator'
            WHEN u.followers_count >= 10000 THEN 'Macro Influencer'
            WHEN u.followers_count >= 1000 THEN 'Micro Influencer'
            ELSE 'Nano / Regular'
        END AS status
    FROM (
        SELECT * 
        FROM Users 
        WHERE is_active = 1 AND is_verified = 1
    ) u
    WHERE u.followers_count IS NOT NULL
) t1
WHERE t1.f_count > 1000
ORDER BY t1.f_count DESC;
```

#### Why is the above query bad?
1. **Three levels of deep nesting:** Requires mental gymnastics to untangle what the query is actually computing.
2. **Cryptic aliases:** Column names are renamed unpredictably (`u_id`, `u_name`, `f_count`, `t1`).
3. **Scattered filtering:** `is_active` is filtered in the innermost subquery, `followers_count IS NOT NULL` in the middle subquery, and `f_count > 1000` in the outermost query.

---

### The Solution: Clean Refactored Query Using CTEs

```sql
-- ✅ CLEAN, REFACTORED CTE QUERY (Self-Documenting & Maintainable)

WITH VerifiedActiveUsers AS (
    -- Step 1: Filter verified and active accounts only
    SELECT 
        user_id,
        TRIM(username) AS username,
        followers_count
    FROM Users
    WHERE is_active = 1 
      AND is_verified = 1
      AND followers_count IS NOT NULL
),
InfluencerTierClassification AS (
    -- Step 2: Clean formatting and assign business category tiers
    SELECT 
        user_id,
        UPPER(username) AS clean_username,
        followers_count,
        CASE 
            WHEN followers_count >= 100000 THEN 'Mega Creator'
            WHEN followers_count >= 10000 THEN 'Macro Influencer'
            WHEN followers_count >= 1000 THEN 'Micro Influencer'
            ELSE 'Nano / Regular'
        END AS influencer_tier
    FROM VerifiedActiveUsers
)
-- Step 3: Final extraction of users exceeding 1,000 followers
SELECT 
    user_id,
    clean_username AS username,
    followers_count,
    influencer_tier
FROM InfluencerTierClassification
WHERE followers_count > 1000
ORDER BY followers_count DESC;
```

---

### Refactoring Benefits Summary:

1. **Step-by-Step Data Pipeline:**
   * Step 1 handles **Data Hygiene** (`is_active = 1`, `is_verified = 1`, non-null check).
   * Step 2 handles **Business Logic & Feature Engineering** (Influencer tier categorization).
   * Step 3 handles **Presentation & Final Filtering** (`followers_count > 1000`).
2. **Single Responsibility Principle in SQL:**
   * If marketing decides to change the tier threshold (e.g., changing Micro Influencer to 2,000), only `InfluencerTierClassification` needs an edit. The rest of the query remains untouched.
3. **Seamless Team Collaboration:**
   * New engineers or data analysts joining the team can immediately read CTE names and understand the pipeline without asking for query explanations.

#### Output:
| user_id | username | followers_count | influencer_tier |
|:---|:---|:---|:---|
| 1 | VIRAT.KOHLI | 270000000 | Mega Creator |
| 2 | SHRADDHAKAPOOR | 93000000 | Mega Creator |
| 3 | ALIAABHATT | 85000000 | Mega Creator |
| 4 | RANVEERSINGH | 47000000 | Mega Creator |
| 5 | BHUVAN.BAM22 | 19000000 | Mega Creator |
