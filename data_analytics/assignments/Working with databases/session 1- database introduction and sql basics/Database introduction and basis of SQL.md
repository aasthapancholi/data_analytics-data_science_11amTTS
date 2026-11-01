# Module 2: Working with Databases - Session 1: Database Introduction and SQL Basics

---

## 1. Install MySQL or PostgreSQL on your system and create a new database named 'music_streaming_app' using the command line or GUI tool of your choice.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL Service)
* In XAMPP, open the **XAMPP Control Panel** and start the **MySQL** and **Apache** services.
* Open your browser and navigate to `http://localhost/phpmyadmin/` (or click **Admin** next to MySQL in XAMPP Control Panel).
* Go to the **SQL** tab or click on **Databases** to create the new database.

#### SQL Command:
```sql
CREATE DATABASE music_streaming_app;
USE music_streaming_app;
```

#### Verification:
The database `music_streaming_app` is successfully created and selected as the active database.

![Database Creation](Screenshot%202026-09-26%20115334.png)

---

## 2. Inside the 'music_streaming_app' database, create a table called 'playlists' with columns: playlist_id (integer, primary key), name (varchar), and created_by (varchar).

### SOLUTION:
To store playlist details, we define the `playlists` table with three columns:
- `playlist_id`: Integer, defined as the `PRIMARY KEY` to uniquely identify each playlist.
- `name`: `VARCHAR(100)` to store the title of the playlist.
- `created_by`: `VARCHAR(50)` to store the name of the user who created it.

#### SQL Command:
```sql
CREATE TABLE playlists (
    playlist_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_by VARCHAR(50) NOT NULL
);
```

#### Table Structure:
| Column Name   | Data Type     | Key         | Null Allowed | Description                     |
|:--------------|:--------------|:------------|:-------------|:--------------------------------|
| `playlist_id` | INT           | PRIMARY KEY | No           | Unique identifier for playlist  |
| `name`        | VARCHAR(100)  |             | No           | Name/title of the playlist      |
| `created_by`  | VARCHAR(50)   |             | No           | Username of playlist creator    |

---

## 3. Insert three sample rows into the 'playlists' table representing playlists like 'Bollywood Hits', 'Chill Vibes', and 'Workout Mix', each created by a different user.

### SOLUTION:
We insert 3 records into the `playlists` table with unique playlist IDs and distinct creator names:

#### SQL Command:
```sql
INSERT INTO playlists (playlist_id, name, created_by) VALUES
(1, 'Bollywood Hits', 'Amit'),
(2, 'Chill Vibes', 'Pooja'),
(3, 'Workout Mix', 'Rahul');
```

#### Data in `playlists` Table:
| playlist_id | name           | created_by |
|:------------|:---------------|:-----------|
| 1           | Bollywood Hits | Amit       |
| 2           | Chill Vibes    | Pooja      |
| 3           | Workout Mix    | Rahul      |

---

## 4. Write an SQL SELECT query to display all playlists created by the user 'Amit' from the 'playlists' table.
*(Hint: Use the WHERE clause to filter by the 'created_by' column.)*

### SOLUTION:
To retrieve only the playlists made by 'Amit', we use the `SELECT` statement filtered by the `WHERE` condition on `created_by`.

#### SQL Command:
```sql
SELECT playlist_id, name, created_by
FROM playlists
WHERE created_by = 'Amit';
```

#### Query Result:
| playlist_id | name           | created_by |
|:------------|:---------------|:-----------|
| 1           | Bollywood Hits | Amit       |

---

## 5. Open ChatGPT or Copilot and ask it to explain the difference between a table, a row, and a column in SQL using an example from a food delivery app like Zomato. Paste the explanation you receive into your assignment.

### SOLUTION:

In SQL and Relational Database Management Systems (RDBMS), data is organized systematically into structures. Here is the difference between a **Table**, a **Row**, and a **Column** explained using a food delivery app like **Zomato**:

---

### 1. Table (The Container / Collection)
* **What it is:** A table is a structured collection of related data stored together in rows and columns, similar to a complete spreadsheet or ledger.
* **Zomato Example:** Imagine an **`orders`** table in Zomato. This single table holds the complete record of every food order placed by all customers across the platform.

---

### 2. Column (The Attribute / Category)
* **What it is:** A column is a vertical set of data values representing a specific property, category, or characteristic that every record will possess.
* **Zomato Example:** In the `orders` table, the individual columns define what details are captured for each order:
  * `order_id` (Unique identifier for the order)
  * `customer_name` (Name of the person who ordered)
  * `restaurant_name` (Restaurant preparing the food)
  * `item_name` (Dish ordered)
  * `total_amount` (Final bill amount)

---

### 3. Row (The Record / Individual Entry)
* **What it is:** A row (also known as a record or tuple) is a single horizontal entry that contains all the actual values for one specific entity or transaction.
* **Zomato Example:** When customer *Amit* orders a *Paneer Butter Masala* from *Haldiram's*, that entire transaction forms **one single row**:
  * `[101, 'Amit', 'Haldiram\'s', 'Paneer Butter Masala', 350.00]`

---

### Visual Representation of the Zomato `orders` Table:

| order_id *(Column)* | customer_name *(Column)* | restaurant_name *(Column)* | item_name *(Column)*   | total_amount *(Column)* |
|:--------------------|:-------------------------|:---------------------------|:-----------------------|:------------------------|
| **101**             | **Amit**                 | **Haldiram's**             | **Paneer Butter Masala** | **₹350**                | *(<- Row 1: Amit's Order)* |
| **102**             | **Pooja**                | **Domino's Pizza**         | **Farmhouse Pizza**    | **₹499**                | *(<- Row 2: Pooja's Order)* |
| **103**             | **Rahul**                | **Subway**                 | **Veggie Delite Sub**  | **₹240**                | *(<- Row 3: Rahul's Order)* |

### Summary Comparison:
| Concept | Database Role | Zomato Analogy |
|:---|:---|:---|
| **Table** | The entire dataset of a specific entity | The entire **Order History Log** |
| **Column** | The specific feature/attribute being measured | **Order ID**, **Restaurant**, or **Price** |
| **Row** | One single complete instance/event | **One customer's specific food order** |
