



## Transactions 
a **transaction** is a single logical unit of work that accesses and possibly modifies the contents of a database.
* Txns access data using **read-and-write** operations.
* In order to maintain consistency in a database, before and after the txn, certain properties are followed - **ACID** properties - Atomicity, Consistency, Isolation, Durability.
* ACID is a set of properties that guarantees that database txns are processed realiably. 