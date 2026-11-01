# Module 2: Working with Databases - Session 2: Database Setup & Create Table

---

## 1. Install MySQL Community Server or SQLite on your system and verify the installation by connecting to the database using the command line or a GUI tool like MySQL Workbench or DB Browser for SQLite.

### SOLUTION:
* **Tool Used:** XAMPP (MySQL / phpMyAdmin GUI)
* **Installation & Verification Steps:**
  1. Open the **XAMPP Control Panel**.
  2. Start the **Apache** and **MySQL** services (both module indicators will turn green with active port 3306 for MySQL).
  3. Open your browser and navigate to `http://localhost/phpmyadmin/` to access the graphical database manager.
  4. Alternatively, click the **Shell** button in XAMPP Control Panel and verify the connection via command line by typing:
     ```bash
     mysql -u root -p
     ```
     Press **Enter** (default password in XAMPP is blank). A welcome banner with the MariaDB/MySQL version confirms a successful connection.

*(Optional: Add screenshot of XAMPP Control Panel running MySQL or phpMyAdmin interface)*
<!-- ![XAMPP MySQL Verification](mysql_verification.png) -->

---

## 2. Create a new database named 'foodie_app' to simulate a Zomato-style backend.

### SOLUTION:
In phpMyAdmin (or MySQL CLI), we create the database `foodie_app` and switch context to this new database.

#### SQL Command:
```sql
-- Create database
CREATE DATABASE foodie_app;

-- Select the database for active use
USE foodie_app;
```

#### Verification:
The database `foodie_app` is created and listed under the schemas sidebar in phpMyAdmin.

---

## 3. Write a CREATE TABLE statement to define a 'restaurants' table in the 'foodie_app' database with the following columns: id (integer, primary key), name (varchar/character, max 100), cuisine (varchar/character, max 50), rating (decimal, e.g., 4.5), and location (varchar/character, max 100).

### SOLUTION:
We define the `restaurants` table with appropriate data types and constraints:
- `id`: `INT PRIMARY KEY`
- `name`: `VARCHAR(100)`
- `cuisine`: `VARCHAR(50)`
- `rating`: `DECIMAL(2, 1)` (allows numbers like 4.5, 5.0)
- `location`: `VARCHAR(100)`

#### SQL Command:
```sql
CREATE TABLE restaurants (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    cuisine VARCHAR(50) NOT NULL,
    rating DECIMAL(2, 1) NOT NULL,
    location VARCHAR(100) NOT NULL
);
```

#### Table Schema:
| Column Name | Data Type | Constraint | Description |
|:---|:---|:---|:---|
| `id` | `INT` | `PRIMARY KEY` | Unique ID for the restaurant |
| `name` | `VARCHAR(100)` | `NOT NULL` | Restaurant name (e.g. 'Haldiram\'s') |
| `cuisine` | `VARCHAR(50)` | `NOT NULL` | Type of cuisine (e.g. 'North Indian', 'Chinese') |
| `rating` | `DECIMAL(2, 1)` | `NOT NULL` | Food rating between 0.0 and 5.0 |
| `location` | `VARCHAR(100)` | `NOT NULL` | City / Area branch |

---

## 4. Design and create a 'users' table for a Flipkart-style app with columns: user_id (primary key), username, email, phone_number, and created_at (date/time). Pick appropriate data types for each column.
*(Hint: Think about which columns should be unique and which data types best fit email and phone numbers.)*

### SOLUTION:

#### Design Rationale:
1. **`user_id` (`INT PRIMARY KEY AUTO_INCREMENT`):** Automatically assigns a unique number to each new user.
2. **`username` (`VARCHAR(50) NOT NULL`):** Suitable for user display names.
3. **`email` (`VARCHAR(100) NOT NULL UNIQUE`):** Enforces unique email accounts so two users cannot sign up with the same address.
4. **`phone_number` (`VARCHAR(15) NOT NULL UNIQUE`):**
   * *Why `VARCHAR` instead of `INT` or `BIGINT`?*
     * Phone numbers often begin with leading zeros (e.g., `098...`) which numeric types strip off.
     * Accommodates international formats and symbols like `+91`.
     * Phone numbers are never used for mathematical calculations.
   * `UNIQUE` ensures each phone number is linked to only one account.
5. **`created_at` (`TIMESTAMP DEFAULT CURRENT_TIMESTAMP`):** Automatically captures the exact registration date and time.

#### SQL Command:
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone_number VARCHAR(15) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Table Schema:
| Column Name | Data Type | Key / Constraints | Purpose |
|:---|:---|:---|:---|
| `user_id` | `INT` | `PRIMARY KEY`, `AUTO_INCREMENT` | Unique identifier for each shopper |
| `username` | `VARCHAR(50)` | `NOT NULL` | Customer's full or display name |
| `email` | `VARCHAR(100)` | `NOT NULL`, `UNIQUE` | Login email address |
| `phone_number` | `VARCHAR(15)` | `NOT NULL`, `UNIQUE` | Contact number for OTP & deliveries |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | Account creation timestamp |

---

## 5. Intentionally make a mistake in your CREATE TABLE statement (such as missing a comma or using an unsupported data type), run it, and then fix the error based on the message you receive.
*(Hint: Take a screenshot of the error and the corrected SQL statement for your records.)*

### SOLUTION:

### Step 1: Intentionally Flawed SQL Statement (Missing Comma)
We deliberately omit the comma `,` between `id INT PRIMARY KEY` and `name VARCHAR(50)`:

```sql
CREATE TABLE test_table (
    id INT PRIMARY KEY
    name VARCHAR(50)
);
```

### Step 2: The Error Received
When executing the query, MySQL/phpMyAdmin returns a syntax error:
```text
Error: #1064 - You have an error in your SQL syntax; check the manual that corresponds to your MariaDB/MySQL server version for the right syntax to use near 'name VARCHAR(50)
)' at line 3
```

### Step 3: Error Diagnosis & Explanation
* **Why did this error happen?**
  SQL requires a comma `,` to separate each column definition in the `CREATE TABLE` list. Without the comma after `PRIMARY KEY`, the SQL parser treats `name` as an unexpected token following the `id` definition, triggering error `#1064 (Syntax Error)`.

### Step 4: Corrected SQL Statement
We fix the error by adding the missing comma `,` at the end of line 2:

```sql
CREATE TABLE test_table (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);
```

#### Result:
The query executes successfully:
```text
Query OK, 0 rows affected (0.02 sec)
```

*(Optional: Insert screenshot of the error message and the successful execution)*
<!-- ![Syntax Error and Fix](sql_error_fix.png) -->
