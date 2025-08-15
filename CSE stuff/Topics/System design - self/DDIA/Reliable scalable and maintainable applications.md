
#read what is stream processing? (sending a message to another process?)

datastorage tools no longer neatly sit inside traditional categories
- [[redis]] is a datastore, but apparently its also used a message queue
- [[kafka]] is a message store but it has database like durability guarantees. 

example of an app that combines multiple data systems. notice that the application code handles 
![[Pasted image 20250813235033.png]]
#read full text index


## Important concerns
[[Reliability]]:
- system works even in the face of adversity (hardware fault and system faults)
[[Scalability]]:
- Things work even if system grows (traffic volume or complexity)
Maintainability:
- people are able to maintain the current behaviour as well as adapt the system to support new usecases.
--- 
# Reliability
things continue to work the way they were intended even with software or hardware or user faults. 
- ***things that can go wrong are called faults***
we can only prevent certain faults.
fault != failure. failure is when system stops doing its stuff.

#read chaos monkey of netflix
https://netflix.github.io/chaosmonkey/

### hardware faults
when we have 10k disks, and a disk is supposed to die around 10 around years.
then we get about 1 disk to die day.
we usually have redundant setups. 

redundancy may not work for all cases, we are moving towards system that can tolerate loss of machines. 

### software errors
examples:
- #read look into leap second crash on jun30 2012, which messed up apps due to a bug in linux kernel 
- leaks
- cascading fails
---
# Scalability

systems ability to cope with increased load.
`X or Y doesnt scale` -> doesnt mean anything, we should talk about "what if the system grows, then how will things be handled?"


### wtf is load?

#note we describe it using `load parameters`
example:
- requests/second to a server
- read to write ratio to a db
- simultaneous connections
- cache hit rate

### twitter example for load params

Post
- lets say 5k-12k req/sec
timeline-
- lets say 300k/sec (queries to view tweets)

if we just handle for peak case (12k writes a sec), its the naive way.
twitter has a fan out problem
- each user follows many people, and each user is followed by many people.

2 ways to handle it

1 -

```sql

-- Posting a tweet simply inserts the new tweet into a global collection of tweets.
-- When a user requests their home timeline, look up all the people they follow,
-- find all the tweets for each of those users, and merge them (sorted by time). In a relational database like in Figure 1-2, you could write a query such as:

SELECT tweets.

*

, users.

* FROM tweets

JOIN users ON tweets.sender_id = users.id

JOIN follows ON follows.followee_id = users.id

WHERE follows.follower_id = current_user
```
![[Pasted image 20250815143345.png]]



2 -

```
. Maintain a cache for each user’s home timeline—like a mailbox of tweets for

each recipient user (see Figure 1-3). When a user posts a tweet, look up all the

people who follow that user, and insert the new tweet into each of their home

timeline caches. The request to read the home timeline is then cheap, because its

result has been computed ahead of time.
```
![[Pasted image 20250815143355.png]]


the second approach requires a lot of work. 
if a user has 100 followers on average, then a tweet by a user will be replicated 100 times.
that means, 5k twt/seconds ---> 500k writes per second.

but some people have millions of followers, the follower count varies wildly for some users.
hence, we probably shouldnt follow approach 2 for these edge case users (celebrities)

this means, the follower count is a `load parameter`
it decides which approach to follow. 
twitter follows both approaches.
app1 for celebs, and app2 to for others. 

## Performance

how do resources (cpu memory network etc) change with load params.

latency vs response time
- The response time is what the client sees: besides
the actual time to process the request (the service time), it includes
network delays and queueing delays. Latency is the duration that a
request is waiting to be handled—during which it is latent, await‐
ing service.

random reasons for latency:
-  context switch to a background process, the loss of a
network packet and TCP retransmission, a garbage collection pause, a page fault
forcing a read from disk, mechanical vibrations in the server rack.

#insight average response time isnt a good indicator of how users experience the delay, we should use percentiles and take the median. 
median of 200ms means 50% users had less than 200ms response time.

this median is called p50.

sometimes its useful to note the p95, p99 and p999.


```
 For example, if the 95th percentile response time
is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out
of 100 requests take 1.5 seconds or more. 
```

these high percentiles are called tail latencies

```
For example, Amazon
describes response time requirements for internal services in terms of the 99.9th per‐
centile, even though it only affects 1 in 1,000 requests.
```

big companies like amazon care about p999 because these 0.001 people are probably their highest paying customers. 
```
Amazon has also observed that a 100 ms increase in response time reduces
sales by 1%
```

p9999 may be too much to optimize for. 

queuing delays are often a large part of these high percentiles.
a server can only handle so many parallel requests. this will cause blocking of further requests, called of #note head-of-line blocking

in practice
```
Even if only a small percentage of backend calls are slow, the
chance of getting a slow call increases if an end-user request requires multiple back‐
end calls, and so a higher proportion of end-user requests end up being slow (an
effect known as **tail latency amplification**
```
^^^ there are some sophesticated ways to get percentiles instead of just the standard histograms that solve this issue


### Coping with load

horizontal scaling and vertical scaling
#note Distributing load across multiple machines is also known as a shared-nothing architecture. 

there is no one size fits all
- volume of reads
- volume of writes 
- volume of data to store 
- response time requirements
so many diff things to consider

```
a system that is designed to handle 100,000 requests per second, each
1 kB in size, looks very different from a system that is designed for 3 requests per
minute, each 2 GB in size—even though the two systems have the same data through‐
put.
```

#insight load params is what we architect around. if our assumptions for load params are  wrong, then our efforts to handle scaling will be counter productive.

# Maintainability

```
Operability

	Make it easy for operations teams to keep the system running smoothly.

Simplicity

	Make it easy for new engineers to understand the system, by removing as much
	complexity as possible from the system. (Note this is not the same as simplicity
	of the user interface.)

Evolvability

	Make it easy for engineers to make changes to the system in the future, adapting
	it for unanticipated use cases as requirements change. Also known as extensibility, modifiability, or plasticity.
```

operability
- good docs
- good monitoring
- avoid dependence
- predictable behaviour
- self healing if possible

Simplicity
- remove complexity by abstraction
- ex: SQL

Evolvability
- TDD (lol)

****