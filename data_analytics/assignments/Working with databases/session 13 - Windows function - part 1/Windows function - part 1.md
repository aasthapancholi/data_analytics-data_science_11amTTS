# Module 2: Working with Databases - Session 13: Window Functions - Part 1 (OVER, ROW_NUMBER, RANK, DENSE_RANK & PARTITION BY)

---

## 1. Create a table named `Playlists` with columns: `id`, `user_id`, `playlist_name`, and `total_likes`. Insert at least 8 sample rows with different users and playlists, making sure some playlists have the same `user_id`.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Table Design with Grouped Foreign Relationships & Ranking Preparation
* **Table Design:**
  * `Playlists`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique playlist identifier)
    * `user_id`: `INT NOT NULL` (Spotify user account identifier)
    * `playlist_name`: `VARCHAR(100) NOT NULL` (Title of the playlist)
    * `total_likes`: `INT NOT NULL` (Number of followers/likes saved by listeners)

#### SQL Setup Script:
```sql
-- 1. Create Playlists table
CREATE TABLE Playlists (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    playlist_name VARCHAR(100) NOT NULL,
    total_likes INT NOT NULL
);

-- 2. Insert 9 sample rows across multiple users with intentional metric ties
INSERT INTO Playlists (user_id, playlist_name, total_likes) VALUES
-- User 101 (Aarav): Has a tie at 12,500 likes
(101, 'Bollywood Chill Vibes', 12500),
(101, 'Punjabi Workout Beats', 12500),
(101, 'Retro 90s Classics', 8400),

-- User 102 (Diya): Global top score & tie at 9,800 likes
(102, 'Indie Acoustic Coffeehouse', 15200),
(102, 'Lofi Beats for Study', 9800),
(102, 'Late Night Rain & Jazz', 9800),

-- User 103 (Rohan): Global tie with Diya at 15,200 likes
(103, 'EDM Festival Anthems', 15200),
(103, 'Midnight Drive Synthwave', 4300),
(103, 'Deep Focus Alpha Waves', 2100);

-- 3. Retrieve all records to verify insertion
SELECT * FROM Playlists;
```

#### Output (Initial Table State):
| id | user_id | playlist_name | total_likes |
|:---|:---|:---|:---|
| 1 | 101 | Bollywood Chill Vibes | 12500 |
| 2 | 101 | Punjabi Workout Beats | 12500 |
| 3 | 101 | Retro 90s Classics | 8400 |
| 4 | 102 | Indie Acoustic Coffeehouse | 15200 |
| 5 | 102 | Lofi Beats for Study | 9800 |
| 6 | 102 | Late Night Rain & Jazz | 9800 |
| 7 | 103 | EDM Festival Anthems | 15200 |
| 8 | 103 | Midnight Drive Synthwave | 4300 |
| 9 | 103 | Deep Focus Alpha Waves | 2100 |

---

## 2. Write a SQL query using `ROW_NUMBER()` and the `OVER()` clause to assign a unique row number to each playlist, ordered by `total_likes` in descending order.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Window Ranking Function `ROW_NUMBER()` across the entire table dataset

---

### SQL Query:
```sql
SELECT 
    ROW_NUMBER() OVER (ORDER BY total_likes DESC) AS row_num,
    id AS playlist_id,
    playlist_name,
    user_id,
    total_likes
FROM Playlists;
```

---

### How `ROW_NUMBER()` Works:
1. **Window Definition:**
   * The clause `OVER (ORDER BY total_likes DESC)` instructs SQL to open a window covering the entire table and sort the records from the highest likes to the lowest likes.
2. **Sequential Unbroken Numbering:**
   * `ROW_NUMBER()` assigns a strict, incremental integer ($1, 2, 3, 4, \dots$) to every row.
   * **Crucial Rule:** Even when two playlists share the exact same number of likes (e.g. *Indie Acoustic Coffeehouse* and *EDM Festival Anthems* both have 15,200 likes), `ROW_NUMBER()` guarantees distinct consecutive numbers (1 and 2). It **never creates duplicates or skips numbers**.

#### Output:
| row_num | playlist_id | playlist_name | user_id | total_likes |
|:---|:---|:---|:---|:---|
| 1 | 4 | Indie Acoustic Coffeehouse | 102 | 15200 |
| 2 | 7 | EDM Festival Anthems | 103 | 15200 |
| 3 | 1 | Bollywood Chill Vibes | 101 | 12500 |
| 4 | 2 | Punjabi Workout Beats | 101 | 12500 |
| 5 | 5 | Lofi Beats for Study | 102 | 9800 |
| 6 | 6 | Late Night Rain & Jazz | 102 | 9800 |
| 7 | 3 | Retro 90s Classics | 101 | 8400 |
| 8 | 8 | Midnight Drive Synthwave | 103 | 4300 |
| 9 | 9 | Deep Focus Alpha Waves | 103 | 2100 |

---

## 3. Use the `RANK()` function with the `OVER()` clause to rank all playlists by `total_likes`, and display the `playlist_name`, `user_id`, `total_likes`, and their rank.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Competition Ranking with Gaps using `RANK()`

---

### SQL Query:
```sql
SELECT 
    RANK() OVER (ORDER BY total_likes DESC) AS playlist_rank,
    playlist_name,
    user_id,
    total_likes
FROM Playlists;
```

---

### How `RANK()` Works (Olympic / Competition Style):
1. **Handling Ties:**
   * When multiple rows share identical ordering values (`total_likes`), `RANK()` assigns them the **same rank number**.
   * Example: Both *Indie Acoustic Coffeehouse* and *EDM Festival Anthems* share 15,200 likes, so both are assigned **Rank 1**.
2. **Skipping Ranks (Gaps):**
   * After a tie, `RANK()` skips subsequent ranks based on the count of tied records.
   * Since two items tied for Rank 1, **Rank 2 is skipped**. The next playlists (*Bollywood Chill Vibes* and *Punjabi Workout Beats* at 12,500 likes) receive **Rank 3**.
   * Since two items tied for Rank 3, **Rank 4 is skipped**, so the next rank assigned is **Rank 5**.

#### Output:
| playlist_rank | playlist_name | user_id | total_likes |
|:---|:---|:---|:---|
| 1 | Indie Acoustic Coffeehouse | 102 | 15200 |
| 1 | EDM Festival Anthems | 103 | 15200 |
| 3 | Bollywood Chill Vibes | 101 | 12500 |
| 3 | Punjabi Workout Beats | 101 | 12500 |
| 5 | Lofi Beats for Study | 102 | 9800 |
| 5 | Late Night Rain & Jazz | 102 | 9800 |
| 7 | Retro 90s Classics | 101 | 8400 |
| 8 | Midnight Drive Synthwave | 103 | 4300 |
| 9 | Deep Focus Alpha Waves | 103 | 2100 |

---

## 4. Write a SQL query using `DENSE_RANK()` and `PARTITION BY user_id` to rank each user's playlists by `total_likes`, showing `playlist_name`, `user_id`, `total_likes`, and dense rank.
*(Hint: This will show how popular each playlist is within each user's account, similar to how Spotify might rank your top playlists.)*

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** Window Partitioning (`PARTITION BY`) combined with Consecutive Ranking (`DENSE_RANK()`)

---

### SQL Query:
```sql
SELECT 
    user_id,
    DENSE_RANK() OVER (
        PARTITION BY user_id 
        ORDER BY total_likes DESC
    ) AS user_dense_rank,
    playlist_name,
    total_likes
FROM Playlists
ORDER BY user_id ASC, user_dense_rank ASC;
```

---

### Key Technical Insights:

1. **`PARTITION BY user_id`:**
   * Divides the entire dataset into independent logical partitions (buckets) per `user_id`.
   * The ranking counter resets to **1** for every distinct user.
2. **`DENSE_RANK()` (No Gaps):**
   * Assigns identical ranks to tied values without skipping subsequent ranks.
   * For User 101:
     * *Bollywood Chill Vibes* (12,500) $\rightarrow$ Rank 1
     * *Punjabi Workout Beats* (12,500) $\rightarrow$ Rank 1
     * *Retro 90s Classics* (8,400) $\rightarrow$ **Rank 2** *(Notice Rank 2 is NOT skipped!)*

---

### Comprehensive Comparison: `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()`

| Function | Ties Behavior | Gaps in Numbering? | Real-World Analogy |
|:---|:---|:---|:---|
| **`ROW_NUMBER()`** | Arbitrary sequential order | **Never** skips ($1, 2, 3, 4$) | Pagination / Indexing row items |
| **`RANK()`** | Same rank for ties | **Yes**, skips numbers ($1, 1, 3, 5$) | Olympic medals / Leaderboard standings |
| **`DENSE_RANK()`** | Same rank for ties | **Never** skips ($1, 1, 2, 3$) | Tier levels / Class score percentiles |

#### Output:
| user_id | user_dense_rank | playlist_name | total_likes |
|:---|:---|:---|:---|
| 101 | 1 | Bollywood Chill Vibes | 12500 |
| 101 | 1 | Punjabi Workout Beats | 12500 |
| 101 | 2 | Retro 90s Classics | 8400 |
| 102 | 1 | Indie Acoustic Coffeehouse | 15200 |
| 102 | 2 | Lofi Beats for Study | 9800 |
| 102 | 2 | Late Night Rain & Jazz | 9800 |
| 103 | 1 | EDM Festival Anthems | 15200 |
| 103 | 2 | Midnight Drive Synthwave | 4300 |
| 103 | 3 | Deep Focus Alpha Waves | 2100 |

---

## 5. Imagine you want to show the top 2 playlists per user based on `total_likes`, like Spotify's 'Your Top Playlists' feature. Write a query using a window function to select only the top 2 playlists for each user.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL 8.0+)
* **Concept:** CTE / Subquery Filtering with Window Ranking (Top-N Per Group)

---

### SQL Execution Order Rule: Why a CTE or Subquery is Required
> **Fundamental SQL Rule:** *Window functions cannot be evaluated directly inside the `WHERE` or `HAVING` clause!*
>
> **SQL Order of Execution:**
> 1. `FROM`
> 2. `WHERE` *(Row-level filtering)*
> 3. `GROUP BY`
> 4. `HAVING`
> 5. **`SELECT` (Window functions are computed here)**
> 6. `ORDER BY`
> 7. `LIMIT`
>
> Because `WHERE` runs **before** `SELECT`, the database has not yet calculated the window ranking. We must compute the ranking in a CTE first, and then filter in the outer query.

---

### SQL Query:

#### **Method A: Using a CTE with `DENSE_RANK()` (Recommended for Spotify Ties)**
> *If a user has multiple playlists tied for 1st or 2nd place, `DENSE_RANK() <= 2` ensures all top-tier favorites are included.*

```sql
WITH RankedUserPlaylists AS (
    SELECT 
        id AS playlist_id,
        user_id,
        playlist_name,
        total_likes,
        DENSE_RANK() OVER (
            PARTITION BY user_id 
            ORDER BY total_likes DESC
        ) AS rank_pos
    FROM Playlists
)
SELECT 
    user_id,
    rank_pos,
    playlist_name,
    total_likes
FROM RankedUserPlaylists
WHERE rank_pos <= 2
ORDER BY user_id ASC, rank_pos ASC, total_likes DESC;
```

#### **Method B: Using a CTE with `ROW_NUMBER()` (Strict Top 2 Limit)**
> *If Spotify's UI layout strictly has 2 card slots per user and cannot expand for ties, use `ROW_NUMBER()`:*

```sql
WITH StrictTopTwoPlaylists AS (
    SELECT 
        id AS playlist_id,
        user_id,
        playlist_name,
        total_likes,
        ROW_NUMBER() OVER (
            PARTITION BY user_id 
            ORDER BY total_likes DESC, id ASC
        ) AS row_num
    FROM Playlists
)
SELECT 
    user_id,
    row_num,
    playlist_name,
    total_likes
FROM StrictTopTwoPlaylists
WHERE row_num <= 2
ORDER BY user_id ASC, row_num ASC;
```

#### Output (Method A - Including Top-2 Tiers):
| user_id | rank_pos | playlist_name | total_likes |
|:---|:---|:---|:---|
| 101 | 1 | Bollywood Chill Vibes | 12500 |
| 101 | 1 | Punjabi Workout Beats | 12500 |
| 101 | 2 | Retro 90s Classics | 8400 |
| 102 | 1 | Indie Acoustic Coffeehouse | 15200 |
| 102 | 2 | Lofi Beats for Study | 9800 |
| 102 | 2 | Late Night Rain & Jazz | 9800 |
| 103 | 1 | EDM Festival Anthems | 15200 |
| 103 | 2 | Midnight Drive Synthwave | 4300 |
