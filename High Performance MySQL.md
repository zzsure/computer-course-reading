# CHAPTER 1 MySQL Architecture and History

## MySQL's Logical Architecture
- The topmost layer: connection handling, authentication, security, and so forth.
- The Second layer: code for query parsing, analysis, optimization, caching, and all the built-in functions
- The third layer: storage engine, storing and retrieving all data stored "in" MySQL

## Connection Management and security
Once a client has connected, the server verifies whether the client has privileges for each query it issues

## Optimization and Execution
The optimizer does not really care what storage engine a particular table uses, but the storage engine does affect how the server optimizes the query.

## Concurrency Control
MySQL has to do this at two levels: the server level and the storage engine level.

## Read/Write Locks
In the ddatabase world, locking happens all the time: MySQL has to prevent one client from reading a piece of data while another is changing it.

## Lock Granularity
MySQL, on the other hand, does offer choices. Its storage engines can implement their own locking policies and lock granularitiees.

### Table locks
Table locks have variations for good performance in specific situations. For example, READ LOCAL table locks allow some types of concurrent write operations. Write locks also have a higher prioritty than read locks, so a request for a write lock will advance to the front of the lock queue even if readers are already in the queue.

### Row locks
The locking style that offers the greatest concurrency is the use of row locks. Row-level locking, as this strategy is commonly known, is avalilable in the InnoDB and XtraDB storage engines, among others.

## Transactions
Transactions aren't enough unless the system passes the ACID test.
- Atomicity: A transaction must function as a single indivisible unit of work so that the entire transaction is either applied or rolled back.
- Consistency: The database should always move from one consistent state to the next.
- Isolation: The results of a transaction are usually invisible to other transactions until the transaction is complete.
- Durability: Once committed, a transaction's changes are permanent.

## Isolation levels
- READ UNCOMMITTED: transactions can view the results of uncommitted transactions.
- READ COMMITTED: a transaction will see only those changes made by transactions that were already committed when it began, and its changes won't be visible to others until it has committed.
- REPEATABLE READ: It guarantees that any rows a transaction reads will "look the same" in subsequent reads within the same transaction, but in theory it still allows another tricky problem: phantom reads.
- SERIALIZABLE: In a nutshell, SERIALIZABLE places a lock on every rrow it reads.

## Deadlocks
A deadlock is when two or more transactions are mutually holding and requesting locks on the same resources, creating a cycle of dependencies.

## Transaction Logging
Transaction logging helps make transactions more efficient. Instead of updating the tables on disk each time a change occurs, the storage engine can change its in-memory copy of the data. This is very fast.

## Transactions in MySQL

### AUTOCOMMIT
MySQL operates in AUTOCOMMIT mode by default. This means that unless you've explicitly begun a transaction, it automatically executes each query in a separate transaction.

### Mixing storage engines in transactions
MySQL doesn't manage transactions at the server level. Instead, the underlying storage engines implement transactions themselves. This means you can't reliably mix different engines in a single transaction.

### implicit and explicit locking
InnoDB uses a two-phase locking protocol. It can acquire locks at any time during a transaction, but it does not release them until COMMIT or ROLLBACK.

## Multiversion Concurrency Control
MVCC works by keeping a snapshot of the data as it existed at some point in time. This means transactions can see a consistent view of the data, no matter how long they run.

## MySQL's Storage Engines
When you create a table named MyTable, MySQL stores the table definition in MyTable.frm. You can use the SHOW TABLE STATUS command to display information about tables.
