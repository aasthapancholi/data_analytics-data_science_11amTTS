# What is SQL?

SQL stands for Structured Query Language.

It is the standard language used to communicate with relational databases.

SQL is used to:

- Create databases and tables
- Insert data
- Update records
- Delete records
- Fetch or search data
- Manage user access
- Generate reports

## Why SQL is important

SQL helps users work with large amounts of structured data easily.

It is one of the most important tools in data analytics, software development, and database management.

## Common SQL commands

### 1. CREATE
Used to create a database or table.

```sql
CREATE TABLE Students (
    StudentID INT,
    Name VARCHAR(50),
    Age INT,
    City VARCHAR(50)
);
```

### 2. INSERT
Used to insert data into a table.

```sql
INSERT INTO Students (StudentID, Name, Age, City)
VALUES (1, 'Aman', 20, 'Delhi');
```

### 3. SELECT
Used to read data from a table.

```sql
SELECT * FROM Students;
```

### 4. UPDATE
Used to change existing data.

```sql
UPDATE Students
SET City = 'Mumbai'
WHERE StudentID = 1;
```

### 5. DELETE
Used to remove data.

```sql
DELETE FROM Students
WHERE StudentID = 1;
```

### 6. DROP
Used to delete a table or database.

```sql
DROP TABLE Students;
```

## SQL examples in simple form

### Print all student names

```sql
SELECT Name FROM Students;
```

### Find students from Delhi

```sql
SELECT * FROM Students
WHERE City = 'Delhi';
```

### Sort records in ascending order

```sql
SELECT * FROM Students
ORDER BY Age ASC;
```

## SQL in real life

SQL is used in:

- Banking systems
- Online shopping websites
- Student management systems
- Customer relationship management systems
- Inventory management
- Data analysis projects

## SQL vs NoSQL

SQL is used for relational databases, while NoSQL is used for non-relational databases.

### SQL

- Uses tables
- Fixed schema
- Good for structured data
- Best for reporting and transactions

### NoSQL

- Flexible schema
- Good for unstructured or semi-structured data
- Common in big data and real-time applications

## Summary

SQL is the standard language used to work with relational databases.

It allows users to create, read, update, and delete data effectively.

Without SQL, managing data in relational databases would be difficult and time-consuming.

## Keywords

- SQL
- Database
- Table
- Query
- SELECT
- INSERT
- UPDATE
- DELETE
- CREATE
- DROP

---

This note introduces SQL as the language used to work with relational databases and explains some of its basic commands.
