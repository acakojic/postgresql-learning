# PostgreSQL SERIALIZABLE Transaction Isolation Level

PostgreSQL offers various transaction isolation levels to control the visibility of uncommitted
changes during a transaction. Among these, the `SERIALIZABLE` isolation level provides the highest
degree of isolation. It ensures that when multiple transactions are executed concurrently, the
database reaches a state as if these transactions were executed serially.

## What Does SERIALIZABLE Guarantee?

The 'SERIALIZABLE' level prevents dirty reads, non-repeatable reads, and phantom reads. Essentially,
it guarantees that transactions appear to be serialized, even though they may be executed
concurrently. If two transactions overlap in a manner that could violate serializability, PostgreSQL
rolls back one of the transactions with a serialization failure. This maintains the database's
consistency.

## Example Schema
For demonstration, let's consider a simple `bank_accounts` table:
```sql
CREATE TABLE bank_accounts (
  id SERIAL PRIMARY KEY,
  balance INTEGER
);

-- Insert two rows into the table
INSERT INTO bank_accounts (balance) VALUES (1000), (2000);
```
The initial state of the table is:

| id      | balance |
|---------|---------|
| 1       | 1000    | 
| 2       | 2000    |

### Transaction 1
In this transaction, we'll deduct 200 units from the balance of the account with `id=1`.
```sql
-- Start the transaction and set isolation level
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check balance for account with id=1
SELECT balance FROM bank_accounts WHERE id = 1;
-- Output: balance = 1000

-- Update balance for account with id=1
UPDATE bank_accounts SET balance = balance - 200 WHERE id = 1;

COMMIT;
```

After executing Transaction 1, the `bank_accounts` table would look like:

| id      | balance |
|---------|---------|
| 1       | 800     | 
| 2       | 2000    |


### Transaction 2 (Executed Concurrently)
In this concurrent transaction, we'll add 200 units to the balance of the account with `id=2`.
```sql
-- Start the transaction and set isolation level
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check balance for account with id=1
SELECT balance FROM bank_accounts WHERE id = 1;
-- Output: balance = 800 OR 1000, depending on whether Transaction 1 has committed

-- Update balance for account with id=2
UPDATE bank_accounts SET balance = balance + 200 WHERE id = 2;

COMMIT;
```

After both transactions are executed, the `bank_accounts` table will be:

| id      | balance |
|---------|---------|
| 1       | 800     | 
| 2       | 2200    |

## Summary
The `SERIALIZABLE` isolation level in PostgreSQL ensures the highest level of transactional integrity. 
If Transaction 1 and Transaction 2 overlap in a way that could violate serializability, 
 PostgreSQL will roll back one of the transactions, requiring a retry. 
This mechanism ensures the database remains in a consistent and predictable state.

By understanding and appropriately using the SERIALIZABLE isolation level, 
you can design robust and consistent database systems.