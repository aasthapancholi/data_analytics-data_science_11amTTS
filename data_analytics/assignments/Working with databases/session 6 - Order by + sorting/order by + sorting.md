# Module 2: Working with Databases - Session 6: ORDER BY & Sorting

---

## 1. Write an SQL query to display all products from a 'products' table and sort them by price in ascending order, similar to how Flipkart lists items from lowest to highest price.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Concept:** The `ORDER BY` clause sorts the result set. Specifying `ASC` sorts numerical values from smallest to largest (lowest price to highest price). While `ASC` is the default behavior in SQL, writing it explicitly makes the query's intent clear.

#### Optional Setup Script:
```sql
-- Create sample products table
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

-- Insert sample records
INSERT INTO products (product_name, category, price) VALUES
('Boat Rockerz 450', 'Electronics', 1499.00),
('USB-C Fast Cable', 'Accessories', 299.00),
('Dell Wireless Mouse', 'Electronics', 699.00),
('Noise Smartwatch 2', 'Wearables', 1999.00),
('Samsung 43" 4K Smart TV', 'Appliances', 28999.00),
('Cotton T-Shirt', 'Clothing', 499.00),
('HP 15s Laptop', 'Electronics', 44999.00);
```

#### SQL Query:
```sql
SELECT * 
FROM products 
ORDER BY price ASC;
```

#### Output (Sorted Lowest to Highest Price):
| id | product_name | category | price |
|:---|:---|:---|:---|
| 2 | USB-C Fast Cable | Accessories | 299.00 |
| 6 | Cotton T-Shirt | Clothing | 499.00 |
| 3 | Dell Wireless Mouse | Electronics | 699.00 |
| 1 | Boat Rockerz 450 | Electronics | 1499.00 |
| 4 | Noise Smartwatch 2 | Wearables | 1999.00 |
| 5 | Samsung 43" 4K Smart TV | Appliances | 28999.00 |
| 7 | HP 15s Laptop | Electronics | 44999.00 |

---

## 2. Modify your previous query to show the top 5 most expensive products using ORDER BY with DESC and LIMIT.

### SOLUTION:
* **Concept:** To find the most expensive products:
  1. Use `ORDER BY price DESC` to sort prices from highest to lowest.
  2. Append `LIMIT 5` to restrict the returned records to the top 5 entries.

#### SQL Query:
```sql
SELECT * 
FROM products 
ORDER BY price DESC 
LIMIT 5;
```

#### Output (Top 5 Most Expensive Products):
| id | product_name | category | price |
|:---|:---|:---|:---|
| 7 | HP 15s Laptop | Electronics | 44999.00 |
| 5 | Samsung 43" 4K Smart TV | Appliances | 28999.00 |
| 4 | Noise Smartwatch 2 | Wearables | 1999.00 |
| 1 | Boat Rockerz 450 | Electronics | 1499.00 |
| 3 | Dell Wireless Mouse | Electronics | 699.00 |

---

## 3. Given a 'movies' table with columns 'title', 'release_year', and 'rating', write an SQL query to list all movies sorted first by release_year in descending order (latest first), then by rating in descending order (highest rated first).

### SOLUTION:
* **Concept:** Multi-column sorting allows primary and secondary ordering:
  * Primary sort: `release_year DESC` orders movies from newest to oldest.
  * Secondary sort (Tie-breaker): If two or more movies share the same release year, `rating DESC` ensures the higher-rated movie appears first within that year group.

#### Optional Setup Script:
```sql
-- Create sample movies table
CREATE TABLE movies (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    release_year INT NOT NULL,
    rating DECIMAL(3, 1) NOT NULL
);

-- Insert sample records
INSERT INTO movies (title, release_year, rating) VALUES
('Oppenheimer', 2023, 8.9),
('Barbie', 2023, 7.3),
('Dune: Part Two', 2024, 8.6),
('Deadpool & Wolverine', 2024, 8.0),
('Interstellar', 2014, 8.7),
('Whiplash', 2014, 8.5);
```

#### SQL Query:
```sql
SELECT title, release_year, rating 
FROM movies 
ORDER BY release_year DESC, rating DESC;
```

#### Output:
| title | release_year | rating |
|:---|:---|:---|
| Dune: Part Two | 2024 | 8.6 |
| Deadpool & Wolverine | 2024 | 8.0 |
| Oppenheimer | 2023 | 8.9 |
| Barbie | 2023 | 7.3 |
| Interstellar | 2014 | 8.7 |
| Whiplash | 2014 | 8.5 |

*(Notice how in 2024, `Dune: Part Two` (8.6) ranks above `Deadpool & Wolverine` (8.0), and in 2023, `Oppenheimer` (8.9) ranks above `Barbie` (7.3)).*

---

## 4. Write an SQL query to display the first 10 restaurants from a 'restaurants' table, sorted alphabetically by name, just like Zomato's A-Z listing.
*(Hint: Use ORDER BY with LIMIT.)*

### SOLUTION:
* **Concept:** 
  * Applying `ORDER BY name ASC` on textual data sorts strings alphabetically from 'A' to 'Z'.
  * `LIMIT 10` limits the output to the first 10 rows in alphabetical order.

#### Optional Setup Script:
```sql
-- Create sample restaurants table
CREATE TABLE restaurants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    city VARCHAR(50) NOT NULL
);

INSERT INTO restaurants (name, cuisine, city) VALUES
('Barbeque Nation', 'Buffet', 'Ahmedabad'),
('Domino''s Pizza', 'Fast Food', 'Surat'),
('Gwalia Sweets', 'Desserts', 'Ahmedabad'),
('Haldiram''s', 'North Indian', 'Surat'),
('KFC', 'Fast Food', 'Vadodara'),
('Little Italy', 'Italian', 'Ahmedabad'),
('McDonald''s', 'Fast Food', 'Surat'),
('Pizza Hut', 'Fast Food', 'Ahmedabad'),
('Subway', 'Healthy', 'Surat'),
('Swati Snacks', 'Gujarati', 'Ahmedabad'),
('The Belgian Waffle Co.', 'Desserts', 'Vadodara'),
('Under The Fig Tree', 'Cafe', 'Ahmedabad');
```

#### SQL Query:
```sql
SELECT * 
FROM restaurants 
ORDER BY name ASC 
LIMIT 10;
```

#### Output (First 10 Restaurants in Alphabetical Order A-Z):
| id | name | cuisine | city |
|:---|:---|:---|:---|
| 1 | Barbeque Nation | Buffet | Ahmedabad |
| 2 | Domino's Pizza | Fast Food | Surat |
| 3 | Gwalia Sweets | Desserts | Ahmedabad |
| 4 | Haldiram's | North Indian | Surat |
| 5 | KFC | Fast Food | Vadodara |
| 6 | Little Italy | Italian | Ahmedabad |
| 7 | McDonald's | Fast Food | Surat |
| 8 | Pizza Hut | Fast Food | Ahmedabad |
| 9 | Subway | Healthy | Surat |
| 10 | Swati Snacks | Gujarati | Ahmedabad |

---

## 5. Suppose you want to display the top 3 trending songs from a 'songs' table based on play_count, but if two songs have the same play_count, the more recently added song should come first. Write the SQL query to achieve this.
*(Hint: Use ORDER BY with multiple columns.)*

### SOLUTION:
* **Concept:**
  * Primary metric: `play_count DESC` prioritizes the most streamed / trending songs.
  * Tie-breaking metric: `added_date DESC` ensures that if two songs have an identical number of plays, the song added more recently appears first.
  * `LIMIT 3`: Caps the returned list to the top 3 trending tracks.

#### Optional Setup Script:
```sql
-- Create sample songs table
CREATE TABLE songs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    artist VARCHAR(100) NOT NULL,
    play_count INT NOT NULL,
    added_date DATE NOT NULL
);

-- Insert sample records with a tie scenario in play_count
INSERT INTO songs (title, artist, play_count, added_date) VALUES
('Espresso', 'Sabrina Carpenter', 1500000, '2026-04-12'),
('Birds of a Feather', 'Billie Eilish', 1500000, '2026-05-18'), -- Same plays, added later
('Die With A Smile', 'Lady Gaga & Bruno Mars', 1800000, '2026-08-16'),
('Taste', 'Sabrina Carpenter', 950000, '2026-08-23'),
('Good Luck, Babe!', 'Chappell Roan', 1200000, '2026-04-05');
```

#### SQL Query:
```sql
SELECT title, artist, play_count, added_date 
FROM songs 
ORDER BY play_count DESC, added_date DESC 
LIMIT 3;
```

#### Output (Top 3 Trending Songs with Tie-Breaker):
| title | artist | play_count | added_date |
|:---|:---|:---|:---|
| Die With A Smile | Lady Gaga & Bruno Mars | 1800000 | 2026-08-16 |
| Birds of a Feather | Billie Eilish | 1500000 | 2026-05-18 |
| Espresso | Sabrina Carpenter | 1500000 | 2026-04-12 |

*(Notice how `Birds of a Feather` and `Espresso` both have 1,500,000 plays, but `Birds of a Feather` ranks higher because its `added_date` is newer (`2026-05-18` vs `2026-04-12`)).*
