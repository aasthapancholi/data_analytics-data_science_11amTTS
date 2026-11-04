# Module 2: Working with Databases - Session 9: SQL JOINs (Part 1 - INNER, LEFT, RIGHT JOIN)

---

## 1. Create two tables in your database: 'restaurants' (`id`, `name`, `city`) and 'dishes' (`id`, `restaurant_id`, `dish_name`, `price`). Insert at least 3 restaurants and 2-3 dishes for each restaurant.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `restaurants`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique restaurant identifier)
    * `name`: `VARCHAR(100) NOT NULL` (Restaurant name)
    * `city`: `VARCHAR(50) NOT NULL` (Location city)
  * `dishes`:
    * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique dish item identifier)
    * `restaurant_id`: `INT NULL` (Foreign key pointing to `restaurants.id`)
    * `dish_name`: `VARCHAR(100) NOT NULL` (Item title)
    * `price`: `DECIMAL(10, 2) NOT NULL` (Price in ₹)

> **Design Tip for Demonstrating JOINs:**
> * We insert 3 restaurants with dishes, plus a 4th restaurant (*'Chai Point'*) with **no dishes** to demonstrate `LEFT JOIN` (Question 3).
> * We also insert an 8th dish (*'Special Mystery Platter'*) linked to a non-existent `restaurant_id = 99` to simulate a data error for `RIGHT JOIN` (Question 4).

#### SQL Script:
```sql
-- 1. Create parent table: restaurants
CREATE TABLE restaurants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL
);

-- 2. Create child table: dishes
CREATE TABLE dishes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    restaurant_id INT NULL,
    dish_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

-- 3. Insert restaurants
INSERT INTO restaurants (name, city) VALUES
('Swati Snacks', 'Ahmedabad'),
('Little Italy', 'Surat'),
('Sankalp', 'Ahmedabad'),
('Chai Point', 'Vadodara'); -- Restaurant with no dishes on menu

-- 4. Insert dishes
INSERT INTO dishes (restaurant_id, dish_name, price) VALUES
-- Swati Snacks (id: 1)
(1, 'Panki', 180.00),
(1, 'Handvo', 220.00),
(1, 'Fafda Jalebi', 150.00),
-- Little Italy (id: 2)
(2, 'Pasta Alfredo', 450.00),
(2, 'Margherita Pizza', 399.00),
-- Sankalp (id: 3)
(3, 'Masala Dosa', 210.00),
(3, 'Idli Vada Combo', 160.00),
-- Orphaned dish (Simulated data error: restaurant_id 99 does not exist in restaurants)
(99, 'Special Mystery Platter', 299.00);

-- Verify records
SELECT * FROM restaurants;
SELECT * FROM dishes;
```

#### Output (Initial State):
**`restaurants` Table:**
| id | name | city |
|:---|:---|:---|
| 1 | Swati Snacks | Ahmedabad |
| 2 | Little Italy | Surat |
| 3 | Sankalp | Ahmedabad |
| 4 | Chai Point | Vadodara |

**`dishes` Table:**
| id | restaurant_id | dish_name | price |
|:---|:---|:---|:---|
| 1 | 1 | Panki | 180.00 |
| 2 | 1 | Handvo | 220.00 |
| 3 | 1 | Fafda Jalebi | 150.00 |
| 4 | 2 | Pasta Alfredo | 450.00 |
| 5 | 2 | Margherita Pizza | 399.00 |
| 6 | 3 | Masala Dosa | 210.00 |
| 7 | 3 | Idli Vada Combo | 160.00 |
| 8 | 99 | Special Mystery Platter | 299.00 |

---

## 2. Write an SQL INNER JOIN query to display each dish along with its restaurant name and city, similar to how Zomato shows dish details with the restaurant info.

### SOLUTION:
* **Concept:** `INNER JOIN` returns only those rows where there is a match in **both** tables (`dishes.restaurant_id = restaurants.id`).
* Restaurants without dishes (*Chai Point*) and dishes without matching restaurants (*Special Mystery Platter*) are excluded.

#### SQL Query:
```sql
SELECT 
    d.id AS dish_id,
    d.dish_name,
    d.price,
    r.name AS restaurant_name,
    r.city
FROM dishes d
INNER JOIN restaurants r 
    ON d.restaurant_id = r.id;
```

#### Output:
| dish_id | dish_name | price | restaurant_name | city |
|:---|:---|:---|:---|:---|
| 1 | Panki | 180.00 | Swati Snacks | Ahmedabad |
| 2 | Handvo | 220.00 | Swati Snacks | Ahmedabad |
| 3 | Fafda Jalebi | 150.00 | Swati Snacks | Ahmedabad |
| 4 | Pasta Alfredo | 450.00 | Little Italy | Surat |
| 5 | Margherita Pizza | 399.00 | Little Italy | Surat |
| 6 | Masala Dosa | 210.00 | Sankalp | Ahmedabad |
| 7 | Idli Vada Combo | 160.00 | Sankalp | Ahmedabad |

---

## 3. Write an SQL LEFT JOIN query to list all restaurants and their dishes, showing restaurants even if they currently have no dishes on the menu.
*(Hint: Use LEFT JOIN so restaurants without dishes still appear in the results with NULL for dish columns.)*

### SOLUTION:
* **Concept:** `LEFT JOIN` returns all records from the left table (`restaurants`), along with matched records from the right table (`dishes`). If a restaurant has no dishes, the dish columns will contain `NULL`.

#### SQL Query:
```sql
SELECT 
    r.id AS restaurant_id,
    r.name AS restaurant_name,
    r.city,
    d.dish_name,
    d.price
FROM restaurants r
LEFT JOIN dishes d 
    ON r.id = d.restaurant_id;
```

#### Output:
| restaurant_id | restaurant_name | city | dish_name | price |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Ahmedabad | Panki | 180.00 |
| 1 | Swati Snacks | Ahmedabad | Handvo | 220.00 |
| 1 | Swati Snacks | Ahmedabad | Fafda Jalebi | 150.00 |
| 2 | Little Italy | Surat | Pasta Alfredo | 450.00 |
| 2 | Little Italy | Surat | Margherita Pizza | 399.00 |
| 3 | Sankalp | Ahmedabad | Masala Dosa | 210.00 |
| 3 | Sankalp | Ahmedabad | Idli Vada Combo | 160.00 |
| 4 | Chai Point | Vadodara | *NULL* | *NULL* |

*(Notice how `Chai Point` appears in the listing with `NULL` for `dish_name` and `price`).*

---

## 4. Write an SQL RIGHT JOIN query to display all dishes and their restaurant names, including any dishes that might not be linked to a restaurant (simulate a data error where a dish has a restaurant_id that doesn't match any restaurant).

### SOLUTION:
* **Concept:** `RIGHT JOIN` returns all records from the right table (`dishes`), plus matched records from the left table (`restaurants`). If a dish has an unlinked `restaurant_id` (like `99`), the restaurant columns will return `NULL`.

#### SQL Query:
```sql
SELECT 
    d.id AS dish_id,
    d.dish_name,
    d.price,
    d.restaurant_id,
    r.name AS restaurant_name,
    r.city
FROM restaurants r
RIGHT JOIN dishes d 
    ON r.id = d.restaurant_id;
```

#### Output:
| dish_id | dish_name | price | restaurant_id | restaurant_name | city |
|:---|:---|:---|:---|:---|:---|
| 1 | Panki | 180.00 | 1 | Swati Snacks | Ahmedabad |
| 2 | Handvo | 220.00 | 1 | Swati Snacks | Ahmedabad |
| 3 | Fafda Jalebi | 150.00 | 1 | Swati Snacks | Ahmedabad |
| 4 | Pasta Alfredo | 450.00 | 2 | Little Italy | Surat |
| 5 | Margherita Pizza | 399.00 | 2 | Little Italy | Surat |
| 6 | Masala Dosa | 210.00 | 3 | Sankalp | Ahmedabad |
| 7 | Idli Vada Combo | 160.00 | 3 | Sankalp | Ahmedabad |
| 8 | Special Mystery Platter | 299.00 | 99 | *NULL* | *NULL* |

*(Notice how `Special Mystery Platter` appears despite having an invalid `restaurant_id` of 99, with `restaurant_name` and `city` displaying `NULL`).*

---

## 5. Given this scenario: You want to show a list of all playlists and the songs inside them, like Spotify. Explain which JOIN type (INNER, LEFT, or RIGHT) you would use to show all playlists, even if some are empty, and write the SQL query for it.

### SOLUTION:

### JOIN Type Recommendation: **`LEFT JOIN`**

#### Why `LEFT JOIN` is the correct choice:
1. **Preserves Empty Playlists:** When a user creates a new Spotify playlist, it initially has 0 songs. If an `INNER JOIN` were used, empty playlists would be completely eliminated from the output, disappearing from the user's library view.
2. **Left-to-Right Hierarchy:** In relational design, the master entity (`playlists`) sits on the left side of the query. A `LEFT JOIN` guarantees that **every playlist** from the left table is returned. For empty playlists, song columns simply evaluate to `NULL`.
3. **Why Not RIGHT JOIN?** While a `RIGHT JOIN` could achieve this by swapping table order (`FROM songs RIGHT JOIN playlists`), standard SQL conventions favor `LEFT JOIN` for natural left-to-right reading order (Primary Entity &rarr; Dependent Entity).

---

### SQL Queries:

#### **Model A: Simplified Direct Relation (`playlists` &rarr; `songs`)**
```sql
SELECT 
    p.playlist_id,
    p.playlist_name,
    s.song_id,
    s.song_name,
    s.artist,
    s.duration
FROM playlists p
LEFT JOIN songs s 
    ON p.playlist_id = s.playlist_id
ORDER BY p.playlist_id ASC;
```

#### **Model B: Normalized Many-to-Many Architecture (`playlists` &harr; `playlist_songs` &harr; `songs`)**
In production apps like Spotify, songs can belong to multiple playlists, connected via a junction table `playlist_songs`:

```sql
SELECT 
    p.playlist_id,
    p.playlist_name,
    s.song_name,
    s.artist
FROM playlists p
LEFT JOIN playlist_songs ps 
    ON p.playlist_id = ps.playlist_id
LEFT JOIN songs s 
    ON ps.song_id = s.song_id
ORDER BY p.playlist_id ASC;
```

#### Expected Output Example:
| playlist_id | playlist_name | song_name | artist |
|:---|:---|:---|:---|
| 1 | Workout Hits | Blinding Lights | The Weeknd |
| 1 | Workout Hits | Levitating | Dua Lipa |
| 2 | Chill Vibes | Kesariya | Arijit Singh |
| 3 | My New Playlist (Empty) | *NULL* | *NULL* |
