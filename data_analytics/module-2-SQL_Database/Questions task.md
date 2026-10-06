# SQL DDL Questions and Answers

## 1. What is the difference between `CREATE`, `ALTER`, `DROP`, and `TRUNCATE` in SQL?

- `CREATE` creates a new database object, such as a table.
- `ALTER` changes the structure of an existing database object.
- `DROP` removes a database object, including its structure and data.
- `TRUNCATE` removes all rows from a table while keeping the table structure. Whether it resets identity or auto-increment values depends on the database.

## 2. How do you create a table named `employees` with columns `employee_id`, `name`, `salary`, and `department_id`?

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    salary DECIMAL(10, 2),
    department_id INT
);
```

## 3. How do you add a new column `email` to an existing `employees` table?

```sql
ALTER TABLE employees
ADD COLUMN email VARCHAR(255);
```

## 4. How do you modify the data type of the `salary` column?

The syntax varies by database. For example, in PostgreSQL:

```sql
ALTER TABLE employees
ALTER COLUMN salary TYPE DECIMAL(12, 2);
```

In MySQL:

```sql
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12, 2);
```

## 5. How do you rename a column in an existing table?

For example, to rename `name` to `employee_name`:

```sql
ALTER TABLE employees
RENAME COLUMN name TO employee_name;
```

## 6. How do you rename the `employees` table to `staff`?

```sql
ALTER TABLE employees
RENAME TO staff;
```

## 7. How do you remove the `email` column from the `employees` table?

```sql
ALTER TABLE employees
DROP COLUMN email;
```

## 8. How do you create a table with a primary key?

Declare the primary key in the table definition:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2),
    department_id INT
);
```

## 9. How do you create a table with a `NOT NULL` constraint?

Add `NOT NULL` to a column that must always have a value:

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10, 2),
    department_id INT
);
```

## 10. How do you create a table with a `UNIQUE` constraint?

Add `UNIQUE` to a column whose values must not be duplicated:

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);
```

Constraints can also be combined in one table definition:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10, 2),
    department_id INT,
    email VARCHAR(255) UNIQUE
);
```
