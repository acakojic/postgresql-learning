# Understanding PostgreSQL Transactions with `BEGIN`

## Table of Contents

1. [Introduction](#introduction)
2. [The Role of `BEGIN`](#the-role-of-begin)
3. [Ending a Transaction](#ending-a-transaction)
4. [Scenario: Uncommitted Transactions and Multiple Clients](#scenario-uncommitted-transactions-and-multiple-clients)
5. [Conclusion](#conclusion)

## Introduction

Transactions in PostgreSQL help maintain the ACID properties—Atomicity, Consistency, Isolation, and
Durability—of the database. One of the fundamental aspects of transactions is the `BEGIN` keyword,
which marks the start of a transaction block. This article explores the mechanics and implications
of `BEGIN`.

## The Role of `BEGIN`

The `BEGIN` keyword signals the start of a transaction. When a transaction is initiated
with `BEGIN`, the subsequent SQL queries will be part of this transaction until it is either
committed using `COMMIT` or rolled back using `ROLLBACK`.

```sql
-- Start the transaction
BEGIN;
-- Subsequent SQL statements go here
```

## Ending a Transaction

Once a transaction is started with `BEGIN`, it can be ended in two ways:

1. `COMMIT`: This will make all changes made during the transaction
    ```sql
    COMMIT; 
    ```
2. `ROLLBACK`: If you encounter an error or need to undo the changes, use `ROLLBACK`.
    ```sql
    ROLLBACK; 
    ```

## Scenario: Uncommitted Transactions and Multiple Clients

An interesting scenario arises when a transaction is initiated but not committed. Let's consider two
clients:
<br>
Client `A` and Client `B`, interacting with the same PostgreSQL database.

### Client A

Client A starts a transaction and updates some data but does not commit.

```sql
-- Client A
BEGIN;
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
-- No COMMIT or ROLLBACK yet
```

### Client B

Now, let's see how this uncommitted transaction affects Client B:

1. `Default Isolation (READ COMMITTED)`:
   <br>If Client B tries to read the `balance` for `id = 1`, it will see the old balance, as the
   changes from Client A are not yet committed.
    ```sql
    -- Client B
    SELECT balance FROM accounts WHERE id = 1;
    ```
2. `Isolation Level as SERIALIZABLE`:
   <br>If the isolation level is set to SERIALIZABLE,
   Client B may get a serialization failure if it tries to update the accounts table while Client
   A's transaction is still open.
    ```sql
    -- Client B
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    BEGIN;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    ```

### Uncommitted Transaction Effects

If Client A finally decides to `ROLLBACK` or is terminated unexpectedly, all the changes made will
be undone, and it will be as if the transaction never happened.

## Conclusion

The `BEGIN` keyword is crucial for transaction management in PostgreSQL. Understanding its behavior
and implications, especially in multi-client environments, is essential for effective database
operation and consistency. Learning how to manage uncommitted transactions can help in designing
robust database applications.