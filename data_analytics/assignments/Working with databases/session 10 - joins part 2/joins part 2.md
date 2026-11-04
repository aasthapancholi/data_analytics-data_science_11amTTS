# Module 2: Working with Databases - Session 10: Advanced JOINs (FULL, SELF, Multi-Table JOINs & Optimization)

---

## 1. Create two tables: `Influencers` (`id`, `name`) and `Collaborations` (`id`, `influencer1_id`, `influencer2_id`, `collab_date`). Write a SQL FULL JOIN query to list all influencers and show their collaboration partner names if any, including influencers with no collaborations.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `Influencers`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique identifier for the creator)
    * `name`: `VARCHAR(100) NOT NULL` (Creator / handle name)
  * `Collaborations`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique collaboration ID)
    * `influencer1_id`: `INT NOT NULL` (First collaborator ID)
    * `influencer2_id`: `INT NOT NULL` (Second collaborator ID)
    * `collab_date`: `DATE NOT NULL` (Date of campaign/post)

#### SQL Setup Script:
```sql
-- 1. Create Influencers table
CREATE TABLE Influencers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);

-- 2. Create Collaborations table
CREATE TABLE Collaborations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    influencer1_id INT NOT NULL,
    influencer2_id INT NOT NULL,
    collab_date DATE NOT NULL
);

-- 3. Insert sample influencers
INSERT INTO Influencers (name) VALUES
('Virat Kohli'),      -- id: 1
('Anushka Sharma'),  -- id: 2
('Ranveer Singh'),   -- id: 3
('Deepika Padukone'),-- id: 4
('Bhuvan Bam');      -- id: 5 (Solo creator, no collaborations)

-- 4. Insert collaborations
INSERT INTO Collaborations (influencer1_id, influencer2_id, collab_date) VALUES
(1, 2, '2026-05-10'),
(3, 4, '2026-06-15'),
(1, 3, '2026-08-20');
```

---

### SQL Queries:

#### **Standard ANSI SQL (`FULL OUTER JOIN`):**
*(Supported in PostgreSQL, SQL Server, Oracle, SQLite 3.39+)*
```sql
SELECT 
    i.id AS influencer_id,
    i.name AS influencer_name,
    p.name AS partner_name,
    c.collab_date
FROM Influencers i
FULL OUTER JOIN Collaborations c 
    ON i.id = c.influencer1_id
FULL OUTER JOIN Influencers p 
    ON c.influencer2_id = p.id;
```

#### **MySQL / phpMyAdmin Compatible Implementation (Simulating FULL JOIN via `UNION`):**
> *Note for XAMPP users:* MySQL does not support the `FULL JOIN` keyword natively. In MySQL, a FULL JOIN is executed by taking the `UNION` of a `LEFT JOIN` and a `RIGHT JOIN`:

```sql
SELECT 
    i.name AS influencer_name,
    p.name AS partner_name,
    c.collab_date
FROM Influencers i
LEFT JOIN Collaborations c ON i.id = c.influencer1_id
LEFT JOIN Influencers p ON c.influencer2_id = p.id

UNION

SELECT 
    i.name AS influencer_name,
    p.name AS partner_name,
    c.collab_date
FROM Influencers i
RIGHT JOIN Collaborations c ON i.id = c.influencer2_id
LEFT JOIN Influencers p ON c.influencer1_id = p.id;
```

#### Output:
| influencer_name | partner_name | collab_date |
|:---|:---|:---|
| Virat Kohli | Anushka Sharma | 2026-05-10 |
| Virat Kohli | Ranveer Singh | 2026-08-20 |
| Ranveer Singh | Deepika Padukone | 2026-06-15 |
| Anushka Sharma | Virat Kohli | 2026-05-10 |
| Deepika Padukone | Ranveer Singh | 2026-06-15 |
| Bhuvan Bam | *NULL* | *NULL* |

*(Notice how `Bhuvan Bam` is retained in the output despite having zero collaborations).*

---

## 2. Using a SELF JOIN, write a query on a table called `Playlists` (`id`, `user_id`, `playlist_name`, `parent_playlist_id`) to display each playlist alongside its parent playlist name, similar to how Spotify shows nested playlists.
*(Hint: Join Playlists with itself on parent_playlist_id = id.)*

### SOLUTION:
* **Concept:** A `SELF JOIN` joins a table to itself using table aliases.
  * `child`: Represents the current sub-playlist.
  * `parent`: Represents the containing playlist or folder.
  * We use `LEFT JOIN` so top-level root playlists (where `parent_playlist_id IS NULL`) are not discarded.

#### SQL Setup Script:
```sql
CREATE TABLE Playlists (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    playlist_name VARCHAR(100) NOT NULL,
    parent_playlist_id INT NULL
);

INSERT INTO Playlists (user_id, playlist_name, parent_playlist_id) VALUES
(101, 'My Music Library', NULL),  -- Top-level root folder (id: 1)
(101, 'Workout Hits', 1),         -- Child of folder 1 (id: 2)
(101, 'Cardio Beats', 2),         -- Nested child of playlist 2 (id: 3)
(101, 'Chill Vibes', 1),          -- Child of folder 1 (id: 4)
(101, 'Focus & Study', NULL);     -- Top-level root folder (id: 5)
```

#### SQL Query:
```sql
SELECT 
    child.id AS playlist_id,
    child.playlist_name AS playlist_name,
    COALESCE(parent.playlist_name, 'None (Root Playlist)') AS parent_playlist_name
FROM Playlists child
LEFT JOIN Playlists parent 
    ON child.parent_playlist_id = parent.id;
```

#### Output:
| playlist_id | playlist_name | parent_playlist_name |
|:---|:---|:---|
| 1 | My Music Library | None (Root Playlist) |
| 2 | Workout Hits | My Music Library |
| 3 | Cardio Beats | Workout Hits |
| 4 | Chill Vibes | My Music Library |
| 5 | Focus & Study | None (Root Playlist) |

---

## 3. Given three tables: `Users` (`id`, `username`), `Orders` (`id`, `user_id`, `order_date`), and `Payments` (`id`, `order_id`, `amount`), write a SQL query using multiple JOINs to display each username, their order date, and payment amount, showing all users even if they have no orders or payments.

### SOLUTION:
* **Concept:** Multi-table outer joins require chaining `LEFT JOIN`s from the primary entity (`Users` &rarr; `Orders` &rarr; `Payments`).
  * If an `INNER JOIN` was used at any stage, users who have never placed an order or whose orders have not completed payment would be eliminated from the final output.

#### SQL Setup Script:
```sql
CREATE TABLE Users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL
);

CREATE TABLE Orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    order_date DATE NOT NULL
);

CREATE TABLE Payments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
);

INSERT INTO Users (username) VALUES ('Aarav'), ('Diya'), ('Rohan'), ('Pooja');
INSERT INTO Orders (user_id, order_date) VALUES (1, '2026-10-01'), (2, '2026-10-02');
INSERT INTO Payments (order_id, amount) VALUES (1, 1499.00); -- Order 1 paid, Order 2 unpaid, Rohan & Pooja have no orders
```

#### SQL Query:
```sql
SELECT 
    u.id AS user_id,
    u.username,
    o.order_date,
    p.amount AS payment_amount
FROM Users u
LEFT JOIN Orders o 
    ON u.id = o.user_id
LEFT JOIN Payments p 
    ON o.id = p.order_id;
```

#### Output:
| user_id | username | order_date | payment_amount |
|:---|:---|:---|:---|
| 1 | Aarav | 2026-10-01 | 1499.00 |
| 2 | Diya | 2026-10-02 | *NULL* |
| 3 | Rohan | *NULL* | *NULL* |
| 4 | Pooja | *NULL* | *NULL* |

---

## 4. You notice that your JOIN query between Zomato's Restaurants and Reviews tables is returning duplicate rows for some restaurants. Modify your query to eliminate duplicates and explain in one line why the duplicates were happening.
*(Hint: Use DISTINCT or GROUP BY and consider the relationship between restaurants and reviews.)*

### SOLUTION:

#### One-Line Explanation:
> **Why duplicates happen:** *Duplicates occur because Restaurants and Reviews share a One-to-Many ($1:N$) relationship, causing SQL to duplicate the restaurant row for every matching review found in the Reviews table.*

---

#### Fix Option A: Using `DISTINCT` (To return unique restaurants that have reviews)
```sql
SELECT DISTINCT 
    r.id AS restaurant_id,
    r.name AS restaurant_name,
    r.city
FROM Restaurants r
INNER JOIN Reviews rev 
    ON r.id = rev.restaurant_id;
```

#### Fix Option B: Using `GROUP BY` (Best practice when aggregating review metrics)
```sql
SELECT 
    r.id AS restaurant_id,
    r.name AS restaurant_name,
    r.city,
    COUNT(rev.id) AS total_reviews,
    ROUND(AVG(rev.rating), 1) AS avg_rating
FROM Restaurants r
INNER JOIN Reviews rev 
    ON r.id = rev.restaurant_id
GROUP BY r.id, r.name, r.city;
```

#### Output:
| restaurant_id | restaurant_name | city | total_reviews | avg_rating |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Ahmedabad | 15 | 4.6 |
| 2 | Little Italy | Surat | 8 | 4.3 |

---

## 5. Write two different JOIN queries on a Products and Categories table (like Flipkart) to list all products with their category names, but use different join conditions in each. Briefly explain which join condition is more efficient and why.

### SOLUTION:

#### **Query 1: Integer Primary Key & Foreign Key Join (`category_id = id`)**
```sql
SELECT 
    p.id AS product_id,
    p.product_name,
    p.price,
    c.category_name
FROM Products p
INNER JOIN Categories c 
    ON p.category_id = c.id;
```

#### **Query 2: String/Text Natural Join Condition (`category_name = category_name`)**
```sql
SELECT 
    p.id AS product_id,
    p.product_name,
    p.price,
    c.category_name
FROM Products p
INNER JOIN Categories c 
    ON p.category_name = c.category_name;
```

---

### Efficiency Comparison & Explanation:

#### **Winner:** **Query 1 (`ON p.category_id = c.id`) is significantly more efficient.**

#### **Why:**
1. **Numeric vs. String Comparison:** Integer comparisons are fixed in size (4 bytes for `INT`) and execute directly at the CPU register level in single cycles. String matching requires variable-length character-by-character validation and collation checks.
2. **Index Performance & B-Tree Traversal:** Numeric Primary Keys (`c.id`) and Foreign Keys (`p.category_id`) are indexed by default as clustered or non-clustered B-Trees. String indexes consume significantly more memory, buffer cache, and disk I/O.
3. **Database Normalization:** Joining on surrogate integer keys ensures referential integrity and avoids full table scans caused by character discrepancies, spaces, or case mismatches.
