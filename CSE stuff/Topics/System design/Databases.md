(term mention: [[Database]])

### Agenda
- [[Relational database]] & pessimistic locking
- Designing: Airline locking system
- Designing: KV store on relational DB

### [[Relational database]]

Database made out of rows and columns. it is special because we form relations out of the data. 
if things are related, or you want to form relations out of the data, use RDB

They are known for [[ACID]] #read
https://arpitbhayani.me/blogs/atomicity/

**Database indexes**
indexes make **reads** faster, and writes slower. #why #read 
https://www.youtube.com/watch?v=3G293is403I&pp=ygUVYXJwaXQgYmhheWFuaSBpbmRleGVz


### [[Database]] locking

Before an operation that could give inconsistencies if operated on concurrently, we should lock it.

> LOCK operation UNLOCK

**Types of lock**
- exclusive
- shared

**Why to use locks?**
to protect sanity of data.
sanity means: consistency and integrity.
ie, protecting against concurrent updates.

**Risk: Transactional [[deadlock]]** #read
A wants R1, R1 is locked by B, B wants R2 which is locked by A.

Transaction that detects [[deadlock]] kills itself. 
most DBs dont let transactions enter a [[deadlock]] state.

**Shared locks**

if a transaction acquires read lock, then other transactions can also read the data, but *no transactions can write to that row*

>Syntax `SELECT * FROM ... FOR SHARE` #sql

![[Pasted image 20240926154955.png]]

if current transaction wants to write, then we upgrade the lock to exclusive lock

**Exclusive lock**
- other transactions can neither read nor write to the row
>Syntaxt `SELECT * FROM ... FOR UPDATE` #sql 

![[Pasted image 20240926155247.png]]

Transaction 2 will wait for transaction 1 till it completes or aborts, as it has an exclusive lock on a row that transaction 2 needs.
Note that throughput is severely affected here

**SKIP LOCKED**

Removes locked rows from the result set.
> Syntax `SELECT * FROM t where id = 2 FOR UPDATE SKIP LOCKED`

like in the above pic, second transaction waited because id 6 was locked, but 3 and 4 but not locked. we can skip id 6 and continue transaction for 3 and 4

**NOWAIT**

if row in transaction is locked, kill transaction
> Syntax `SELECT * FROM ... where ... FOR UPDATE NOWAIT`


### Airline check-in system

![[Pasted image 20240926155942.png]]

![[Pasted image 20240926160407.png]]
![[Pasted image 20240926160417.png]]
![[Pasted image 20240926160437.png]]

[[SQL]] IMPLEMENTATION

![[Pasted image 20240926163041.png]]

There is an issue here:
For every transaction, the query will be re-evaluated for every transaction that locks and unlocks a row.
T1 acquired lock on R1. T2 to T120 were also trying to get hold of R1 but now they will wait.
Now T1 has successfully written to R1, but R1 now no longer aligns with that T2 to T120 wanted (`user_id` is not null now). **THIS IS CALLED BUSY WAITING**
So T2 to T120 will re-evaluate the query. 
Now T2 will get hold of the next row, and the story repeats.

The above solution will successfully book all the seats without clashes, but its not the best way.

**Better implementation** -> AVOID BUSY WAITING
![[Pasted image 20240926163655.png]]
Much faster. [[Busy waiting]] avoided

#insight Fixed inventory + Contention = Locking

continue 1.35