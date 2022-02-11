---
layout: post
title: "Preventing Phantom Meetings Using Transactions and Serializable Isolation"
tags: conceptual
image: /assets/images/transactions_counters.png
---

In this post, I discuss how time-slot collisions in a meeting scheduling application can be resolved. First, I discuss the business logic of determining a 'time-slot conflict'. Second, I explain why we need transactions to prevent scheduling of concurrent conflicting meetings. Finally, I do a deep-dive on the different levels of isolation provided by database systems, to better understand why simply using transactions (with their default set-up) does not guarantee against conflicting meetings.    


## Project Scope and Design Goals

This project supports the following features:

- **Add new users and rooms:** The system allows for the addition of new users and rooms (they are treated as 'entities' in the data model). Once added, users and rooms can be included in future meetings.
- **Scheduling of meetings**: The system allows for scheduling of new meetings involving one or more users at a given room.
- **Detect conflicts**: While setting up a meeting, the system will check if there are any time-slot conflicts (explained below) involving any of the proposed participants or the proposed room. 

The system interacts with the user on the Command Line. 

Here's the GitHub Repository hosting the project: [https://github.com/oitee/meetings](https://github.com/oitee/meetings)

The system guarantees that a meeting should be permitted to be scheduled **only if** every participating user and the respective room have no time-slot conflict. Even if one entity has a conflict, the system will reject the request for setting up the meeting and prompt the user to try again.

For the purposes of time-slot conflict-resolution, the system treats users and rooms alike. 

### Resolving Time-Slot Conflicts

What constitutes a time-slot conflict? Simply put, it refers to a situation where a single user or room is assigned to more than one meeting at any given point of time. 

With respect to a meeting for a given time-slot A (with a start-point 's' and an end-point 'e'), there can be five types of time-slot conflicts, as shown below. 

- Case 1: A meeting which started before 's', but is scheduled to complete after the 's' (but before 'e'). 

  <img src="https://user-images.githubusercontent.com/85887016/152988848-8511d267-124d-4b91-9833-0a3277d0e36f.png" width="30%">

- Case 2: A meeting which started before 's', and is scheduled to complete after 'e'.

<img src="https://user-images.githubusercontent.com/85887016/152996562-e19c564f-cb32-4a9c-b815-cd2795220b29.png" width="30%">

- Case 3: A meeting which has the same start and end points as time-slot A.

 <img src="https://user-images.githubusercontent.com/85887016/152996595-c2658851-b00d-491c-a777-c939665b433e.png" width="30%">

- Case 4: A meeting which started and ended between points 's' and 'e'.

<img src="https://user-images.githubusercontent.com/85887016/152996634-f4a8cf78-7139-480a-b7a5-0a6a6d2b7108.png" width="30%">

- Case 5: A meeting which started after 's' but before 'e'.

<img src="https://user-images.githubusercontent.com/85887016/152997031-a6f03ac5-0482-4ff5-ac77-09a5e4c7aea2.png" width="30%">

In each of these cases, there is a time-slot conflict, i.e., at least one point where there are two simultaneous meetings.

### How to Resolve Time-Slot Conflicts?

Obviously, if the parties involved in two meetings with conflicting time-slots are distinct and separate, there is no issue. The system will allow both the meetings to continue.(_Note here that when we use 'participants', we include both users and rooms_).

Thus, only if there is at least one common participant between the two conflicting time-slots, do we need to be careful. Thus, at the time of scheduling each meeting, we need to check if there is any potential conflict with respect to any of the participants of the meeting. This can be done using the following SQL query:

```sql
 SELECT entity FROM bookings WHERE 
    entity IN (entity1, entity2... entityN) 
    AND (
            (from_ts <= to_timestamp(start_point) AND to_ts >= to_timestamp(start_point))
            OR
            (from_ts >= to_timestamp(start_point) AND from_ts <= to_timestamp(end_point))
        )

```
In the above example, `(entity1, entity2... entityN)` represents the list of all the participating entities of a proposed meeting and `start_point` and `end_point` represent the two end-points of the time-slot of the proposed meeting.

At the time of creating a new meeting, we run this query on our database. If this returns a non-empty response, it will signify a conflict and the system will prevent the creation of the meeting

## Need for Transactions

In an ideal world, we follow a two-step process while creating a new meeting:
- First, check for conflicts
- Next, insert the new meeting.

This approach has one downside: if there is more than one system trying to write to the database simultaneously, the database may change its state between step one and step two above. For example, let's say there are two systems attempting to simultaneously schedule the same meeting with the same time-slot and entities. It is possible, that read-write sequence interleaves in the following manner:

    System 1 reads the database ... (realizes that there is no conflict)
    System 2 reads the database ... (realizes that there is no conflict)
    System 1 writes the database ...(creates the meeting)
    System 2 writes the database ... (creates the same conflicting meeting)

Thus, we need to take an all-or-nothing approach while reading and writing. This can be achieved by using transactions. 

## What is a Transaction?

Simply put, a transaction represents a single or _atomic_ unit of work performed by a database management system. A transaction is typically used to group multiple reads and writes into one logical unit. Because of their atomic nature, transactions cannot be broken down into its constituent actions: if, in the middle of a transaction, an error or failure takes place which prevents the transaction from being successfully completed, the database will rollback all the intermediate operations of that transaction. 

This is very useful for our use-case, as we can use transactions to ensure an all-or-nothing approach while scheduling meetings: if our reading and writing operations form part of a single transaction, we can potentially prevent partial failures, like the one discussed above.

## ACID Properties

Every database transaction has four key properties: atomicity, consistency, isolation and durability (commonly referred to as 'ACID'). These ACID properties guarantee data validity even in the events of failures, errors and other mishaps.

**Atomicity**: Every operation in the transaction should either all succeed (also called 'committed') or all fail. Partial failure or partial success is disallowed. Without the atomicity guarantee, if an error or a failure takes place during a transaction, it can get very difficult to reason about which operations were successful and which need to be tried again.

**Consistency**: If the database is consistent before execution of a transaction, it should remain consistent after the transaction has been committed. In other words, a transaction should take the database from one valid state to another. If there are any rules or _invariants_ enforced on the data, they should continue to be respected after a transaction is completed. (In fact, consistency is a property of the application layer instead of the database system itself, as the latter cannot prevent the violation of invariants if the application feeds improper or erroneous data. For this reason, it is said that "_the letter C doesn’t really belong in ACID_" [1]). 

**Isolation**: Often, a database needs to execute multiple transactions concurrently. This property provides a guarantee that concurrent transactions will be executed _as if_ they were sequentially or serially executed. In other words, the goal of isolation is to ensure that simultaneous transactions making writes on the same set of objects (rows) of a database should not step onto each other's toes.

**Durability**: Once a transaction is committed, it should persist on the database, even in the wake of a system failure. In the case of single-node databases, this is usually achieved by storing results of transactions on non-volatile memory(disk). In the case of replicated databases, this is achieved by copying the data written during a transaction to a certain number of nodes of that database.

## Writing Transactions in PostgreSQL

To fold multiple queries into one transaction, we should place them between `BEGIN` and `COMMIT` commands. During the middle of a transaction, if our application layer needs to withdraw a transaction,we should use `ROLLBACK` instead of `COMMIT`.

Here's how the SQL expressions for scheduling a meeting can be written as a part of one transaction:

```sql
BEGIN
SELECT entity FROM bookings WHERE
    entity IN (entity1, entity2... entityN)
    AND (
            (from_ts <= to_timestamp(start_point) AND to_ts >= to_timestamp(start_point))
            OR
            (from_ts >= to_timestamp(start_point) AND from_ts <= to_timestamp(end_point))
        )

-- application layer logic: if rows.length > 0 
ROLLBACK;

-- application layer logic: else
INSERT INTO bookings (meeting_id, entity, from_ts, to_ts, created_at, updated_at) 
        VALUES (...);
COMMIT;
```

## Testing with Concurrent Queries

Given the guarantees provided by transactions, we should expect that our application does not schedule conflicting meetings. To test this hypothesis, we can set up a test that makes concurrent and identical queries on the database. To implement this test, I've used the [worker threads](https://nodejs.org/api/worker_threads.html) module, to create `n` number of worker threads that make the same query on the database. See the test here: [https://github.com/oitee/meetings/blob/33aab6b/test/concurrent_requests.js](https://github.com/oitee/meetings/blob/33aab6b/test/concurrent_requests.js)

When I ran this test for the first time, it passed. But when I ran the same test sequentially for ten times (using ```for i in `seq 1 10`; do npm test; done```), it failed twice out of the ten times. For the next ten tests, it failed three times out of ten

<iframe width="560" height="315" src="https://www.youtube.com/embed/Xp3qLWtQ4H8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


```
              meeting_id              |        entity        |        from_ts         |         to_ts          |          created_at           |          updated_at           
--------------------------------------+----------------------+------------------------+------------------------+-------------------------------+-------------------------------
 5f4b0d41-4361-4771-b835-ab0414f570c3 | alice                | 2022-02-15 05:30:00+00 | 2022-02-15 06:30:00+00 | 2022-02-11 04:20:03.342489+00 | 2022-02-11 04:20:03.342489+00
 5f4b0d41-4361-4771-b835-ab0414f570c3 | bob                  | 2022-02-15 05:30:00+00 | 2022-02-15 06:30:00+00 | 2022-02-11 04:20:03.342489+00 | 2022-02-11 04:20:03.342489+00
 43f3e972-4b10-4ab5-8e6e-c621e05654e7 | cat                  | 2022-02-15 05:30:00+00 | 2022-02-15 06:30:00+00 | 2022-02-11 04:20:03.349369+00 | 2022-02-11 04:20:03.349369+00
 43f3e972-4b10-4ab5-8e6e-c621e05654e7 | dog                  | 2022-02-15 05:30:00+00 | 2022-02-15 06:30:00+00 | 2022-02-11 04:20:03.349369+00 | 2022-02-11 04:20:03.349369+00
 4d702995-95b1-4daf-a299-371686b64a5e | alice                | 2022-02-15 19:30:00+00 | 2022-02-15 20:30:00+00 | 2022-02-11 04:20:03.353343+00 | 2022-02-11 04:20:03.353343+00
 860f503e-de33-472c-bbce-63eee3d7afcb | bob                  | 2022-02-15 19:30:00+00 | 2022-02-15 20:30:00+00 | 2022-02-11 04:20:03.356832+00 | 2022-02-11 04:20:03.356832+00
 860f503e-de33-472c-bbce-63eee3d7afcb | cat                  | 2022-02-15 19:30:00+00 | 2022-02-15 20:30:00+00 | 2022-02-11 04:20:03.356832+00 | 2022-02-11 04:20:03.356832+00
 acc22da5-d046-48c7-bc19-6cb0a13d9426 | X_0.7775478561424221 | 2021-12-15 05:30:00+00 | 2021-12-15 07:30:00+00 | 2022-02-11 04:20:03.818958+00 | 2022-02-11 04:20:03.818958+00
 a68b8a42-0147-4044-8362-ac94400c5fe1 | X_0.7775478561424221 | 2021-12-15 05:30:00+00 | 2021-12-15 07:30:00+00 | 2022-02-11 04:20:03.824797+00 | 2022-02-11 04:20:03.824797+00
 e4e602ac-58d4-48de-858c-6f9c7491270a | X_0.7775478561424221 | 2021-12-15 05:30:00+00 | 2021-12-15 07:30:00+00 | 2022-02-11 04:20:03.806509+00 | 2022-02-11 04:20:03.806509+00
(10 rows)

```

When a test fails, the same meeting (as shown in the above schema) with the same time-slot (between `2021-12-15 05:30:00+00` and `2021-12-15 07:30:00+00`) gets inserted multiple times.  So, why did my tests fail _some of the times_? What happened to the ACID properties of transactions?



## Different Levels of Isolation

To better understand why my tests were failing sporadically, it is important to understand that databases enforce different degrees of isolation among concurrent transactions. Note that, 'isolation' was described above as a guarantee that the database system will execute concurrent transactions in such a manner that it will _appear as if_ they were executed serially, i.e., one after the other. In fact, serial isolation (i.e., converting multiple concurrent transactions into a set of sequential transactions) is rarely used in practice. This is because serializable isolation has substantial performance costs which can slow down the response time of a database system. For this reason, most databases provide weaker levels of isolation:

> "_Even on a single-node database, the penalties associated with providing serializability can be severe, including decreased concurrency, reduced performance, and the possibility of deadlock. Accordingly, since the earliest database systems such as System R in 1976, databases have provided a range of user configurable “weak isolation” properties. These properties do not guarantee serializability but offer benefits such as increased concurrency and ease of implementation._" [[1](http://www.bailis.org/papers/hat-hotos2013.pdf)].

When two or more concurrent transactions try to write on the same object of a database or one transaction reads an object that is being concurrently modified by another concurrent transaction, we can have concurrency issues, which are also called 'race conditions'. Each level of isolation provides guarantees against some or all race conditions. 


### Read Committed

When a change made by a transaction has been committed, that change becomes permanent on the database and the transaction loses its right to 'undo' that change. However, uncommitted changes are always revocable. Working with uncommitted changes should ideally be avoided. Reading some other transaction's uncommitted changes is called 'dirty reads'. Writing on another transaction's uncommitted changes is called 'dirty writes'

'Read committed'—the first (or weakest) level of isolation—provides guarantees against dirty reads and dirty writes. This is the default isolation level in PostgreSQL.

Dirty writes are prevented by locking relevant rows where writes take place. When a transaction needs to write a specific row, the database will lock that row. Only once the transaction is completed (aborted or committed) will this lock be opened. At a time, only one transaction can lock a row. So if there is a second transaction that needs to write on the same row, it needs to wait for the first transaction to be completed. 

As for prevention of dirty reads, when a write lock is applied on a row, the database maintains two values for that row: the original value and the uncommitted value. Thus, read-only transactions can access the original value till the time the write lock is lifted. 

**Read committed will not be adequate for our use-case, as our application does not rely on dirty reads or writes.** 


### Snapshot Isolation and Repeatable Read

In a snapshot isolation, each transaction works on a _consistent snapshot_ of the database, i.e., the database as it stood at the beginning of the transaction. 

Snapshot isolation is implemented by a technique called multi-version concurrency control (MVCC). The core principle of snapshot isolation is that for each transaction, they will read a consistent snapshot of the database, as it stood, when that transaction began. As a corollary, if the database progressed further (i.e., some uncommitted changes were committed during the course of a transaction), the transaction will not see those future changes. This ensures that writes do not block reads, and reads do not block writes. This can be especially useful for taking backups of a large database: the transaction making a copy of the database at a particular point in time (a read-only transaction) will not be impeded by other transactions that are writing on some of the rows of that database. However, note that snapshot isolation implements write locks as well, i.e., when a transaction is writing on a row, no other transaction can write it. 

When we implement MVCC, the database may potentially need to maintain several versions of the database, each representing the ‘snapshot’ of the database when each ongoing transaction was initiated. 

Snapshot isolation level is typically referred to as ‘repeatable read’ in SQL. However, these two terms are not exactly identical.

> “_...it defines repeatable read, which looks superficially similar to snapshot isolation. PostgreSQL and MySQL call their snapshot isolation level repeatable read because it meets the requirements of the standard, and so they can claim standards compliance._
>
> _Unfortunately, the SQL standard’s definition of isolation levels is flawed—it is ambiguous, imprecise, and not as implementation-independent as a standard should be. Even though several databases implement repeatable read, there are big differences in the guarantees they actually provide, despite being ostensibly standardized. There has been a formal definition of repeatable read in the research literature, but most implementations don’t satisfy that formal definition. And to top it off, IBM DB2 uses “repeatable read” to refer to serializability. As a result, nobody really knows what repeatable read means_”[1] 

In addition to preventing dirty reads and writes, snapshot isolation also prevents non-repeatable reads: re-reading the same set of rows will not yield a different result. 

Also, PostgreSQL's implementation of repeatable read automatically detects _lost update_. A lost update happens when two concurrent transactions read the same row(s), modify the data and write that modified data on that row(s) (_read-modify-write_ cycle). When two such transactions are executed concurrently, one of the writes will be lost. Take the example of a database that maintains counters. Each transaction is required to read the data of the counter, and increment it by one and write the new data onto the database. Now if two transactions are fired at the same time, they will both read the same value, and they will both update the counter by one. So, while we made two queries for incrementing the value of the same counter, the value actually got increased once. (The other increment was 'lost'). (_I had encountered this particular problem while generating counters for my URL shortening application. [Read here](https://otee.dev/2021/12/20/twirl-link-shortening.html)_).

<img src="/assets/images/transactions_counters.png" width="100%">
    _Source: Designing Data Intensive Applications [1]_

There are two explicit ways to prevent lost updates. First, we can use atomic operations, i.e., we read, modify and write the data in one single query. Second, we use explicit locking. When we use explicit locking, we tell the database to prevent any other transaction from reading or writing on the rows on which our transaction is working on, till the present transaction is completed. This can be done by using `FOR UPDATE` at the end of the `SELECT` query.

Other than these explicit ways, PostgreSQL also automatically detects if there is a lost update during a repeatable read transaction.

**Snapshot isolation will not be adequate for our use-case, as working with consistent snapshots cannot prevent parallel (and concurrent) transactions from making the same writes.** Lost update seems close enough to our use case. However, as our writing operation is akin to creating a new row (as opposed to updating an existing row), Postgres cannot automatically prevent concurrent insertions of the same meeting.

### Serializable Isolation 

Serializable isolation offers the highest degree of protection. 

> "_Serializable isolation is usually regarded as the strongest isolation level. It guarantees that even though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency. Thus, the database guarantees that if the transactions behave correctly when run individually, they con‐ tinue to be correct when run concurrently—in other words, the database prevents all possible race conditions._" [1]

There are three alternative implementations of serializability:

- Serial Execution: Literally executing transactions serially, by using a single thread always.

- Two phase locking: Concurrent reads are permitted on rows where no write operation is underway. If any transaction is making a write operation, it will lock the relevant row and every other transaction that wants to either read or write on that row will need to wait till the first transaction is completed. This approach is also called a _pessimistic concurrency control_ mechanism, because the database system assumes the worst, i.e., every concurrent write operation on a transaction will fail, and guards against that eventuality. (Of course, this is more optimistic than single-threaded serial execution)  

- Serializable Snapshot Isolation: Unlike two-phase locking, in this approach, the database system allows for writes and reads to take place concurrently. At the time of committing a transaction, the database checks if there is any violation of isolation properties, in which case it will abort that transaction. This approach is referred to as the 'optimistic concurrency' approach.

In the case of lost updates (involving a read-modify-write cycle), an easy fix is to use explicit locks on the relevant rows. However, what happens when the first read operation looks for the _absence_ of rows (meeting a certain criteria)? If this criterion is met, i.e., if no rows exist meeting a certain condition, we write the database by inserting a new row. In this case, there is no row we can explicitly lock to prevent a write skew or a lost update. **These kinds of concurrency issues are called 'phantom reads' and this is exactly the reason why our tests often fail.** Because of its nature and the guarantees it provides, serializable isolation can consistently prevent phantom reads. 

### Using Serializable Isolation to Prevent Time Collisions

Let's summarize the different levels of isolation and the respective guarantees they provide:
<br> <br>

| Isolation Level                      | Dirty Reads and Writes | Non-Repeatable Reads | Lost Updates               | Phantom Reads|
|--------------------------------------|------------------------|----------------------|----------------------------|--------------|
| Read Committed                       | Not possible           | Possible             | Possible                   | Possible     |
| Snapshot isolation / Repeatable Read | Not possible           | Not possible         | Not Possible (in Postgres) | Possible     |
| Serializable Isolation               | Not possible           | Not possible         | Not possible               | Not Possible |

<br> <br>

We need to use serializable isolation for our meeting scheduling application, as we know that phantom reads are possible when concurrent transactions try to schedule meetings on the database. So, here's the modified SQL query for inserting a new meeting on the database:

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT entity FROM bookings WHERE 
    entity IN (entity1, entity2... entityN) 
    AND (
            (from_ts <= to_timestamp(start_point) AND to_ts >= to_timestamp(start_point))
            OR
            (from_ts >= to_timestamp(start_point) AND from_ts <= to_timestamp(end_point))
        );
-- if rows.length > 0 
ROLLBACK;
-- else
INSERT INTO bookings (meeting_id, entity, from_ts, to_ts, created_at, updated_at) 
        VALUES (...);
COMMIT;

```

When we use the above query, our tests pass, consistently. 

## References

[1] Martin Kleppmann, Designing Data-Intensive Applications (2017), Ch. 7. 

[2] Peter Bailis et al, HAT, not CAP: Towards Highly Available Transactions, [http://www.bailis.org/papers/hat-hotos2013.pdf](http://www.bailis.org/papers/hat-hotos2013.pdf).

[3] Michael Melanson, Transactions: the limits of isolation, [https://www.michaelmelanson.net/posts/transactions-the-limits-of-isolation/](https://www.michaelmelanson.net/posts/transactions-the-limits-of-isolation/)