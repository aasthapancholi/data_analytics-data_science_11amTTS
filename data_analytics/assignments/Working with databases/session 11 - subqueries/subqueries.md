# Module 2: Working with Databases - Session 11: Subqueries (WHERE, SELECT, IN & Nested Subqueries)

---

## 1. Create a SQL query using a subquery in the `WHERE` clause to find all restaurants from a `Restaurants` table whose average rating is higher than the average rating of all restaurants in the city.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Concept:** Correlated Subquery / Scalar Subquery in `WHERE` clause
* **Table Design:**
  * `Restaurants`:
    * `restaurant_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique identifier for each restaurant)
    * `name`: `VARCHAR(100) NOT NULL` (Restaurant name)
    * `city`: `VARCHAR(50) NOT NULL` (City location, e.g., 'Ahmedabad', 'Mumbai')
    * `rating`: `DECIMAL(3, 2) NOT NULL` (Average user rating, e.g., 4.50, 3.80)

#### SQL Setup Script:
```sql
-- 1. Create Restaurants table
CREATE TABLE Restaurants (
    restaurant_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL,
    rating DECIMAL(3, 2) NOT NULL
);

-- 2. Insert sample restaurants across different cities
INSERT INTO Restaurants (name, city, rating) VALUES
('Swati Snacks', 'Ahmedabad', 4.50),
('Gordhan Thal', 'Ahmedabad', 4.10),
('Manek Chowk Street Eats', 'Ahmedabad', 3.70),
('Agashiye', 'Ahmedabad', 4.60),
('Toran Dining Hall', 'Ahmedabad', 3.90),
('Bastian', 'Mumbai', 4.70),
('Bademiya', 'Mumbai', 3.80),
('Britannia & Co.', 'Mumbai', 4.20),
('The Bombay Canteen', 'Mumbai', 4.60),
('Sardar Refreshments', 'Mumbai', 3.50);

-- Verify table data
SELECT * FROM Restaurants;
```

---

### SQL Queries:

#### **Method A: Correlated Subquery (Evaluates for each restaurant against its OWN city's average rating)**
> *This is the most common industry and interview pattern. It dynamically calculates the average rating of the specific city where the restaurant operates and returns restaurants outperforming their local city benchmark.*

```sql
SELECT 
    r1.restaurant_id,
    r1.name AS restaurant_name,
    r1.city,
    r1.rating,
    ROUND((
        SELECT AVG(r2.rating) 
        FROM Restaurants r2 
        WHERE r2.city = r1.city
    ), 2) AS city_avg_rating
FROM Restaurants r1
WHERE r1.rating > (
    SELECT AVG(r2.rating)
    FROM Restaurants r2
    WHERE r2.city = r1.city
)
ORDER BY r1.city, r1.rating DESC;
```

#### **Method B: Single City Subquery (For a specified target city, e.g., 'Ahmedabad')**
> *If analyzing a single city's market at a time:*

```sql
SELECT 
    restaurant_id,
    name AS restaurant_name,
    city,
    rating
FROM Restaurants
WHERE city = 'Ahmedabad'
  AND rating > (
      SELECT AVG(rating)
      FROM Restaurants
      WHERE city = 'Ahmedabad'
  )
ORDER BY rating DESC;
```

---

### Step-by-Step Execution Breakdown:
1. **Inner Query (Subquery):**
   * For Method A, for every restaurant row in outer query `r1`, the subquery executes: `SELECT AVG(rating) FROM Restaurants r2 WHERE r2.city = r1.city`.
   * For 'Ahmedabad', City Average = $(4.50 + 4.10 + 3.70 + 4.60 + 3.90) / 5 = 4.16$.
   * For 'Mumbai', City Average = $(4.70 + 3.80 + 4.20 + 4.60 + 3.50) / 5 = 4.16$.
2. **Outer Query (WHERE evaluation):**
   * Only records whose `rating > city_avg_rating` pass the filter.
   * Ahmedabad restaurants passing: Swati Snacks (4.50), Agashiye (4.60).
   * Mumbai restaurants passing: Bastian (4.70), Britannia & Co. (4.20), The Bombay Canteen (4.60).

#### Output:
| restaurant_id | restaurant_name | city | rating | city_avg_rating |
|:---|:---|:---|:---|:---|
| 4 | Agashiye | Ahmedabad | 4.60 | 4.16 |
| 1 | Swati Snacks | Ahmedabad | 4.50 | 4.16 |
| 6 | Bastian | Mumbai | 4.70 | 4.16 |
| 9 | The Bombay Canteen | Mumbai | 4.60 | 4.16 |
| 8 | Britannia & Co. | Mumbai | 4.20 | 4.16 |

---

## 2. Write a SQL query that uses a subquery in the `SELECT` statement to display each user's name from a `Users` table along with the total number of orders they have placed from an `Orders` table, like a summary you might see in a Zomato user profile.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Concept:** Correlated Scalar Subquery in `SELECT` projection
* **Table Design:**
  * `Users`:
    * `user_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique user ID)
    * `name`: `VARCHAR(100) NOT NULL` (User's full name)
    * `email`: `VARCHAR(100) NOT NULL UNIQUE` (Contact email)
  * `Orders`:
    * `order_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique order ID)
    * `user_id`: `INT NOT NULL` (Foreign key linking to Users)
    * `order_date`: `DATE NOT NULL` (Date order was placed)
    * `total_amount`: `DECIMAL(10, 2) NOT NULL` (Order bill value)

#### SQL Setup Script:
```sql
-- 1. Create Users table
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE
);

-- 2. Create Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- 3. Insert sample Users
INSERT INTO Users (name, email) VALUES
('Aarav Sharma', 'aarav@gmail.com'),
('Diya Patel', 'diya@gmail.com'),
('Rohan Verma', 'rohan@gmail.com'),
('Pooja Mehta', 'pooja@gmail.com'),
('Karan Singhania', 'karan@gmail.com');

-- 4. Insert sample Orders (Notice: Karan has 0 orders, Diya has 3 orders)
INSERT INTO Orders (user_id, order_date, total_amount) VALUES
(1, '2026-10-01', 350.00),
(1, '2026-10-15', 520.00),
(2, '2026-10-02', 890.00),
(2, '2026-10-18', 240.00),
(2, '2026-10-25', 1150.00),
(3, '2026-10-12', 450.00),
(4, '2026-10-20', 600.00);
```

---

### SQL Queries:

#### **Primary Query: Subquery in `SELECT` Statement**
```sql
SELECT 
    u.user_id,
    u.name AS user_name,
    (
        SELECT COUNT(*)
        FROM Orders o
        WHERE o.user_id = u.user_id
    ) AS total_orders
FROM Users u
ORDER BY total_orders DESC, u.name ASC;
```

---

### Detailed Technical Explanation:
1. **Scalar Subquery Behavior:**
   * A subquery in the `SELECT` clause must return exactly **one value** (scalar) per row of the outer query.
   * `COUNT(*)` in the inner query aggregates rows matching `o.user_id = u.user_id`.
2. **Handling Users With Zero Orders:**
   * Unlike an `INNER JOIN` (which drops users with no orders) or `SUM()` (which returns `NULL`), `COUNT(*)` on zero matching rows cleanly returns `0`. This is the exact behavior seen on consumer profiles like Zomato/Swiggy.
3. **Comparison with `LEFT JOIN` Alternative:**
   ```sql
   -- Equivalent query using LEFT JOIN & GROUP BY:
   SELECT 
       u.user_id,
       u.name AS user_name,
       COUNT(o.order_id) AS total_orders
   FROM Users u
   LEFT JOIN Orders o ON u.user_id = o.user_id
   GROUP BY u.user_id, u.name
   ORDER BY total_orders DESC;
   ```
   * *When to use which?* The `SELECT` subquery is concise and reads naturally when projecting summary counters for a dashboard or profile view. A `LEFT JOIN` is typically more performant on very large datasets when multiple aggregated metrics (`SUM(amount)`, `AVG(amount)`) are needed simultaneously.

#### Output:
| user_id | user_name | total_orders |
|:---|:---|:---|
| 2 | Diya Patel | 3 |
| 1 | Aarav Sharma | 2 |
| 4 | Pooja Mehta | 1 |
| 3 | Rohan Verma | 1 |
| 5 | Karan Singhania | 0 |

---

## 3. Given a `Movies` table and a `Reviews` table, write a SQL query using `IN` with a subquery to list all movies that have at least one review with a rating of 5 stars, as seen in BookMyShow's top-rated section.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Concept:** Multi-Row Subquery with `IN` Membership Operator
* **Table Design:**
  * `Movies`:
    * `movie_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique movie identifier)
    * `title`: `VARCHAR(150) NOT NULL` (Movie title)
    * `genre`: `VARCHAR(50) NOT NULL` (Genre category)
    * `release_year`: `INT NOT NULL` (Year of release)
  * `Reviews`:
    * `review_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique review ID)
    * `movie_id`: `INT NOT NULL` (Foreign key referencing Movies)
    * `user_name`: `VARCHAR(100) NOT NULL` (Reviewer name)
    * `rating`: `INT NOT NULL` (Star rating from 1 to 5)
    * `review_comment`: `TEXT` (Optional feedback comment)

#### SQL Setup Script:
```sql
-- 1. Create Movies table
CREATE TABLE Movies (
    movie_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(150) NOT NULL,
    genre VARCHAR(50) NOT NULL,
    release_year INT NOT NULL
);

-- 2. Create Reviews table
CREATE TABLE Reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    movie_id INT NOT NULL,
    user_name VARCHAR(100) NOT NULL,
    rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    review_comment TEXT,
    FOREIGN KEY (movie_id) REFERENCES Movies(movie_id) ON DELETE CASCADE
);

-- 3. Insert sample Movies
INSERT INTO Movies (title, genre, release_year) VALUES
('Kantara: A Legend', 'Action / Drama', 2022),
('Oppenheimer', 'Biography / Drama', 2023),
('Interstellar', 'Sci-Fi / Adventure', 2014),
('Fighter', 'Action / Thriller', 2024),
('Bade Miyan Chote Miyan', 'Action / Comedy', 2024);

-- 4. Insert sample Reviews
INSERT INTO Reviews (movie_id, user_name, rating, review_comment) VALUES
(1, 'Rahul S.', 5, 'Absolute masterpiece! Divine climax.'),
(1, 'Priya K.', 4, 'Great visuals and music.'),
(2, 'Aditya M.', 5, 'Nolan does it again! Outstanding performance.'),
(2, 'Sneha T.', 5, 'Brilliant pacing and cinematography.'),
(3, 'Vikas G.', 5, 'Mind-bending cinematic experience.'),
(4, 'Ankit B.', 3, 'Average action, predictable plot.'),
(4, 'Neha P.', 4, 'Good aerial scenes.'),
(5, 'Ritu D.', 2, 'Disappointing storyline.');
```

---

### SQL Queries:

#### **Primary Query: Using `IN` with a Subquery**
```sql
SELECT 
    movie_id,
    title,
    genre,
    release_year
FROM Movies
WHERE movie_id IN (
    -- Subquery: List of all movie_ids that have earned at least one 5-star rating
    SELECT DISTINCT movie_id
    FROM Reviews
    WHERE rating = 5
)
ORDER BY title ASC;
```

---

### How the `IN` Operator and Subquery Work:
1. **Subquery Execution (Inner):**
   * Evaluates `SELECT DISTINCT movie_id FROM Reviews WHERE rating = 5`.
   * Resulting set: `{1, 2, 3}` (representing *Kantara*, *Oppenheimer*, and *Interstellar*).
2. **Outer Query Filtering:**
   * SQL checks each row in `Movies`:
     * `movie_id = 1`: In set `{1, 2, 3}`? $\rightarrow$ **TRUE** (Kept)
     * `movie_id = 2`: In set `{1, 2, 3}`? $\rightarrow$ **TRUE** (Kept)
     * `movie_id = 3`: In set `{1, 2, 3}`? $\rightarrow$ **TRUE** (Kept)
     * `movie_id = 4`: In set `{1, 2, 3}`? $\rightarrow$ **FALSE** (Excluded)
     * `movie_id = 5`: In set `{1, 2, 3}`? $\rightarrow$ **FALSE** (Excluded)
3. **Why `DISTINCT` in the Subquery is Recommended:**
   * Although `IN` automatically treats the subquery result as a mathematical set, adding `DISTINCT` prevents passing redundant duplicate keys in memory, especially in large production tables with millions of reviews.

#### Output:
| movie_id | title | genre | release_year |
|:---|:---|:---|:---|
| 1 | Kantara: A Legend | Action / Drama | 2022 |
| 3 | Interstellar | Sci-Fi / Adventure | 2014 |
| 2 | Oppenheimer | Biography / Drama | 2023 |

---

## 4. Write a nested SQL query to find the names of all sellers from a `Sellers` table on a Flipkart-style platform who have sold products in every category listed in a `Categories` table.
*(Hint: Use nested subqueries to compare seller's categories with the complete list of categories.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Concept:** Relational Division using Nested Subqueries
* **Table Design:**
  * `Sellers`:
    * `seller_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique seller identifier)
    * `seller_name`: `VARCHAR(100) NOT NULL` (Registered seller/store name)
    * `rating`: `DECIMAL(2, 1) NOT NULL` (Seller performance rating)
  * `Categories`:
    * `category_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Category identifier)
    * `category_name`: `VARCHAR(100) NOT NULL UNIQUE` (e.g., 'Electronics', 'Fashion', 'Home Decor', 'Books')
  * `Products`:
    * `product_id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique product ID)
    * `seller_id`: `INT NOT NULL` (Seller offering this product)
    * `category_id`: `INT NOT NULL` (Category the product belongs to)
    * `product_name`: `VARCHAR(150) NOT NULL`
    * `price`: `DECIMAL(10, 2) NOT NULL`
    * Foreign keys referencing `Sellers(seller_id)` and `Categories(category_id)`

#### SQL Setup Script:
```sql
-- 1. Create Sellers table
CREATE TABLE Sellers (
    seller_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_name VARCHAR(100) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL
);

-- 2. Create Categories table
CREATE TABLE Categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL UNIQUE
);

-- 3. Create Products table
CREATE TABLE Products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_id INT NOT NULL,
    category_id INT NOT NULL,
    product_name VARCHAR(150) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (seller_id) REFERENCES Sellers(seller_id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES Categories(category_id) ON DELETE CASCADE
);

-- 4. Insert 4 platform categories
INSERT INTO Categories (category_name) VALUES
('Electronics'), -- category_id: 1
('Fashion'),     -- category_id: 2
('Home Decor'),  -- category_id: 3
('Books');       -- category_id: 4

-- 5. Insert Sellers
INSERT INTO Sellers (seller_name, rating) VALUES
('SuperComNet Retail', 4.8),     -- Has products in ALL 4 categories
('OmniTech Logistics', 4.5),     -- Has products in 3 categories (Missing Books)
('RetailNet India', 4.9),        -- Has products in ALL 4 categories
('Trendz Apparel Hub', 4.1);     -- Has products in 1 category only (Fashion)

-- 6. Insert Products mapping sellers to categories
-- SuperComNet Retail (seller_id = 1): Covers Categories 1, 2, 3, 4
INSERT INTO Products (seller_id, category_id, product_name, price) VALUES
(1, 1, 'Bluetooth Headphones', 1999.00),
(1, 2, 'Cotton Polo T-Shirt', 699.00),
(1, 3, 'LED Table Lamp', 899.00),
(1, 4, 'Atomic Habits (Book)', 450.00);

-- OmniTech Logistics (seller_id = 2): Covers Categories 1, 2, 3 (Missing 4: Books)
INSERT INTO Products (seller_id, category_id, product_name, price) VALUES
(2, 1, 'Wireless Mouse', 499.00),
(2, 2, 'Running Shoes', 1499.00),
(2, 3, 'Wall Clock', 599.00);

-- RetailNet India (seller_id = 3): Covers Categories 1, 2, 3, 4
INSERT INTO Products (seller_id, category_id, product_name, price) VALUES
(3, 1, 'USB-C Fast Charger', 599.00),
(3, 2, 'Denim Jeans', 1299.00),
(3, 3, 'Cushion Covers Set', 399.00),
(3, 4, 'The Psychology of Money', 350.00);

-- Trendz Apparel Hub (seller_id = 4): Covers Category 2 only
INSERT INTO Products (seller_id, category_id, product_name, price) VALUES
(4, 2, 'Casual Shirt', 799.00),
(4, 2, 'Leather Jacket', 3499.00);
```

---

### SQL Queries:

#### **Method 1: Nested Subquery with `COUNT(DISTINCT)` Comparison (Matches Hint directly)**
> *This method counts the distinct categories the seller sells in via an inner subquery and compares that directly with a second subquery that retrieves the total category count from the platform.*

```sql
SELECT 
    s.seller_id,
    s.seller_name,
    s.rating
FROM Sellers s
WHERE (
    -- Subquery 1: Number of distinct categories this specific seller offers
    SELECT COUNT(DISTINCT p.category_id)
    FROM Products p
    WHERE p.seller_id = s.seller_id
) = (
    -- Subquery 2: Total number of all categories listed on Flipkart
    SELECT COUNT(*) 
    FROM Categories
)
ORDER BY s.seller_id ASC;
```

---

#### **Method 2: Double `NOT EXISTS` (Formal Relational Division)**
> *The classic theoretical relational division query: "Find sellers such that there does NOT exist a category that this seller has NOT sold products in."*

```sql
SELECT 
    s.seller_id,
    s.seller_name,
    s.rating
FROM Sellers s
WHERE NOT EXISTS (
    -- Outer nested subquery: Categories
    SELECT c.category_id
    FROM Categories c
    WHERE NOT EXISTS (
        -- Inner nested subquery: Check if the seller has a product in this category
        SELECT 1
        FROM Products p
        WHERE p.seller_id = s.seller_id
          AND p.category_id = c.category_id
    )
)
ORDER BY s.seller_id ASC;
```

---

### How Method 1 Executes (Step-by-Step):
1. **Benchmark Subquery:**
   * `SELECT COUNT(*) FROM Categories` yields `4`.
2. **Evaluation for each Seller in `Sellers`:**
   * **SuperComNet Retail (`seller_id = 1`):** Distinct categories sold = `{1, 2, 3, 4}` $\rightarrow$ Count = `4`. Since $4 = 4$, **Included**.
   * **OmniTech Logistics (`seller_id = 2`):** Distinct categories sold = `{1, 2, 3}` $\rightarrow$ Count = `3`. Since $3 \neq 4$, **Excluded**.
   * **RetailNet India (`seller_id = 3`):** Distinct categories sold = `{1, 2, 3, 4}` $\rightarrow$ Count = `4`. Since $4 = 4$, **Included**.
   * **Trendz Apparel Hub (`seller_id = 4`):** Distinct categories sold = `{2}` $\rightarrow$ Count = `1`. Since $1 \neq 4$, **Excluded**.

#### Output:
| seller_id | seller_name | rating |
|:---|:---|:---|
| 1 | SuperComNet Retail | 4.8 |
| 3 | RetailNet India | 4.9 |
