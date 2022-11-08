---
title: "Why Is Atomicity Not Enough?"
date: 2022-05-11T10:29:19-04:00
draft: false
tags: ["ACID", "Isolation level", "Database transactions", "Concurrent transactions"]
---

When we study transactions in [relational databases](https://en.wikipedia.org/wiki/Relational_database), one of the first
things we learn are the guarantees that a transaction must provide.
[ACID(Atomicity, Consistency, Isolation, Durability)](https://en.wikipedia.org/wiki/ACID) are the properties that we desire. Here, I will discuss the Isolation level in more detail and
show that atomicity alone is not enough when handling concurrency.

One classic example of the importance of atomicity is moving money
between accounts. So, imagine that we have two accounts and we would like to transfer the total amount from one account to another one. In a relational database, what we need to do is three steps:
1. Reading the amount from the source account
2. Update the amount in the source account to 0
3. Add the amount from the source to the destination account

What happens if we lose the connection with the database after completing step 2 and before completing step 3? Without any protection, the money only disappears from our source account.
However, the database transaction ensures that this will not happen.

The atomicity ensures all or nothing. Therefore, in this example, all three
steps will either complete or fail. In other words, there is no way that we
finish the transaction by doing partial work.

There is nothing wrong with this example if we consider that there are not
concurrent transactions. However, if there are concurrent
transactions, the result could be very different. The isolation level helps
us protect against unexpected behaviors in the presence of concurrent
transactions.

Isolation levels were introduced in the [SQL-92](https://en.wikipedia.org/wiki/SQL-92); you can find the entire proposal of this version [here](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt).
There are four different isolation levels: Read Uncommitted, Read committed, Repeatable Read, and Serializable.
We will describe the behaviors and guarantees of each isolation level below. Furthermore, I will show examples of possible problems in these isolation levels and how we can mitigate them.

# Premisses

In all the examples here, we are assuming that the database is a relational database, and we have a table called accounts with the following data only:

| **id** 	| **type** 	| **user_id** 	| **amount** 	|
|--------	|----------	|-------------	|------------	|
| 1      	| C        	| 1           	| 500        	|
| 2      	| S        	| 1           	| 0          	|

Table: Initial state of the table accounts for all the examples

In addition, at the beginning of each example, the table's state is precisely the one shown above. Also, the vertical progression can be seen as a time progression in all the figures below.

# Isolation Levels

In the Read Uncommitted isolation level, two different
transactions may read data before one transaction commit its changes. The figure below illustrates this situation.

![Dirty read example](/why-is-atomicity-not-enough/read_uncommitted.svg)
*Dirty read example*

In this figure, transaction A reads the source account's amount and updates the amount in the source and the destination account. Before transaction A commits its changes, transaction B reads the amount from the source account and receives 0 as the total amount.
On the one hand, it is good that transaction B could read up-to-date data. On the other hand, if transaction A does a rollback, the data in transaction B is not more valid.
This problem is known as [Dirty read](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Dirty_reads), and it could create a lot of problems if you are trying to make decisions based on this data.

In Read committed isolation level, the Dirty read does not happen. However, another problem could occur. The figure below demonstrates this problem.

![Non-repeatable read example](/why-is-atomicity-not-enough/Non-repeatable_read.svg)
*Non-repeatable read example*

In this figure, transaction A reads and updates the amount from the source account. After that, transaction B reads the source's amount. Then, transaction A commits its change. After that, transaction B reads the amount from the source account again, and the values are different. On the one hand, it is good that transaction B has up-to-date data. On the other hand, it could create problems for this transaction if it already took some decisions with the old data. This problem is known as [non-repeatable read](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Non-repeatable_reads).

In Repeatable read isolation level, the Dirty read and non-repeatable read problems do not happen. However, another problem could occur. The figure below illustrates this problem.

![Phantom record example](/why-is-atomicity-not-enough/phantom_records.svg)
*Phantom record example*

In this figure, transaction A reads all the checking account amounts
for the user_id 1 and receives only one record as a result. Transaction B
creates a new checking account for user_id 1 and successfully commits the transaction. Then, transaction A reads all the amounts again, and now it receives two records as a result. Similarly, it is good that transaction A is reading the up-to-date data on the one hand. On the other hand, if it makes some decision before the second reading, the premises of this decision could not be valid anymore. This problem is known as [Phantom records](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Phantom_reads).

The Serializable isolation level protects us against the three types of
problems described previously. To do it, this isolation level runs all the
transactions like they are run in a serial way, which is, one by one.

Maybe you are wondering why not always use the Serializable isolation level. The answer is performance. The database sacrifices performance
to ensure the ACID guarantees.

So, the Read uncommited isolation level could be seen as the more performative but with less guarantees, while the Serializable isolation level is the less performative but with more guarantees. The table below, extracted with some modifications from [here](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt), summarizes the isolation levels and their guarantees and possible problems.


|                      	| **Dirty Ready** 	| **Non-Repeatable Read** 	| **Phantom records** 	|
|----------------------	|-----------------	|-------------------------	|---------------------	|
| **Read Uncommitted** 	| Possible        	| Possible                	| Possible            	|
| **Read Committed**   	| Not possible    	| Possible                	| Possible            	|
| **Repeatable Read**  	| Not possible    	| Not possible            	| Possible            	|
| **Serializable**     	| Not possible    	| Not possible            	| Not possible        	|

Table: Possible problems in each isolation level

# What to do then?

Regarding all the problems and performance described previously, what are our options? We can
use explicit locks in the records we need. In this case, we can
provide the guarantees that the Serializable isolation level offers, but only in the records we choose to lock instead of in all that the transaction touches.

In this case, we decide that performance is significant and rely
on the developer to lock the correct records. On the one hand, this is the perfect balance between performance and guarantees. On the other hand, this could create many problems for us in the presence of concurrent transactions if developers do not block the accurate records.

The figure below shows how to transfer the total amount from the source account to the destination account, guaranteeing that the result will always be the same even with concurrent transactions and regardless of the isolation level.

![Locking records example](/why-is-atomicity-not-enough/locking_records.svg)
*Locking records example*

In this figure, transaction A reads the amount from the source account. The difference here is that we use a write lock that locks all the records
retrieved by this query. The write lock is provided by the command `for update`. Transaction B tries to read the same data using a reading lock. Since transaction A is running with the writing lock, transaction B is waiting for the result from the database. Transaction A updates the amount in the source and destination accounts and commits the operation. Only after this the database responds to transaction B with the up-to-date data.

# More resources

The [SQL-92 proposed](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) is an excellent resource and was where the isolation levels were proposed. The MySQL reference manual has an excellent explanation of [transactions and isolation levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html).
Also, it has a good session about [explicit locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html
).
In addition, Chapter 7 of the book [Designing Data-Intensive Applications](https://dataintensive.net/) is entirely dedicated to transactions and isolation levels.

# References

1. https://en.wikipedia.org/wiki/Relational_database
2. https://en.wikipedia.org/wiki/ACID
3. https://en.wikipedia.org/wiki/SQL-92
4. http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt
5. https://en.wikipedia.org/wiki/Isolation_(database_systems)#Dirty_reads
6. https://en.wikipedia.org/wiki/Isolation_(database_systems)#Non-repeatable_reads
7. https://en.wikipedia.org/wiki/Isolation_(database_systems)#Phantom_reads
8. https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html
9. https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html
10. https://dataintensive.net/
