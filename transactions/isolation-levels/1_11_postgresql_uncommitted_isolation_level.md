# PostgreSQL Handling Uncommitted Reads in Transaction Isolation Level

## Uncommitted Read

Uncommitted Read refers to the situation where one transaction is allowed to read data that is being
modified but not yet committed by another transaction. This can result in issues such as dirty
reads, phantom reads, and non-repeatable reads, which compromise the integrity and consistency of
the database.

## Handling in PostgreSQL

PostgreSQL uses a Multi-Version Concurrency Control (MVCC) system to manage transaction isolation.
By default, PostgreSQL operates at a level that prevents uncommitted reads. Specifically, it
operates at the "Read Committed" isolation level as a default, which means one transaction will not
see the uncommitted changes made by another transaction.

In PostgreSQL, you can set different isolation levels using the SET TRANSACTION command, but none of
these levels allow for uncommitted reads by default.

Here are the transaction isolation levels in PostgreSQL:

* Read Committed
* Repeatable Read
* Serializable

Each level provides stronger consistency guarantees, but none of them permit uncommitted reads,
thereby preserving data integrity and consistency.

## Conclusion
So, to put it simply, PostgreSQL handles uncommitted reads by not allowing them in the first place,
thanks to its MVCC architecture and its default "Read Committed" isolation level.