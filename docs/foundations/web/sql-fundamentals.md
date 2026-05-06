# SQL Fundamentals

Databases store and organize application data. Web applications commonly use databases for users, sessions, products, posts, permissions, logs, and configuration.

Understanding databases and SQL is important for web security because many critical vulnerabilities involve unsafe database queries, especially SQL injection.

---

## What Is a Database?

A database is a structured collection of data designed for storage, retrieval, and management.

Examples of data stored in web application databases:

- User accounts
- Password hashes
- Blog posts
- Orders
- Messages
- API tokens
- Permissions
- Audit logs

---

## Relational vs. Non-Relational Databases

Databases are commonly grouped into relational and non-relational systems.

| Type | Description | Examples |
|---|---|---|
| Relational / SQL | Stores data in tables with rows and columns | MySQL, PostgreSQL, MSSQL, SQLite |
| Non-relational / NoSQL | Stores data in flexible formats like documents or key-value pairs | MongoDB, Redis, Cassandra |

Relational databases are common in traditional web applications, authentication systems, and business applications.

Non-relational databases are common in flexible, high-scale, or document-heavy systems.

---

## Tables, Rows, and Columns

A relational database organizes data into tables.

Example `users` table:

| id | username | email | role |
|---:|---|---|---|
| 1 | alice | alice@example.com | user |
| 2 | bob | bob@example.com | admin |

| Concept | Meaning |
|---|---|
| Table | Collection of related data |
| Column | Attribute or field |
| Row | One record |
| Cell | One value in a row |

Example:

```text
Table: users
Column: username
Row: alice's user record
```

---

## Data Types

Columns have data types.

Common SQL data types:

| Type | Purpose |
|---|---|
| `INT` | Whole numbers |
| `VARCHAR` | Variable-length text |
| `TEXT` | Longer text |
| `BOOLEAN` | True/false values |
| `DATE` | Date value |
| `TIMESTAMP` | Date and time |
| `DECIMAL` | Precise numeric values |

Example:

```sql
CREATE TABLE users (
  id INT,
  username VARCHAR(50),
  email VARCHAR(255),
  active BOOLEAN
);
```

---

## Primary Keys

A primary key uniquely identifies each row in a table.

Example:

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  username VARCHAR(50),
  email VARCHAR(255)
);
```

In this table, `id` uniquely identifies each user.

Primary keys should be:

- Unique
- Stable
- Not null
- Used to reference specific records

---

## Foreign Keys

A foreign key links one table to another.

Example:

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY,
  user_id INT,
  total DECIMAL(10, 2),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

In this example:

```text
orders.user_id -> users.id
```

This means each order belongs to a user.

---

## Basic SQL Commands

SQL is used to create, read, update, and delete data.

| Action | SQL Command |
|---|---|
| Create data | `INSERT` |
| Read data | `SELECT` |
| Update data | `UPDATE` |
| Delete data | `DELETE` |
| Create structure | `CREATE` |
| Modify structure | `ALTER` |
| Remove structure | `DROP` |

---

## SELECT

`SELECT` reads data from a table.

```sql
SELECT username, email
FROM users;
```

Select all columns:

```sql
SELECT *
FROM users;
```

Filter results:

```sql
SELECT *
FROM users
WHERE role = 'admin';
```

---

## INSERT

`INSERT` adds new data.

```sql
INSERT INTO users (id, username, email, role)
VALUES (1, 'alice', 'alice@example.com', 'user');
```

---

## UPDATE

`UPDATE` modifies existing data.

```sql
UPDATE users
SET role = 'admin'
WHERE username = 'alice';
```

!!! warning
    Be careful with `UPDATE` statements. Without a `WHERE` clause, all rows may be modified.

---

## DELETE

`DELETE` removes data.

```sql
DELETE FROM users
WHERE id = 1;
```

!!! warning
    Be careful with `DELETE` statements. Without a `WHERE` clause, all rows may be deleted.

---

## WHERE Clauses

`WHERE` filters query results.

```sql
SELECT *
FROM users
WHERE username = 'alice';
```

Common operators:

| Operator | Meaning |
|---|---|
| `=` | Equal |
| `!=` | Not equal |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |
| `LIKE` | Pattern match |
| `IN` | Match a list |
| `AND` | Both conditions true |
| `OR` | Either condition true |

Example:

```sql
SELECT *
FROM users
WHERE role = 'admin' AND active = true;
```

---

## LIKE

`LIKE` searches for patterns.

```sql
SELECT *
FROM users
WHERE email LIKE '%@example.com';
```

Pattern symbols:

| Symbol | Meaning |
|---|---|
| `%` | Any number of characters |
| `_` | One character |

---

## ORDER BY

`ORDER BY` sorts results.

```sql
SELECT *
FROM users
ORDER BY username ASC;
```

Descending order:

```sql
SELECT *
FROM users
ORDER BY created_at DESC;
```

---

## LIMIT

`LIMIT` restricts how many rows are returned.

```sql
SELECT *
FROM users
LIMIT 10;
```

Useful for:

- Pagination
- Testing queries
- Preventing huge result sets

---

## JOIN

`JOIN` combines data from multiple tables.

Example tables:

```text
users
- id
- username

orders
- id
- user_id
- total
```

Query:

```sql
SELECT users.username, orders.total
FROM users
JOIN orders ON orders.user_id = users.id;
```

This returns order data with the username attached.

---

## Common Database Systems

| Database | Type | Notes |
|---|---|---|
| MySQL | Relational | Common in PHP/Linux stacks |
| PostgreSQL | Relational | Powerful open-source database |
| MSSQL | Relational | Common in Microsoft environments |
| SQLite | Relational | File-based, lightweight |
| MongoDB | NoSQL | Document-based |
| Redis | NoSQL | Key-value store, often in memory |

---

## NoSQL Example

NoSQL databases may store records as JSON-like documents.

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "username": "alice",
  "email": "alice@example.com",
  "roles": ["user", "editor"]
}
```

NoSQL can be useful when data structure is flexible, but it still requires secure query handling.

---

## SQL Injection

SQL injection happens when user input is inserted into a database query unsafely.

Vulnerable example:

```sql
SELECT *
FROM users
WHERE username = '$username'
AND password = '$password';
```

If input is concatenated directly into the query, an attacker may alter the query logic.

Example malicious input:

```text
' OR '1'='1
```

Resulting query:

```sql
SELECT *
FROM users
WHERE username = '' OR '1'='1'
AND password = '';
```

Security impact may include:

- Authentication bypass
- Data extraction
- Data modification
- Data deletion
- Privilege escalation
- Remote code execution in some cases

---

## Preventing SQL Injection

The main defense is parameterized queries, also called prepared statements.

Instead of building a query by concatenating strings, the query structure and user values are separated.

Conceptual example:

```sql
SELECT *
FROM users
WHERE username = ?
AND password = ?;
```

Other defenses:

- Input validation
- Least-privilege database accounts
- Safe error handling
- Avoiding dynamic SQL when possible
- Web application firewall as defense-in-depth
- Logging and monitoring suspicious queries

!!! important
    Escaping input manually is not a replacement for parameterized queries.

---

## Password Storage in Databases

Applications often store password hashes in databases.

Bad:

```text
password = password123
```

Better:

```text
password_hash = bcrypt/argon2 hash
```

Good password storage uses:

- Strong password hashing algorithm
- Unique salt
- Slow hashing function
- No plaintext passwords
- No reversible encryption for normal password storage

Recommended algorithms:

- Argon2
- bcrypt
- scrypt
- PBKDF2

---

## Database Security Basics

Good database security includes:

- Strong authentication
- Least-privilege users
- Network restrictions
- Encrypted connections
- Backups
- Monitoring
- Secure password storage
- Patch management
- Avoiding public exposure

Example:

```text
Web app database user should only have permissions needed by the app.
It should not be a database administrator account.
```

---

## Useful SQL Examples

Create a table:

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  username VARCHAR(50),
  email VARCHAR(255),
  role VARCHAR(20)
);
```

Insert data:

```sql
INSERT INTO users (id, username, email, role)
VALUES (1, 'alice', 'alice@example.com', 'user');
```

Read data:

```sql
SELECT *
FROM users
WHERE username = 'alice';
```

Update data:

```sql
UPDATE users
SET role = 'admin'
WHERE id = 1;
```

Delete data:

```sql
DELETE FROM users
WHERE id = 1;
```

Join tables:

```sql
SELECT users.username, orders.total
FROM users
JOIN orders ON orders.user_id = users.id;
```

---

## Quick Reference

| Concept | Meaning |
|---|---|
| Database | Organized data storage |
| Table | Collection of related records |
| Column | Attribute or field |
| Row | Single record |
| Primary key | Unique row identifier |
| Foreign key | Link to another table |
| SQL | Query language for relational databases |
| NoSQL | Flexible non-relational database style |
| SELECT | Read data |
| INSERT | Add data |
| UPDATE | Modify data |
| DELETE | Remove data |
| JOIN | Combine tables |
| SQL injection | Unsafe query manipulation through user input |

---

## Notes to Remember

- Relational databases store data in tables.
- Rows are records; columns are fields.
- Primary keys uniquely identify records.
- Foreign keys create relationships between tables.
- SQL is used to query and manage relational data.
- NoSQL stores data in flexible formats like documents or key-value pairs.
- SQL injection happens when user input changes query logic.
- Parameterized queries are the main defense against SQL injection.
- Database accounts should follow least privilege.