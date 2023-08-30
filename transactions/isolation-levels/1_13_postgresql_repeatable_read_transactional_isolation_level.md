# PostgreSQL Repeatable Read Transaction Isolation Level

When working with databases, it's essential to understand how transactions interact with each other,
especially when dealing with concurrent operations. In PostgreSQL, one of the key mechanisms that
control this interaction is the transaction isolation level. In this article, we will focus on the
'REPEATABLE READ' transaction isolation level in PostgreSQL, explaining its behavior and giving some
practical examples to illustrate its functionality.

## What Are Transaction Isolation Levels?

Transaction isolation levels determine the extent to which operations in one transaction are
isolated from those in other transactions. The SQL standard defines four isolation levels:

1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE

<br>

These isolation levels aim to prevent or allow certain types of concurrency issues:

1. `Dirty Reads`: One transaction sees uncommitted changes from another transaction.
2. `Non-repeatable Reads`: A row read once within the same transaction is modified by another
   transaction before being read again.
3. `Phantom Reads`: A query in a transaction returns different sets of rows when rerun due to
   changes made by another transaction.

## Repeatable Read Isolation Level

### Behavior

The `REPEATABLE READ` level in PostgreSQL has the following characteristics:

1. `Dirty Reads`: Prevented. Transactions do not see uncommitted changes from other transactions.
2. `Non-repeatable Reads`: Prevented. A transaction retains a consistent view of a row throughout
   its lifespan, even if the row is modified by other transactions.
3. `Phantom Reads`: Allowed but limited. Due to the use of Multi-Version Concurrency Control (MVCC),
   a `REPEATABLE READ` transaction in PostgreSQL won't generally see newly inserted rows by other
   transactions that commit after it starts.

### How to use

To initiate a transaction with `REPEATABLE READ`, use the following SQL commands:
```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Your transaction operations here

COMMIT;
```

### Practical Examples
To better understand `REPEATABLE READ`, let's look at some practical scenarios using a `products` table:

#### Initial Setup
```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50),
  price INT
);

INSERT INTO products (name, price) VALUES
('Apple', 1),
('Banana', 2),
('Cherry', 3);
```

#### Scenario: Non-repeatable Read with READ COMMITTED
1. Transaction 1 (READ COMMITTED)
```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM products WHERE name = 'Apple';
```
This returns Apple with a price of 1.

2. Transaction 2
```sql
UPDATE products SET price = 5 WHERE name = 'Apple';
COMMIT;
```

3. Transaction 1 (READ COMMITTED)
```sql
SELECT * FROM products WHERE name = 'Apple';
```
This now returns Apple with a price of 5, causing a non-repeatable read.

#### Scenario: Using REPEATABLE READ
1. Transaction 1 (REPEATABLE READ)
```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM products WHERE name = 'Apple';
```
This returns Apple with a price of 1 (or 5, depending on previous transactions).

2. Transaction 2
```sql
UPDATE products SET price = 10 WHERE name = 'Apple';
COMMIT;
```

3. Transaction 1 (REPEATABLE READ)
```sql
SELECT * FROM products WHERE name = 'Apple';
```
This will still return Apple with the original price, preventing a non-repeatable read.

## Conclusion
The `REPEATABLE READ` transaction isolation level in PostgreSQL is a powerful tool for managing concurrent transactions. It prevents dirty and non-repeatable reads but allows limited phantom reads due to PostgreSQL's MVCC mechanism. However, it's essential to be aware of the performance implications and the potential for serialization failures. Always test thoroughly to ensure that the chosen isolation level suits your application's requirements.
