# PostgreSQL - Rollback in Transaction

In PostgreSQL, the `ROLLBACK` command is used to undo all the modifications made during the current
transaction. This is useful in maintaining the consistency and integrity of your database,
especially when a transaction involves multiple changes across tables. If an error occurs at any
point in the transaction, you can use `ROLLBACK` to undo any changes made so far, essentially
reverting the database to its state before the transaction began. 

```sql
BEGIN;
INSERT INTO table_name WHERE column_name = 'some_value';
ROLLBACK; -- undo changes
```

### Best Practices
1. `Always Use Transactions`: Wrapping your SQL commands within a transaction (between `BEGIN` and `COMMIT` or `ROLLBACK`) is generally a good practice.

2. `Error Handling`: You should have error-handling logic that will issue a `ROLLBACK` command if any part of the transaction fails.

3. `Short Transactions`: Keep your transactions as short as possible to reduce lock contention.

### Example: Transferring Money Between Accounts
Let's say you have a simple banking system where you need to transfer money between two accounts.

Here's a naive approach without error handling:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

Now, let's improve this with error handling and ROLLBACK.
```sql
DO $$  -- PostgreSQL anonymous code block
DECLARE
  v_error BOOLEAN := FALSE;
BEGIN
  -- Start the transaction
  BEGIN;

    -- Debit Account 1
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    IF NOT FOUND THEN
      RAISE 'Account 1 not found';
    END IF;

    -- Credit Account 2
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    IF NOT FOUND THEN
      RAISE 'Account 2 not found';
    END IF;

    -- If we reach this point, it means no error has occurred
    COMMIT;

EXCEPTION
  WHEN OTHERS THEN
    -- An error occurred, so rollback the transaction
    ROLLBACK;
    v_error := TRUE;
END;
$$;
```
### Explanation:
1. We begin an anonymous code block using `DO $$` and declare a variable `v_error` to track if any error occurs.

2. We start a transaction with `BEGIN`.

3. We execute SQL commands to update the account balances. If any account is not found, we raise an error using the `RAISE` statement.

4. If all commands are successful, we reach the `COMMIT` statement, and the transaction is saved.

5. If any error occurs, the `EXCEPTION` block is executed, issuing a `ROLLBACK` command to undo all changes made during this transaction.

## Conclusion
By using `ROLLBACK` in conjunction with proper error handling, we ensure that the database remains in a consistent state, even when an error occurs during a transaction.
