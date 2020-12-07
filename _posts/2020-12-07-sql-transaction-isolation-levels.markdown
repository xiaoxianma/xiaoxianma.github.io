---
layout:     post
title:      "SQL Transaction Isolation Levels"
subtitle:   ""
date:       2020-12-07 15:15:00
author:     "Xiaoxianma"
header-img: "img/home-bg-oakley-on-keyboard.jpg"
tags:
    - db
    - sql
    - transaction
---

As we know that, in order to maintain consistency in a database, it follows ACID properties. Among these four properties (Atomicity, Consistency, Isolation and Durability) Isolation determines how transaction integrity is visible to other users and systems. It means that a transaction should take place in a system in such a way that it is the only transaction that is accessing the resources in a database system.

## Concurrency Side Effects

### Dirty reads
This refers to the situation where transaction A reads an uncommitted set of data that transaction B is working on. This can be problematic if transaction B fails or is rolled back.

### Non-repeatable reads
This is the situation where a piece of data, which is read twice inside the same transaction, cannot be guaranteed to contain the same information. From the example this would mean that the second read picked up the data that user B had changed in the product table.

### Phantom reads
This refers to the case when transaction A inserts or deletes a row from a set of data, that transaction B is currently reading.


## Four Different Isolation Levels

### Read Uncommitted
This isolation level specifies that statements can read rows that have been modified by other transactions, but not yet committed. This is the lowest isolation level and consequently, many side effects are present. Reads are not blocked by exclusive locks and do not need to take shared locks; in essence they can do whatever they want. This means that you will allow a lot of concurrency, but you’ll sacrifice the reliability of the data.

### Read Committed
This is the default isolation level for SQL Server. It stops statements from reading data that has been modified but not yet committed by other transactions. This prevents dirty reads from taking place, but not phantom or non-repeatable reads. It does this by using shared locks for reads.  
`While T1 is updating, T2 is blocked to read`

### Repeatable Read
This isolation level stops statements from reading data that has been modified but not yet committed by other transactions. It also prevents other transactions from modifying data that has been read by the current transaction until it has completed. It does this by generating shared locks on all data that is read and holding these locks until the transaction is finished.
`While T1 is reading, T2 is blocked to update`

### Serializable
Specifies the following:  

- Statements are prevented from reading data that has been modified but not yet committed by other transactions.
- Transactions cannot modify data that has been read by the current transaction until the current transaction completes.
- Other transactions aren’t allowed to insert new rows into a table read by the current transaction, if their key values fall in the range of keys read by any statements in the current transaction. So they are blocked until the current transaction completes.
- Range locks are placed on the range of key values that match the search conditions of each statement executed in a transaction.
`While T1 is reading, T2 is blocked to insert/delete into a table`

## Cheatsheet Table
| Isolation Level  | Dirty reads | Non-repeatable reads | Phantoms    |
|------------------|-------------|----------------------|-------------|
| Read Uncommitted | May occur   | May occur            | May occur   |
| Read Committed   | Don't occur | May occur            | May occur   |
| Repeatable Read  | Don't occur | Don't occur          | May occur   |
| Serializable     | Don't occur | Don't occur          | Don't occur |