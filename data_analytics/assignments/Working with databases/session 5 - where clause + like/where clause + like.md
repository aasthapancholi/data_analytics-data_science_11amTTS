# Module 2: Working with Databases - Session 5: WHERE Clause & Pattern Matching (LIKE)

---

## 1. Create a table called `Restaurants` with columns: `id`, `name`, `cuisine`, `rating`, and `city`. Insert at least 5 sample records representing real or fictional restaurants you might find on Zomato.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* **Table Design:**
  * `id`: `INT PRIMARY KEY AUTO_INCREMENT` (Unique identifier for each restaurant)
  * `name`: `VARCHAR(100) NOT NULL` (Restaurant name)
  * `cuisine`: `VARCHAR(50) NOT NULL` (Primary food category/style)
  * `rating`: `DECIMAL(2, 1) NOT NULL` (Customer rating score out of 5.0)
  * `city`: `VARCHAR(50) NOT NULL` (City where the outlet is located)

#### SQL Script:
```sql
-- 1. Create the Restaurants table
CREATE TABLE Restaurants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL,
    city VARCHAR(50) NOT NULL
);

-- 2. Insert sample Zomato-style restaurant records
INSERT INTO Restaurants (name, cuisine, rating, city) VALUES
('Swati Snacks', 'Gujarati', 4.5, 'Ahmedabad'),
('Swadisht Restaurant', 'South Indian', 4.2, 'Surat'),
('Little Italy', 'Italian', 4.4, 'Ahmedabad'),
('Mainland China', 'Chinese', 3.9, 'Vadodara'),
('Swagat Dine', 'North Indian', 3.4, 'Surat'),
('Sankalp', 'South Indian', 4.6, 'Ahmedabad');

-- 3. Retrieve all records to verify insertion
SELECT * FROM Restaurants;
```

#### Output (Initial Table State):
| id | name | cuisine | rating | city |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Gujarati | 4.5 | Ahmedabad |
| 2 | Swadisht Restaurant | South Indian | 4.2 | Surat |
| 3 | Little Italy | Italian | 4.4 | Ahmedabad |
| 4 | Mainland China | Chinese | 3.9 | Vadodara |
| 5 | Swagat Dine | North Indian | 3.4 | Surat |
| 6 | Sankalp | South Indian | 4.6 | Ahmedabad |


---

## 2. Write a SQL query to find all restaurants in the `Restaurants` table that have a rating greater than 4.0 and are located in either 'Ahmedabad' or 'Surat'.

### SOLUTION:
To combine multiple conditions:
1. `rating > 4.0` filters restaurants with a high rating.
2. `city IN ('Ahmedabad', 'Surat')` or `(city = 'Ahmedabad' OR city = 'Surat')` ensures the location is in either city.
3. Using `AND` joins both conditions. When combining `AND` with `OR`, parentheses `()` are critical to ensure correct operator precedence.

#### SQL Query (Option A - Using `IN`):
```sql
SELECT * 
FROM Restaurants 
WHERE rating > 4.0 
  AND city IN ('Ahmedabad', 'Surat');
```

#### SQL Query (Option B - Using `OR` with parentheses):
```sql
SELECT * 
FROM Restaurants 
WHERE rating > 4.0 
  AND (city = 'Ahmedabad' OR city = 'Surat');
```

#### Output:
| id | name | cuisine | rating | city |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Gujarati | 4.5 | Ahmedabad |
| 2 | Swadisht Restaurant | South Indian | 4.2 | Surat |
| 3 | Little Italy | Italian | 4.4 | Ahmedabad |
| 6 | Sankalp | South Indian | 4.6 | Ahmedabad |

---

## 3. Using the `LIKE` operator, write a query to select all restaurants whose names start with 'Swa' (for example, 'Swagat', 'Swadisht') from the `Restaurants` table.

### SOLUTION:
The `LIKE` operator is used with wildcard characters:
* `%` matches zero, one, or multiple characters.
* Pattern `'Swa%'` searches for strings that begin with the letters `Swa` followed by any sequence of characters.

#### SQL Query:
```sql
SELECT * 
FROM Restaurants 
WHERE name LIKE 'Swa%';
```

#### Output:
| id | name | cuisine | rating | city |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Gujarati | 4.5 | Ahmedabad |
| 2 | Swadisht Restaurant | South Indian | 4.2 | Surat |
| 5 | Swagat Dine | North Indian | 3.4 | Surat |

---

## 4. Write a SQL query using the `BETWEEN` keyword to find all restaurants in the `Restaurants` table with a rating between 3.5 and 4.5 (inclusive).

### SOLUTION:
The `BETWEEN ... AND ...` operator selects values within a specified range. In SQL, `BETWEEN` is **inclusive**, meaning both the boundary values (`3.5` and `4.5`) are included in the results (equivalent to `rating >= 3.5 AND rating <= 4.5`).

#### SQL Query:
```sql
SELECT * 
FROM Restaurants 
WHERE rating BETWEEN 3.5 AND 4.5;
```

#### Output:
| id | name | cuisine | rating | city |
|:---|:---|:---|:---|:---|
| 1 | Swati Snacks | Gujarati | 4.5 | Ahmedabad |
| 2 | Swadisht Restaurant | South Indian | 4.2 | Surat |
| 3 | Little Italy | Italian | 4.4 | Ahmedabad |
| 4 | Mainland China | Chinese | 3.9 | Vadodara |

*(Note: Restaurant #5 with rating 3.4 and #6 with rating 4.6 are excluded as they fall outside the 3.5 - 4.5 range).*

---

## 5. Write a query to find all restaurants whose cuisine is either 'Chinese', 'Italian', or 'South Indian' using the `IN` operator.

### SOLUTION:
The `IN` operator allows you to specify multiple values in a `WHERE` clause. It acts as a shorthand for multiple `OR` conditions (`cuisine = 'Chinese' OR cuisine = 'Italian' OR cuisine = 'South Indian'`), making queries significantly cleaner and easier to maintain.

#### SQL Query:
```sql
SELECT * 
FROM Restaurants 
WHERE cuisine IN ('Chinese', 'Italian', 'South Indian');
```

#### Output:
| id | name | cuisine | rating | city |
|:---|:---|:---|:---|:---|
| 2 | Swadisht Restaurant | South Indian | 4.2 | Surat |
| 3 | Little Italy | Italian | 4.4 | Ahmedabad |
| 4 | Mainland China | Chinese | 3.9 | Vadodara |
| 6 | Sankalp | South Indian | 4.6 | Ahmedabad |
