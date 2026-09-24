# What is DBMS and RDBMS?

## DBMS

A Database Management System (DBMS) is software that is used to create, store, update, and manage data in a database.

It acts as a bridge between the user and the database.

### Functions of DBMS

- Create database and tables
- Insert and store data
- Retrieve data efficiently
- Update and delete records
- Maintain data security
- Control user access
- Reduce data redundancy
- Maintain data integrity

### Examples of DBMS

- MySQL
- Oracle
- SQL Server
- PostgreSQL
- SQLite

## RDBMS

A Relational Database Management System (RDBMS) is a type of DBMS that stores data in tables with rows and columns.

It follows the relational model, where data is organized into related tables.

### Key features of RDBMS

- Data stored in tables
- Relationships between tables
- Primary keys and foreign keys
- Data integrity
- Structured query language (SQL)
- Easy data retrieval and manipulation

### Example

A school database may have:

- Students table
- Courses table
- Marks table

These tables are connected using keys.

## Difference between DBMS and RDBMS

| Feature | DBMS | RDBMS |
|---------|------|-------|
| Data model | Can be non-relational | Relational |
| Structure | May not use tables | Uses tables |
| Data relationships | Limited | Strong relationships |
| SQL support | Not always | Yes |
| Normalization | Not required | Commonly used |
| Examples | File-based systems, some legacy DBMS | MySQL, Oracle, SQL Server |

## Important points

- All RDBMS are DBMS, but not all DBMS are RDBMS.
- RDBMS is more structured and suitable for business applications.
- SQL is used to interact with RDBMS.

## Real-life example

Suppose a company stores data about employees and departments.

- Employee table stores employee details
- Department table stores department details
- The department ID connects both tables

This relationship is managed efficiently by an RDBMS.

## Summary

- DBMS = software for managing databases
- RDBMS = DBMS that stores data in relational tables
- RDBMS is widely used for structured business data
- SQL is the language used to work with RDBMS

## Keywords

- DBMS
- RDBMS
- Table
- Row
- Column
- Primary Key
- Foreign Key
- SQL

---

This note explains the difference between DBMS and RDBMS and why relational databases are important in data management.
