# Note 36

## Transactions

### Summary

The transaction concept is central to the concurrency control facility in a
database. Concurrency control dictates which queries can be executed at the same
time versus which queries must be executed one after the other (i.e., in a
particular order).

### A Tale of two transactions

You are the proud owner of bank account A with a balance of $1,000. Suppose you
owe your friend Bernice $100 and you wish to pay up your debt. At about the
same time, Cora is transferring $150 to your account.

Any of these transfers requires modifying different rows of a table.

Therefore each transfer of a given amount consists of several SQL statements:

1. balance = getBalance(credit account)
2. setBalance(credit account, balance + amount)
3. balance = getBalance(debit account)
4. setBalance(debit account, balance − amount)

(Step 1 and 3 would use a `SELECT` statement; while steps 2 and 4 would use
an `UPDATE` statement.)

The following table shows the two sets of SQL statements to implement these
monetary transfers:

x: A → B                             | y: B → C
-|-
B’s old balance = getBalance(B)      | A’s old balance = getBalance(A)
B’s new balance = old balance + $100 | A’s new balance = old balance + $150
A’s old balance = getBalance(A)      | C’s old balance = getBalance(C)
A’s new balance = old balance − $100 | C’s new balance = old balance − $150

### Activity

List some of the possible executions of the two bank transfers above. That is,
possible interleavings of the SQL statements. Note that the order of the SQL
statements is fixed within a particular transfer (i.e., Step 1 always precedes
Step 2), but the order across transfers is not. For each proposed ordering, give
the balance of account A when all SQL statements have executed.

Example:

Ordering                       | Account's A final balance
-|-
x1, x2, x3, x4, y1, y2, y3, y4 | $1,050

### Transaction

A transaction is a sequence of database operations that must be either completed
or rejected as a whole. Partial execution of a transaction results in
inconsistent or incorrect data.

Saving complete transaction results in the database is called a commit.
Rejecting an incomplete transaction is called a rollback. A rollback reverses
the transaction and resets data to initial values. A variety of circumstances
cause a rollback:

* The operating system detects a device failure (e.g., a disk failure).
* The database detects a conflict between concurrent transactions (e.g., an
  ordering that interleaves the operations of two transfers).
* The application program detects an unsuccessful database operation (e.g., a
  table referred to by an SQL statement no longer exists).

When a failure occurs, the database is notified and rolls back the transaction.

### ACID properties

All transactions must be Atomic, Consistent, Isolated, and Durable, commonly
called the ACID properties:

* **A**tomic. Either all or none of the operations are executed.
* **C**onsistent. All rules governing data are valid when the transaction is
  committed.
* **I**solated. Transactions behave as if each transaction were executed
  serially.
* **D**urable. Transaction is permanently saved in the database once committed.

### Activity

Which ACID properties are violated, if any, in the various orderings you have
studied in the bank transfer example we studied earlier?

### Isolation

Concurrent transactions T1 and T2 can conflict in many ways. Common conflicts
include dirty read, nonrepeatable read, and phantom read.

Reread the relevant section in the text (Section 6.1, "Isolation" paragraphs,
about halfway down the section).

Looking back again at your orderings, which read conflicts, if any, do they
exhibit?

### Transactions with SQL

In MySQL, with InnoDB, by default, individual statements are separate
transactions. In other words, unless specified otherwise, the database
management system provides no consistency or isolation guarantees across
multiple SQL statements. The MySQL autocommit variable controls this behavior.
By default it is ON.

#### Altering the isolation level

```sql
SET [ GLOBAL | SESSION ] TRANSACTION
ISOLATION LEVEL [ SERIALIZABLE
                | REPEATABLE READ
                | READ COMMITTED
                | READ UNCOMMITTED ];
```

By choosing one of the four possible isolation-level settings, a user can obtain
greater concurrency at the detriment of increasing the transaction’s exposure to
other transactions’ uncommitted changes.

#### Transaction boundaries

A transaction boundary is the first or last statement of a transaction. A
transaction boundary is one of three statements:

* The `START TRANSACTION;` statement starts a new transaction.
* The `COMMIT;` statement commits the current transaction.
* The `ROLLBACK;` statement rolls back the current transaction.

All statements between `START TRANSACTION` and the following `COMMIT` or
`ROLLBACK` comprise a single transaction.

#### Transactions with PHP

```php
try {
    $db = new PDO("mysql:host=$servername;dbname=$database;utf8mb4;port=$port",
                  $username,
                  $password);
    $db->setAttribute(PDO::ATTR_AUTOCOMMIT, 0);
    $db->beginTransaction();
    /* SQL statements */
    $db->commit();
} catch (Exception $e) {
    $db->rollBack();
    return $e->getMessage();
}
return true;
```

### References

Sections 6.1 and 6.5 of the text.
