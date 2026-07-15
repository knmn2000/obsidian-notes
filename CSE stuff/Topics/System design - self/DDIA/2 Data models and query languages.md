> **[[System Design MOC|↑ System Design]]** · DDIA 2/5 · [[1 Reliable scalable and maintainable applications|← prev]] · next → [[3 Storage and retrieval]]

```
The limits of my language mean the limits of my world.
	—Ludwig Wittgenstein, Tractatus Logico-Philosophicus (1922)
```

## NoSQL
Not only SQL 

why was it adopted
- less stringent rules, can store anything anyhow unlike strict SQL schemas
- "A need for greater scalability than [[Relational database|relational databases]] can easily achieve, includ‐
ing very large datasets or very high write [[throughput]] " -> why?
- specialized query operations not supported easily by relational models

todays languages are object oriented, SQL DB is not made up of objects
so we have code in between that translates to and fro objects and schemas.
ORMs help with this (like prisma)


## Object - relational mismatch
Another reason why some people may chose nosql
a lot of todays logic is written with object oriented languages

relational DBs are not really compatible with objects, you have to transform objects to relational DBs structure, ie, into rows and columns. 
ORMs help with this (object relational mapping, its in the name)

you can imagine how linkedin must store data in different tables, and rows (having name, region, industry, school, contact info in different tables), but the frontend basically needs an object structure only. so a ORM is required to make the translation

if the linkedin page, was put in a JSON structure instead, 
then it would be something like
```json
{
"userid": 1,
"name": 'blah',
'industry': 'education',
'school': 'manipal'
'connections': [
	{'userid': 2,...}
	{'userid': 3,...}
	{'userid': 5,...}
	{'userid': 9,...}
]
}
```

^ this kind of structure may be directly consumable by the application code. 
The lack of structure can be seen as an advantage (flexibility ++)

also note that json has high locality of data.

### wdym by normalization

definition: “structuring data to reduce duplication, usually via dividing data into multiple related tables"

idea:
	info that is meaningful to humans, may need to change in the future. if such information is replicated, then the changes need to be propagated across all replications. 
	it would be nice to assign an id to the information, keep the information in one place, and the put the id everywhere.

#note this idea of removing duplication is the idea behind normalization.

from this we can realize that normalization makes sense when there are many to one relationships (many people live in one particular region, many people work in one particular industry)
[[Document DB|document DBs]] dont usually have such structure, this is more of a relational DB style structure. 
support for [[joins]] is very weak in document DBs

even if we start off with a join-free document DB, applications usually develop into needing joins, example:

```
Recommendations
Say you want to add a new feature: one user can write a recommendation for
nother user. The recommendation is shown on the résumé of the user who was
recommended, together with the name and photo of the user making the recom‐
mendation. If the recommender updates their photo, any recommendations they
have written need to reflect the new photo. Therefore, the recommendation
should have a reference to the author’s profile.
```


----
### Difference between document based and relational DBs

document
- better locality of data (better performance because no joins etc needed)
- schema is flexible (both a pro and a con)
- application code's schema can fit closely to document DBs schema 

relational
- strong join support
- many to one, many to many relationships


in document models, we can directly access nested data if we need it, we will have to drill to the place we need to go to. so if data is deeply nested and often accessed, itll be a problem.

---
examples from perplexity
1. Example of Many-to-Many in Relational vs Document Models:
    
    - Relational databases use join tables to handle many-to-many relationships. For example, a "books" table, an "authors" table, and a join table "booksAndAuthors" to link them.
        
    - In a document-oriented database like MongoDB, to avoid joins (which are expensive or unavailable), you might denormalize by embedding lists of related IDs or objects within documents. For instance, storing author IDs directly in the book documents and/or book IDs in author documents.
        
    - This can reduce the need for joins but leads to duplicated data and extra logic in the application to keep data consistent when authors or books change.[](https://www.mongodb.com/community/forums/t/schema-design-many-to-many-relationships-and-normalization/209349)​
        
2. Denormalization Example in Document Databases:
    
    - Suppose you have an Event app with a many-to-many relation between events and guests.
        
    - Instead of separate tables or collections with references, you could embed guest info inside event documents and/or event info inside guest documents.
        
    - This decreases query complexity but means application code must handle updates carefully for data consistency when a guest or event changes.[](https://www.mongodb.com/community/forums/t/denormalized-many-to-many-relation-between-collection/292642)​
        
3. Drawbacks and Additional Work:
    
    - When denormalizing many-to-many data, the application has to manage consistency, for example, updating multiple documents when a related entity changes.
        
    - This can lead to code complexity and harder maintenance since the database no longer enforces data integrity automatically.
        
    - It's a compromise where avoiding expensive joins may improve read performance at the cost of increased application logic and potential data anomalies.
---

### schema on read vs schema on write

schema on read
- no hard schema in the DB. when the data is read, only then the schema is understood. flexible. (document DBs)
schema on write
- you need to conform to a schema when you write the data, which then means all the data has to belong to a schema. ( postgres, mySQL)

schema on read is advantageous for when you have to make updates to the data. like adding a middle name property to users is very simple in mongodb
but it will require a migration in postgres.

SOR is better when data is heterogenous.

### data locality

in document DBs, the data locality helps in a lot of cases.
it does not help when you only need a small part of the document for some use case, because you will be fetching the entire document even though you need a small part. this wont happen in relational DB, you can query what you want exactly.

in relational DB, getting what you want exactly can involve multiple joins etc, so need to measure which performs better for our usecase. 

