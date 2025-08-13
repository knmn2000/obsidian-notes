
## Agenda 
- [[CSE stuff/General terms/Caching]] at different levels
- [[Scaling]] 
- delegation
- [[concurrency]]
- communication

all the following things that are being done/discussed reduce the load or help the [[Database]] to work better. it is the most fragile part of a system.
### [[CSE stuff/General terms/Caching]] at different levels
1. main memory of API server 
	- it is limited
	- inconsistency 
		- if there are multiple instances of the service. the maybe difference services are out of sync. we will need a centralized cache in that case.
		- but do we need consistency??? in all cases?
		- maybe a user is always going to server 1, then we can cache things at server 1 for that user. and server 2 has a different cache for different set of users. and when you want consistent caches, we will use the central cache.
2. [[Database]] views ([[Materialized views]]) 
	- DB results coming out of complicated [[joins]] etc can be cached, and we can get data from the cached DB view. we will save the computation of joins.
3. Browser ([[local storage]])
	- someone gave an example that they store personal recommendations in the local storage.  is this how youtube does it? store 100s of recommendations, and whenever user refreshed, fetch random 10-15 out of those recommendations. user thinks that the recommendations are being fetched so fast, but actually all of the recoms were computed already.
4. [[CDN]] (cache response) 
	- first understand what CDN is #read 
	- `user -> CDN -> server -> DB` . if CDN has the answer, the latter half of the communication is saved. things are faster .
5. Disk of [[API]] server 
	- usually disk of server is unused. it stores OS, our npm packages etc. we focus on storing things on the RAM of the server (which is very limited).
	- we can use disk of server to cache things. it is much faster than network calls of course.
		- of course, we wont get consistency. if there are multiple servers, we will not be able to do this, unless we are okay with inconsistency.
6. Load balancer (API calls)

### [[Scaling]]

Question: why cant we scale infinitely in the vertical direction? why does [[AWS]] not give us an option to maybe have 50TB of storage on a server?
Answer: Because of hardware limitations. a phone designed for 256GB of storage will not take 1TB SD card. 

Vertical [[scaling]] pros and cons
- Easy to manage (fewer machines)
- risk of downtime (fewer machines)

Horizontal [[scaling]] pros and cons
- harder to manage (more machines, complicated)
- network partitioning #read
- linear amplification (linearly grow amount of machines)
- fault tolerance (book my show should have followed this)
![[Pasted image 20240924172845.png]]

we should know about the capacity of our machines. how? [[load testing]]. no other way.


If we are building out our medium.com clone, then we should
- scale vertically first
- then horizontally

horizontal scaling can be stretched to infinity.
BUT, we need to ensure that our DB and our cache and actually handle those many concurrent connections.
If they cant, then this scaling doesnt make sense.
hence 
==**ALWAYS SCALE FROM THE BOTTOM UP!**== #insight 

scale DB up first, then the API services,
![[Pasted image 20240924173534.png]]
This has been responsible for a lot of outages.
[[Cascading failures]]

**How to scale a [[Database]]?**
- vertical scaling
- read replicas
	- nuances. how do you decide when to read from replica, and when to read from master. at what times do you write to replica? always or in batch? #read #try
- [[sharding]]
	- creating mutually exclusive subset of data. at the routing layer, we will decide where to send each request. maybe based on user id we will send user to one server, sharding based on user id. #read #try
![[Pasted image 20240925000459.png]]


### [[Delegation]]

#read [[kafka]]

one important insight for performance by hotstar VP engineering

> What does not need to be done in realtime, should not be done in realtime #insight 

![[Pasted image 20240925001903.png]]

in context of building a feature for medium.com where we show the number of blogs of a user in the user's profile card:

![[Pasted image 20240925001948.png]]
the heavy lifting of `total_blogs++` will happen [[async]] by the workers.
#read [[SQS]]

![[Pasted image 20240925002314.png]]
in [[message queues]], the consumers are homogenous, they consume the same thing.
items are pushed into the queue, consumers pull from queue, after pulling, consumers will do the heavy lifting and then remove the item from the queue.
Keep in mind, all the consumers have the same exact logic.
	ex: [[SQS]] [[RabbitMQ]] #read 

**why do we have [[message streams]]?**
	lets say, in the [[message queues]] we want to index the blog as well as increase the blog count. so the consumer will do the `blog_count++` logic , as well as, put the blog on [[elastic search]]
	what if, after incrementing, the consumer crashes and [[elastic search]] never gets the update.
	the root cause of this issue is that we are trying to do 2 things at one time, which is an issue in [[distributed systems]]


> Generally [[message streams]] cost more than [[message queues]] 

in [[message streams]] , the items are in the queue, but the consumers are heterogenous, the independent consumers **iterate** over the items, and process independently. even if one consumer crashes, its okay, that consumer can start again.
Messages/items usually persist here.
example: [[kafka]] [[kinesis]] #read #try 

![[Pasted image 20240925003651.png]]

notice that things are decoupled in the above pic.

**[[kafka]] essentials**
![[Pasted image 20240925003921.png]]
in [[kafka]], we have **topics** . each topic has partitions. in the above pic, `ON_PUBLISH` is the topic, and the blue lines are the partitions

Messages are sent to a topic, and within topics, they are stored in partitions.

Within a partition, ordering is guaranteed. 
Across partition, ordering is not guaranteed.

the messages in the partitions will be deleted as per the deletion policy of kafka config.

**Limitation of [[kafka]]**
number of consumers <= number of partitions

![[Pasted image 20240925004458.png]]
	notice in the above pic, 3 search consumers could be serviced, but the remaining 2  didnt have any messages sent to them.


#read what happens if consumption of a message fails in either queue or stream?


### [[Concurrency]] 

Achieved by threads and/or multiprocessing

**Issue:**
- communication between threads / multiple processes 
- concurrent use of shared resources - [[Database]] , in memory variables etc. (Remember example of simultaneous access of shared bank account)

**Handling concurrency**
	- Locks (optimistic, pessimistic)
	- [[mutex]] and [[semaphores]]
	- go lock free #read ??

An example from medium.com project: if 2 users clap on a blog at the same time, we need to make sure each clap operation is atomic.

### [[Communication]] 

#read #watch actual meaning of REST (arpit bhayani video)
![[Pasted image 20240926003938.png]]

**Short polling** #try 
![[Pasted image 20240926004144.png]]
Disadvantages: [[http]] overhead; too many requests and responses 

**Long polling** #try
![[Pasted image 20240926004305.png]]

**Short polling vs long polling**
- short polling -> we get response right away as we are constantly asking/pinging. 
- long polling -> we keep connection open during the entire duration. one response fetched when operation is done.

**[[Websockets]]** #try 
 ![[Pasted image 20240926004651.png]]
 connection is a persistent [[TCP]] connection.

**[[server]] sent events**  #try
![[Pasted image 20240926005106.png]]
notice that both server sent events and websockets have stock market ticket as an example.
websockets wouldnt be a good usecase there because we are not sending any data to the server giving us stock info. it just needs to be a unidirectional connection, so server sent events are better here.

other examples:
- on twitter, likes update.
- on medium, claps update
- on insta live, reactions update/ likes update


