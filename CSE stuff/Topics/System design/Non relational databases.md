### Agenda:
- Non reln DB
- Design: Slack's realtime comms

![[Screenshot 2024-11-01 at 4.08.35 PM.png]]

Note: There are 2 types of consistencies #read 
		- One in [[CAP]] theorem, and one in [[ACID]].
		- We are concerned with the CAP theorem [[consistency]]
		- What is the difference? [[Diff between consistencies]]


Please understand that we cant just say that non reln DB dont scale, we can even have unscalable reln DB. 
We give up features that we dont want to have the features that we want, and things will scale.
if consistency is not important, we can just choose non reln DB and use its features.


## Types of non reln DBs
 ![[Screenshot 2024-11-01 at 4.22.11 PM.png]]

													Aerospike

[[Document DB]] s vs [[Key Value store]]? (in general, we may have solutions that dont conform to these points)
- Partial updates in document DBs are easy
	- Cross shard queries are expensive
- in key value store, for partial updates, you have to do more steps #read
- in key value, every operation is key based, we can partition easily. cross shard queries are limited. #read 
	- we get [[partition]], but we limit access to data, because a key value may be in only one DB

- **Choose Document DB**: When data is complex, hierarchical, or needs to be queried by multiple fields, with less emphasis on speed for simple operations.
- **Choose Key-Value Store**: When speed and simplicity are key, especially for high-frequency reads/writes, caching, or storing unstructured data where only the key is needed for retrieval.

![[Screenshot 2024-11-01 at 4.30.09 PM.png]]
![[Screenshot 2024-11-01 at 4.40.58 PM.png]]


in the above example, we have the usecase where we know we will only require a few columns (price and timestamp for getting price average for that day), then we can see that column oriented DB are better for analytics oriented use cases. 

notice that insertion would be expensive for column oriented DB #insight
notice that the scope for compression is maximised in column oriented DB #insight

![[Screenshot 2024-11-01 at 4.41.26 PM.png]]

^^^^ Foundational paper on column oriented DB ^^^^ #read

  ![[Screenshot 2024-11-01 at 4.49.26 PM.png]]
  [[Graph database]]

The idea that we should use graph DB for interactions that can be visualized like a graph is bad.
They dont scale well, are heavy etc.

#### Where should we use graph DB? 
- recommendations, social behaviours, fraud detection 
- Collaborative filtering:
	- User A and B are similar, user A bought X, then user B should also be recommended X
- In fraud detection:
	- For example, in credit card companies, if you usually make purchases that relate to X, and suddenly you make a purchase in Y domain, then its suspicious. for example, person that just does domestic purchases tries to make an international purchase.


![[Screenshot 2024-11-06 at 3.14.34 PM.png]]
![[Pasted image 20241106151850.png]]
when correctness is important, go for [[SQL]] #insight
when we need [[sharding]], high ingestion, go for no sql #insight 

### Designing Slack's realtime comms (text)

![[Pasted image 20241106161851.png]]
![[Pasted image 20241106162009.png]]

#### Schemas for the above features
schema doesnt mean we are using[[ relational database ]]btw
![[Pasted image 20241106163342.png]]
( checkpoint-> the point till which you last read a message in a channel )
realize how we were able to add **checkpoint** attribute to the membership table, because we named it in such a way.

in schemas, try to name things as nouns, user_to_channel_map vs membership 
prefer membership, as it is a different entity. we can extend this concept, add more columns to this
its hard to extend user_to_channel_map, we cant rename it to user_to_channel_to_space_map in the future.

But how do you model the **Messages** schema??
people can DM and people can message into channels as well.
how do you differentiate?

Answer: you dont have to, treat everything as a channel, DMs are also a channel.
this way, we will just have a channel_id everywhere.
![[Pasted image 20241106163600.png]]
![[Pasted image 20241106165749.png]]

on slack, we want persistence, and after that we want the people to be notified. 
if we dont persist first, people will be notified, but the message wont be visible to them until after persistence has completed.

For an enterprise application like slack, persistence is more valuable than fast communication.

The sender will make a API call, to the server, the API will save things to DB as well as tell the edge server to notify the users. 
the receivers will have a socket connection with the edge server, so they dont have to refresh the page to view the message

![[Pasted image 20241106170039.png]]
in whatsapp, we dont care about persistence that much because user's device will persist the message anyway. we want faster communication, unlike slack.

the message will go to the edge server (receiver will have [[socket]] connection with it).
the edge server will give it to [[kafka]], which will eventually process and persist it on the DB.


![[Pasted image 20241106170202.png]]
in zoom, we dont have chat persistence at all, so we just communication between the users. simple [[socket]] connection.

continue 1.47

