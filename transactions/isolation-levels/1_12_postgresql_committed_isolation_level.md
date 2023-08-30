# PostgreSQL Committed Read Transaction Isolation Level

PostgreSQL is one of the most popular open-source relational database management systems (RDBMS) in
use today. It offers various isolation levels to control the visibility of uncommitted changes
during a transaction. In this article, we'll delve into the `Committed Read` isolation level, which
actually doesn't exist natively in PostgreSQL but is comparable to the `READ COMMITTED` isolation
level it offers. We'll explore its significance, how it works, and what scenarios it's best suited
for, along with practical examples.

## What is Committed Read Isolation Level?

The term `Committed Read` is generally used to describe an isolation level in which a transaction
only
sees committed changes from other transactions at the time each query is executed. This means each
query within the transaction sees a snapshot of committed data as of that moment, rather than a
consistent snapshot of the database throughout the life of the transaction.

In PostgreSQL, this isolation level is implemented as `READ COMMITTED`.

## Characteristics of READ COMMITTED Isolation Level

Here are some key features of this isolation level:

* Phantom Reads: Possible
* Non-repeatable Reads: Possible
* Dirty Reads: Not possible
* Consistency: Medium

# When to Use READ COMMITTED?

`READ COMMITTED` is useful for workloads where you want to avoid dirty reads but are less concerned
about non-repeatable reads or phantom reads. It's typically suitable for short transactions where
the risk of conflicts is low.

## Example

To understand how `READ COMMITTED` works, let's walk through an example using a simple table named
`employees`.

First, let's create the table and populate it with some initial data:
```sql
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name TEXT,
  salary INT
);

INSERT INTO employees (name, salary) VALUES
  ('Alice', 50000),
  ('Bob', 60000),
  ('Charlie', 70000);
```

### Scenario
Two transactions occur concurrently:

1. Transaction A updates the salary of Alice.
2. Transaction B reads the salary of Alice.

#### Transaction A
In one terminal, start a transaction and update Alice's salary but don't commit yet.
```sql
-- Start Transaction A
BEGIN;

-- Update Alice's salary
UPDATE employees SET salary = 55000 WHERE name = 'Alice';
```

#### Transaction B
In another terminal, set the isolation level to `READ COMMITTED` and read Alice's salary.
```sql
-- Start Transaction B with READ COMMITTED isolation level
BEGIN ISOLATION LEVEL READ COMMITTED;

-- Read Alice's salary
SELECT * FROM employees WHERE name = 'Alice';
```

Since Transaction A has not committed yet, Transaction B will read Alice's old salary of 50000.

Now, go back to Transaction A and commit it.
```sql
-- Commit Transaction A
COMMIT;
```

#### Transaction B Continued
Run the SELECT query in Transaction B again.
```sql
-- Read Alice's salary again
SELECT * FROM employees WHERE name = 'Alice';
```

Now, Transaction B will read the new salary of 55000.

Finally, commit Transaction B.

```sql
-- Commit Transaction B
COMMIT;
```

## Summary
In PostgreSQL, the `READ COMMITTED` isolation level allows transactions to see only committed changes from other transactions as of the time each query is run. This allows for more current reads but opens up the possibility for non-repeatable reads and phantom reads. It's a good choice for applications where you want to prevent dirty reads but can tolerate some level of inconsistency due to concurrent transactions.


