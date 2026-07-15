> **[[System Design MOC|↑ System Design]]** · DDIA 5/5 · [[4 Encoding|← prev]]

******![[Pasted image 20260704001435.png]]
---
## why would you replicate?
- Keep data near to the users
- AVAILABLITY: ![[Pasted image 20260704000922.png|319]]
Amazon has so many zones (us-west-1 us-east-1 us-west-2).
each zone itself has an availability zone inside it. the different availability zones have different power sources, buildings etc etc. so even if 1 or 2 zones go down, we can "fail-over" into the next zone being the primary zone (which means this new zone will become the leader, and all the other zones will just copy stuff from the primary zone)

- [[Scaling|Scale]]: spread out reads across multiple machines.

----------

## Master-slave arch

![[Pasted image 20260705155928.png]]


### synchronous vs asynchronous replication
![[Pasted image 20260705160113.png]]
follower 1 : synchronous. leader waits for follower to say OK
follower 2 : [[async|asynchronous]]. follower1 (leader of follower2) expects that things will _just happen correctly_


usually replication is fast
- but sometimes followers can lag behind (during a failure, or too much load etc)
- then the follower just checks the logs of the leader. follower usually knows uptil what point they were in sync, then follower does the same [[transactions]] as suggested by the leader's logs from the point of when the failure ocurred.

synchronous may sound like a good idea but it obviously blocks writes until follower has processed stuff.

practically, we have one synchronous follower and other async followers. ***semi synchronous***


----
## Failover
- when leader fails -> promote a follower
automatic procedure for this:
- determine leader has failed (health checks fail, no response etc)
- choose a new leader (usually the most up to date follower)
- reconfigure the system to send writes to the new leader

##### What can go wrong?
if async replication: maybe no follower has the latest writes. 
- usually old writes are discarded (MEANS THERE IS NO DURABILITY GUARANTEE)

if 2 leaders are made by mistake (due to miscommunication), then both start accepting writes. this also leads to data corruption/loss of data. usually there are measures to kill one leader if multiples are there.

How do you decide a good timeout for a leader? too short = too many failovers, too long = longer recovery

----
# Methods of replication


### Statement based
- just forward the exact SQL statement recieved to the followers. they can run it on their DB.
- its a bad idea because some statements may have non deterministic functions like RAND() or NOW(). STATEMENTS NEED TO BE DETERMINISTIC 
- autoincrementing columns will mess things up if the data isnt exactly the same across followers

### Write ahead log (WAL) shipping (postgres does this)
we forward the log of exactly what data was changed in what disk block. its very low level information.
this ensures that the data is exact across the followers. 

pros:
Blazing fast and low CPU overhead. The replica just blindly copies disk changes.

disadvantage:
because the data is lowlevel, we need to make sure all the followers are compatible with the leader
The replica must run the exact same operating system, the exact same database version, and have the exact same hardware architecture. You cannot replicate just _one_ table; it's all or nothing.

if a new software update comes, and we want to upgrade, then we will be able to do it with 0 DOWN TIME IF THE LOW LEVEL IMPLEMENTATION DETAIL REMAINS THE SAME
(update the followers, then failover the leader, then update the leader as well)

^ if the low level implementation details changed with the update, then we will have a downtime.


### Logical (row-based) log replication 

one level above WAL
you log all the sql queries, and the replicas execute them
this way, youre agnostic to the env of the replicas, we are just running SQL queries.

pros:
decoupled from software version of db. leader and replicas can be on different versions
disadvantage:
- running actual queries is cpu intensive. slower.

### Trigger-based replication
replication based on same logic/code written by the dev. 
"A trigger lets you register custom application code that is automatically executed
when a data change (write transaction) occurs in a [[Database|database]] system"


----
## Replication lag

if n(reads)>>n(writes) , we should have tons of followers.
but if the replication is synchornous, any replica failure will halt the system

if its async, then its possible some replicas may fall behind, meaning, inconsistent system. (replication lag)
we will get eventual consistency, but its still an issue.

![[Pasted image 20260705230733.png]]

### Reading your own writes

read-after-write consistency or read-your-writes consistency
(prevents the above diagram's issue)

reading the same data that user may have modified, should be fetched from the leader. else the user may be confused.
^ cant always do this, we may potentially keep reading from the leader, which defeats the purpose of replication.

```
For example, user profile information on a social network is nor‐
mally only editable by the owner of the profile, not by anybody else. Thus, a sim‐
ple rule is: always read the user’s own profile from the leader, and any other
users’ profiles from a follower.
```

we can also read from leader based on timestamp. difference between last write and NOW(), we would know where to read from.

we can imagine, that cross device consistency (user may have 2 devices, mobile + desktop) is also a thing. 


### Monotonic reads

make sure that a given user always reads from the same replica. else the following issue will occur
![[Pasted image 20260705232554.png]]

### Consistent prefix reads
didnt understand


---
### Solutions for replication lag
discussed later in part 3


---

## Multi-leader replication








