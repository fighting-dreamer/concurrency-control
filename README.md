# Concurrency Control in Databases

## This project will provide way to simulate various scenarios of concurrent transactions.

### <ins>Database</ins> : 
1. Query processor
2. Storage Engine
3. Server
4. FileSystem

* <ins>**Server**</ins> : It is basically the **executor** for a given request. **Requests** are in form of Queries mostly, in some cases setting values of database.
* <ins>**Query-Processor**</ins> : This component under stand the request and create a execution plan for Server to follow.
* <ins>**Storage Engine**</ins> : This component talks to the filesystem that will eventually hold the data. It serves the calls from the server.
* <ins>**Filesystem**</ins> : This holds the data of our Database.

### DataBase V/S FileSystem
There are few features that Filesystem have that database can't provide : 
1. Performance (given you manage how to use the filesystem).
2. Easy migration to cloud services like Aws S3 etc.
3. Less Cost

But these are rudementary features compared to features that DB provide : 
1. Abstraction of Data structures.
2. Searching Capabilities.
3. Data integrity. 
4. Concurrent operations.
5. Data Security.
6. Recovery.

## we are focusing on Concurrent Operations : 
Concurrent access to data means more than one user is accessing the same data at the same time. 

With this concurrent operations, there are possibilities of Anomalies which may result in Database in in-consistent state.

File system does not provide any procedure to stop anomalies. Whereas DBMS provides a locking system to stop anomalies to occur. 

### What are these Anomolies : 
Lets try to get to all such cases from very basic.
In as database we have two main category of operations : 
1. Read operation.
2. Write Operation.

Database is usually have many entities : 
1. Database
2. Table
3. Rows.
4. Columns.
5. Cell.

* Row : It represent an entry which have related attributes.
* Column : It is usually the particular attribute for a given record/row.
* Cell :  This is single value of given attribute in a given Record/Row.
* Table : Many Rows combine under a single meaning/relation. Term "relation" is exchangeably used for "Table"
* Database : It is collection of many tables.

Read operations are of various types, but essentially we are reading and not making any change. eg. : 
1. you may be reading a table.
2. you may be reading configs of of service.

Write Operations are of much more concern as every change effect the other user as this is change in a shared resource i.e. database. They are of many types :
1. You may add new database.
2. you may add new table in existing database.
3. you may add one or more rows in an existing table.
4. you may update exiting rows for one or more columns.
5. you may delete certain rows.
6. you may update table name.
7. you may delete table.
8. you may update database name.
9. you may delete a database.

Among these writes, you see there are few cetegories emerging, those are : 
1. Writes leading to change in database structure.
2. Writes leading to change in values stored in the database.

If we start considering the otehr aspects that Database provide like : Data security then few more other types of operations come to picture, that are related to Access Control.
1. you may like to revoke/Grant access for a table.
2. you may like to revoke/grant access for a database.

All this leads to different sections of requests to a database : 
1. Database defining requests.
2. Database reading requests.
3. Database manipulation requests.
4. Database access control requests.

SQL (Structured Query Language) the most common format for using databases for day-to-day tasks easily is also structured in such Sections : 
1. **DDL** – Data Definition Language
(CREATE, ALTER, DROP, RENAME, TRUNCATE, COMMENT)
2. **DQl** – Data Query Language
(SELECT)
3. **DML** – Data Manipulation Language
(INSERT, UPDATE, DELETE, MERGE, CALL, EXPLAIN PLAN, LOCK)
4. **DCL** – Data Control Language
(GRANT, REVOKE).

## Using Database :
The main idea of using database is to store our data and use it in required manner to carry out a logical unit of work.
we have understood, the set of operations we can carry out, combining these operations in ordered manner constitutes "**a logical unit of work**".

This logical unit of work is what we commonly call **Transaction**

_Transaction is list of operations to be carried out, that's it._
there is no inherent constraint of transaction to be fulfilled without effecting other transactions. Only constraint is the Atomicity i.e. Either all operations went through OR none of them happened. This is though limited to write operations mostly, as they effect the DB for other concurrent operations. For "read operations", we can't un-read to put it simply.

Enabling Transactions lead to new set of operations and new modules to be part of database system, that effect how query plan should be defined and how server will carry that out and how storage engine will act on it. That component for our understandaing  and for abstraction is "Transaction process Manager" on server (database) side.
Similar "Transaction Manager" sits on the user/client side.

How most "Transaction process Manager" work ?
Well that be covered in later posts and its quite interesting and something I am still exploring more.

How Transaction Manager work i.e. how it manages transaction : 
If we were to define an interface for transaction, it is only having 2 operations.
1. Mark Have Executed the Transaction i.e. : **COMMIT**
2. Discarding any changes made in Transaction i.e. : **ROLLBACK**

To ensure the Server Side understands we are trying to do a transaction, we have a **BEGIN** & **getStatus** method too as seen in JTA (javax.transaction.UserTransaction) 

From above, If we were able to define a Transaction, but IF we were to enable users to do transaction, then in a multi user environment where user is accessing Database concurrently, Atomicity alone can't provide a consistent view of data.
So IF we desire Consistency i.e. changing database state from one valid state to other while maintaining database invariants, what can be done to achive it.
- [ ] Can we let go of the Concurrent execution from users ?
```
This will ensure that consistency is maintained as each operation is moves the data to consistent state and 2 executions are not interfering with each other at all.
```
- [ ] Should we use some kind of Locking to enable 
concurrent execution ?
```
This helps executions to make changes to 2 seperate areas but ensure no concurrent changes are happening to same data at same time, so Some Concurrency while maintaining consistency.
```
- [ ] Should we provide each user its own copy of data to execute upon?
```
Here, we give executions to make changes independently, then the decision is just to which data values to agree upon, and that responsibility may be passed to Database(Server side).
```
- [ ] Can we reorder such operations to ensure consisency ?
```
This is area of study on How to determine the order of operations, though a simple example of all executions happen one after other will ensure consistency.
```
* so...on
*
*
- [ ] Any mixture of above solutions ?
```
Most databases talk about mix of Isolation along with locking way to achieve concurrent exectuions. Isolation is more around having your own copy of data to work upon where based on kind of Isolation, you get different effect of READ and WRITE action.
```

Lets talk about what kind of Concurreny Issues we are dealig with : 
1. **Dirty Write** : Transaction T1 modifies a data item. Another transaction T2 then further modifies that data item before T1 performs a COMMIT or ROLLBACK. If T1 or T2 then performs a ROLLBACK, it is unclear what the correct data value should be. It is a **Write-Write Conflict**.

2. **Dirty Reads** : Transaction T1 modifies a data item. Another transaction T2 then reads that data item before T1 performs a COMMIT or ROLLBACK. If T1 then performs a ROLLBACK, T2 has read a data item that was never committed and so never really existed.
It is a **Write-Read Conflict**.

3. **Non-Repeatable Reads** : Transaction T1 reads a data item. Another transaction T2 then modifies or deletes that data item and commits. If T1 then attempts to reread the data item, it receives a modified value or discovers that the data item has been deleted. It is a **Read-Write Conflict**.

4. **Lost Update** : Transactions try to write/update same data block. It is a **Write-Write Conflict**.

5. **Phantom Reads** : Transaction T1 reads a set of data items satisfying some _search condition_. Transaction T2 then creates data items that satisfy T1’s _search condition_ and commits. If T1 then repeats its read with the same _search condition_, it gets a set of data items different from the first read.

These concurrency issues are commonly called Anomolies.

## Understanding these Anomolies better : 
**Dirty Write** : 
* W1[X], W2[X].. ((C1 OR A1) and (C2, A2) in any order)
```
T1           T2
W1[X]    
            W2[X]
C1/A1   
            C2/A2
```
**Dirty Read** : 
* W1[X], R2[X], ((C1 or A1) and (C2 or A2) in any order)
```
T1           T2
W1[X]    
            R2[X]
C1/A1   
            C2/A2
```
**Non-Repeatable Reads** : 
* R1[X], W2[X], ((C1 or A1) and (C2 or A2) in any order)
```
T1           T2
R1[X]    
            W2[X]
C1/R1   
            C2/R2
```
**Lost Update** : 
* R1[X], W2[X], W1[X], C1, C2
```
T1           T2
R1[X]    
            W2[X]
W1[X]

C1            
            C2
```
**Phantom Reads** : 
* R1[X as per pred], W2[y with pred true], ((C1 or A1) and (C2 or A2) in any order)
```
T1              T2
R1[X ~= pred]    
                W2[Y ~= pred]
C1/R1   
                C2/R2
```

### Caveat : <ins>Phantom Reads V/S Non-Repeatable Reads</ins> : 
**Phantom Reads** says that Transaction actually read something and after few other operations, it tried to read the same data, but got either less no. of Rows [Due to deletion by other transaction] OR got more no. of Rows [Insertion by other Transaction]

but 

**Non-Repeatable Reads** says that Transaction read some data and after few other operations, it tried to read same data but the rows were updated and have changed values.

How can one solve given Anomoly with as limited requirements possible : 
* **Dirty Write** : we can solve this if each transaction is making changes to its own copy of data. If certain transaction commits then the value is updated while if it rollback/abort then the original value is retained.
* **Dirty Read** : we can solve this if each transaction is making changes to its own copy and we limit the other transaction to not be able to read changes made under other transaction(s).
* **Non-Repeatable Reads** : we can solve this if each transaction not only keep track of updated records/rows but also the data read as part of its own copy of data.
* **Lost Update** : Aim of solution is to ensure other transaction can't over-write the vhanges done by given Transaction. Essentially if we provide each transaction its own copy of data, the end result will be either one of the update going through, while other's lost, The only way to ensure serial update for a certain data block is locking that. This results in a new strategy called **Cursor Stability**
* **Phantom Reads** : we can solve this if each transaction not only tracks the updated records but also the data read and keeps map of query used for given data read and provide same result as per that query.

Copy of data for each transaction is acheived using multi-version databases that allow given data to have a seperate version for a given transaction.

We see that, we are talking about putting some kind of limitations and in some way locking if required.
these limitations are mostly trying to seperate the data being operated upon form the data in database.

This  seperation is taken up as Isolations which help in achieving different levels/degree of Consistency.

Most databases come with ANSI SQL Isolation Levels : 
1. **Read UnCommitted** : This isolation level
allows the transaction to be able to read other transaction's update, though it can't make update on data changed by other transaction. i.e. Only READ of un-committed data is allowed.

2. **Read Committed** : This isolation level allows the transaction to only read committed data and only using that committed data, it make updates in a transaction.

3. **Repeatable Reads** : It is one level higher than Read committed as it provide the isolation of Read-committed along with keeping hold on what ever data read whil processing and providing same read output for that data.
4. **Serializable** : It is the highest level of Isolation, where you are only allowing a single transaction to operate on Database, keeping other transaction under pending state. Though all the above Isolation levels are defined keeping Anomolies in mind but this Isolation level is solely there to provide "fully serializable execution". 

The idea behind each level is not only to limit a set of anomolies, but also to have an comparitively stronger consistency guarantee too.

Now that's set most Databases are ACID compliant. What is ACID (Atomicity, Consistency, Isolation, Durability) : These are properties of transaction that are intended to guarantee validity even in the event of errors, power failures, etc.

(Note : **Consistency** here is different from what we read or it means in context of **CAP theorem** (Consistency, Availability, Partition Tolerance))

* **Atomicity** : Transactions are often composed of multiple statements. Atomicity guarantees that each transaction is treated as a single "unit", which either succeeds completely, or fails completely: if any of the statements constituting a transaction fails to complete, the entire transaction fails and the database is left unchanged.
* **Consistency** : Consistency ensures that a transaction can only bring the database from one valid state to another, maintaining database invariants. This does not restrict or talk about concurrent executions.
* **Isolation** : Transactions are often executed concurrently (e.g., multiple transactions reading and writing to a table at the same time). Isolation ensures that concurrent execution of transactions leaves the database in the same state that would have been obtained if the transactions were executed sequentially. This property motivate for concurrent executions.
* **Durability** : Durability guarantees that once a transaction has been committed, it will remain committed even in the case of a system failure (e.g., power outage or crash). This usually means that completed transactions (or their effects) are recorded in non-volatile memory.


