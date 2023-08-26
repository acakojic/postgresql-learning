# Transactions in PostgreSQL: How They Work Without BEGIN and Isolation Levels

## Introduction

In PostgreSQL, a transaction is a unit of work that takes the database from one consistent state to another. Transactions play a crucial role in ensuring data integrity, consistency, and isolation. PostgreSQL has four transaction isolation levels: `SERIALIZABLE`, `REPEATABLE READ`, `READ COMMITTED`, and `READ UNCOMMITTED`. However, in this article, we'll focus on the default behavior of PostgreSQL transactions, which is `READ COMMITTED`.

## Implicit Transactions

Interestingly, you can execute SQL statements in PostgreSQL without explicitly using the `BEGIN` keyword to start a transaction. These are known as "implicit transactions". Each SQL statement runs in its own transaction and commits immediately upon successful completion. Essentially, the database automatically issues a `BEGIN` at the start of the statement and a `COMMIT` at the end.

## Example: Default Insert Statement Without BEGIN

Let's assume you have a table called `students`:

```sql
CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50),
  age INT
);

```

You can insert a new row into the table like this:
```sql
INSERT INTO students (name, age) VALUES ('Alice', 20);

```

In this case, PostgreSQL implicitly starts and commits a transaction surrounding the INSERT statement. This is equivalent to:
```sql
BEGIN;
INSERT INTO students (name, age) VALUES ('Alice', 20);
COMMIT;
```

## Default Isolation Level: READ COMMITTED

The default isolation level in PostgreSQL is READ COMMITTED. At this level, each transaction sees only data that was committed before the transaction itself was started. Let's consider two concurrent transactions to demonstrate this.

```sql
-- Default isolation level is READ COMMITTED
SELECT * FROM students WHERE name = 'Alice';
```
is same as:
```sql
-- Set isolation level to READ COMMITTED
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Start transaction
BEGIN;
-- Query
SELECT * FROM students WHERE name = 'Alice';
COMMIT;
```
and we can ignore
``SET TRANSACTION ISOLATION LEVEL READ COMMITTED;``
because it is default behavior of ``BEGIN;``

Above is the same as:
```sql
-- Start transaction
BEGIN;
-- Query
SELECT * FROM students WHERE name = 'Alice';
COMMIT;
```

It is clear that simple solution have something behind, now we can move to other chapter were 
we will explain start of transaction with word ``BEGIN;``.

