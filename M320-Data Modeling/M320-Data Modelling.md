# M320: Data Modeling
## Chapter 1: Introduction to Data Modelling
### Important Points
#### Analogy: RDBMS & MongoDB
*RDBMS* 		| *MongoDB*
-----------	| -----------
Entity			| Entity
Relationship| Relationship
Database | Database
**Table** | **Collection**
**Attribute/Column**	| **Field**
**Record/Row** | **Document**
**Join**	| **Embed or Link**
#### Misconceptions 
- MongoDB is schemaless so it doesn't matter which fields documents have or how documents differ or no. of collections in a database.
	- All data has some structure , therefore, schema. MongoDB has flexible data model. Its just that MongoDB has very few rules like being correctly defined in BSON and containing primary key.
-  All information regardless of how data should be manipulated can be stored in single document.
	- This may be okay in some use cases but generally this is not how applications use data.
	- For eg: A user profile having use data, that gives current state of users. Profile data may include messages, likes, reviews, friendships, etc. Storing/Updating all this info in single document can cause performance issues to get the latest version of document and degrade application performance.
	
- There is no way to perform a join between documents in MongoDB.
	- Even though **$lookup** is not called as **join** in MongoDB, you can still perform all kinds of joins in MongoDB.

#### Main aspects of MongoDB
- You can run MongoDB anywhere.
- You can use MongoDB for any type of data.
- You can put your data everywhere.
- MongoDB allows applications to  change, scale, and make them more flexible and dynamic.

#### Document Model in MongoDB
- Each mongoDB deployment can have many databases.
- Each database can have one or more collections.
- **J**ava**S**cript **O**bject **N**otation
	- Human readable and writable
	- Machine parsable and generatable.
	- Language independent.
- You can keep your related data in a single document and pull it all down using a single query.
- MongoDB document is similar to a dictionary, map or an associative array.
##### Flexibility of schema:
- Documents in the same collection don't need to have the exact same list of fields.

- Furthermore, the data type in any given field can vary across documents.
- That is, You can have multiple versions of your schema as your application develops over time and all the schema versions can coexist in the same collection.
- MongoDB supports enforcing the schema shape from **no rules**, **rules in a few fields**, to **rules on all the fields** of the documents.
#### Data modelling Constraints
- Hardware:
	- RAM
	- SSD/HDD
-  Data:
	- Size
	- Security, Sovereignty	(Access control)
- Application:
	- Network Latency
	- Database Server/MongoDB
	- Max size of document(16mb), atomicity of updates.(ACID compliance)

- Working set: *"Total body of data that the application uses in the course of normal operations"*
##### Tips for modelling
- Keep frequently used documents in RAM
- Keep indexes in RAM
- Prefer SSD to HDD
- Infrequently used data can be kept on HDDs.
#### Documenting Methodology
- **Phase-1: Identifying the workload of the system.**
	- data size, important reads and writes.
- **Phase-2: Identifying the relationships between pieces of information.**
	- identify them, link or embed related entities. 
- **Phase-3: Applying schema design patterns.**
	- Apply the ones for needed optimizations.

![MongoDB Methodology](https://university-courses.s3.amazonaws.com/M320/methodology.png)


#### Model for Simplicity or Performance
- Doing the right thing first and having a way to share information between the team members is important for having a successful project.
- First of all, recognize the trade off between modeling for simplicity and performance.
- Secondly, use the methodology in a flexible fashion.
- Select the step within a phase that makes sense for your project.
- Finally, regardless of how much you will model, you need to start by describing the workload of your project.

##### Simplicity

- Likely to end up with fewer collections in the design, each document contains more information.
- Avoids any complexity that can slow down development.
- Typically while developing quickly with very few software Engineers.
- Usually limited expectations and smaller requirements in terms of CPU, RAM, I/O, memory, etc.
- For eg. three collections merged into one, less disk accesses are needed to retrieve information. That is, a single read is sufficient instead of four of collections were separated. 
##### Performance
- Resources are likely to be used to maximum, like an engine running at the red line of the gauge.
- Projects that use sharding to scale horizontally generally fall into this category because we often shard when there aren't enough resources. 
- System may require very fast read or writes operation, or it may have to support a ton of operations.
- When modelling for performance, you want to go over all steps of the methodology.
	- Identify important operations and also quantify them in terms of metrics like ops/sec, reqd. latency, identifying if application can work with stale data or are those operations are part of a large query.
- Likely to have more collections in design.
- Need to apply a series of schema design patterns to ensure the best usage of resources like CPU, disk, bandwidth.
##### Balanced
  - For eg: a project that needs to be developed quickly, but also need to apply some transformations and optimization to meet performance requirements
  - Positioning on this access is a bit of an art.
  - It is advised that it is easier to find optimization later on than to remove complexity from an application.

- So prioritize simplicity over performance.

#### Identifying the Workload
CASE STUDY
- Try to quantify and quantify the queries as much as you can. Once you collect the real metrics on your operational system, contrast these with your initial assumption. 
- These differences you observe are needed to adapt the schema design.
- It is likely that a few CRUD operations will drive your design.

## Chapter 2: Relationships
### Important Points
#### Relationships
- Common Relationships:
	- one-to-one		(1-1)	(Customer - Customer_id)
	- one-to-many	(1-N)	(Customer_id - Invoice_id)
	- many-to-many(N-N)	(Invoice_id - product_id)

#### Modified Notations
- Crow Foot Notation	
	- one-to-zillions(Useful in Big Data)
	![one-to-zillions illustration](https://i.ibb.co/CWdbNQS/Annotation-2020-05-23-201822.png)
- Numeral Notation
- minimum
- most likely, median(most likely should be used)
- maximum

##### one-to-many
- An object of a given type is associated with n objects of second type.
- Eg. A person can have n credit cards but each credit card has one and only one person.
- Representations:
	1. Embed 
		1. in the "one" side
		2. in the "many" side

		Usually, embedding in entity the most queried.
	2. References
		1. in the "one" side
		2. in the "many" side

		Usually, referencing in the "many" side

	![One-to-Many, embed in the one side](https://i.ibb.co/gZYt40R/Screenshot-40.png)
	<a href="https://ibb.co/fDv5fq2"><img src="https://i.ibb.co/km1fj5x/Screenshot-41.png" alt="One-to-Many, embed in the many side" border="0"></a>
<a href="https://ibb.co/4MdQKNf"><img src="https://i.ibb.co/DkpqzKb/Screenshot-42.png" alt="One-to-Many, reference in the one side" border="0"></a>
<a href="https://ibb.co/xLCMZ1R"><img src="https://i.ibb.co/9g8qPbC/Screenshot-43.png" alt="One-to-Many, reference in the many side" border="0"></a>

##### many-to-many
- In RDBMS, jump table is used to breakup the many-to-many relationship into two one-many relationships. This does not have to be done with the document model.
- Representations:
	1. Embed 
		1. array of subdocuments in the "many" side
		2. array of subdocuments in the other "many" side

		Usually, only the most 	queried side is considered.
	2. References
		1. array of references in the "many" side
		2. array of references in the other "many" side

		Usually, referencing in the "many" side
<a href="https://ibb.co/Zhm7Ycb"><img src="https://i.ibb.co/sFHz6sX/Screenshot-46.png" alt="array of subdocuments in the 'many' side" border="0"></a>
<a href="https://ibb.co/qj3BwV3"><img src="https://i.ibb.co/XLrXTMr/Screenshot-47.png" alt="array of references in the 'many' side" border="0"></a>
<a href="https://ibb.co/tZvWV7k"><img src="https://i.ibb.co/XyKvchN/Screenshot-48.png" alt="array of references in the other 'many' side" border="0"></a>


- Prefer embedding on the most queried side.
- Prefer embedding for information that is primarily static over time and may benefit from duplication.
- Prefer referencing over embedding to avoid managing duplication.

### Lab: Many-to-Many Relationship

**Problem:
Given the following Collection Relationship Diagram (CRD), identify the relationships that represent Many-to-Many relationships.
We are asking you to identify not only direct Many-to-Many relationships, but also transitives ones. For example a user has a One-to-Many relationship with its reviews and a One-to-Many relationship with its credit cards, making a Many-to-Many relationship between the reviews and the credit cards.**

![https://university-courses.s3.amazonaws.com/M320/lab_relationship_many-to-many-problem.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_many-to-many-problem.png)

**Check all answers that apply:**

- **stores.address.street**  and  **items.description**
- **items.title**  and  **items.reviews.body**
- **items.sold_at**  and  **items.reviews.body**
- **users.credit_cards.number**  and  **items.reviews.body**
- **users.shipping_address.street**  and  **items.reviews.body**

*Answer:*
- **stores.address.street**  and  **items.description**
- **items.sold_at**  and  **items.reviews.body**
- **users.shipping_address.street**  and  **items.reviews.body**

##### one-to-one Relationship
<a href="https://ibb.co/89xKCBV"><img src="https://i.ibb.co/YhdRCWG/Screenshot-49.png" alt="One-to-one representations" border="0"></a>
<a href="https://ibb.co/StgPjjP"><img src="https://i.ibb.co/jyYTNNT/Screenshot-50.png" alt="One-to-one : embed,fields at same level" border="0"></a>
<a href="https://ibb.co/bFh0TdL"><img src="https://i.ibb.co/LrwFmp6/Screenshot-51.png" alt="One-to-one:embed, using subdocuments" border="0"></a>
<a href="https://ibb.co/2hf9KvZ"><img src="https://i.ibb.co/nm5Gk7P/Screenshot-53.png" alt="One-to-one: reference" border="0"></a>
<a href="https://ibb.co/jTRpZJC"><img src="https://i.ibb.co/s2HBJWT/Screenshot-55.png" alt="Recap" border="0"></a>

### Lab: One-to-One Relationship

**Problem:
A legacy database has been ported to MongoDB, resulting in a set of collections that were mapped to their original tables. This port has been quickly identified as a poor solution.
We have been tasked with redesigning the  **employees**  collection to make better use of the document model to make the information clearer.**

![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-problem.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-problem.png)

**While we are restructuring the database, the Human Resources department would like us to move any confidential employee information to a different collection to make the information easier to protect.**
**Consider the following potential schema designs. Each of these designs represents an individual employee and the One-to-One relationship between all of their fields.**
**The ideal schema design should store:**
-   **address information together as an embedded sub-document**
-   **all of an employee's phone numbers together as an embedded sub-document**
-   **all salary and bonus compensation information together as an embedded sub-document**
-   **all confidential information in a separate collection
Once you've identified the ideal design, you can deepen your knowledge by trying to explain why each of the other options is not the preferred design choice.**

**Choose the best answer:**
1.  
![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-1.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-1.png)
2. 
![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-2.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-2.png)
3. 
![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-5.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-5.png)
4. 
![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-3.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-3.png)

![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-4.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-4.png)

*Answer:*

![https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-1.png](https://university-courses.s3.amazonaws.com/M320/lab_relationship_one-to-one-answer-1.png)

##### One-to-Zillion 
<a href="https://ibb.co/D9Nnt65"><img src="https://i.ibb.co/QPZGfqr/Screenshot-56.png" alt="One-to-zillion Representation" border="0"></a>
<a href="https://ibb.co/vqjSDqy"><img src="https://i.ibb.co/YQLxbQn/Screenshot-58.png" alt="One-to-zillion:reference, in the zillion side" border="0"></a>
<a href="https://ibb.co/rkt9FmJ"><img src="https://i.ibb.co/K6GnbXH/Screenshot-59.png" alt="Recap" border="0"></a>


## Chapter 3: Patterns (Part-1)
### Important Points
- Concerns with applying Patterns:
	- Duplication
		- duplicating data across documents
	- Data Staleness
		- accepting staleness in some pieces of data
	- Data integrity Issues
		- writing extra application side logic to ensure referential integrity

#### Duplication
- Why Duplication?
	- Result of embedding information in a given document for faster access
- Concern
	- Challenge for correctness and consistency
##### Case-1: Where duplicating information is better than not doing it.
	
- Let's link orders of products to the address of the customer that placed the order by using a reference to a customer document.
- Updating the address for this customer updates information for the already fulfilled shipments, order that have been already delivered to the customer. This is not the desired behavior.
- The shipments were made to the customer's address at that point in time, either when the order was made or before the customer changed their address.
- So the address reference in a given order is unlikely to be changed.
- Embedding a copy of the address within the shipment document will ensure we keep the correct value.
- When the customer moves, we add another shipping address on file.
- Using this new address for new orders, does not affect the already shipped orders. 
##### Case-2: When the copy data does not ever change.

- Let's say we want to model movies and actors.
- Movies have many actors and actors play in many movies.
- So this is a typical many-to-many relationship.
- Avoiding duplication in a many-to-many relationship requires us to keep two collections and create references between the documents in the two collections.
- If we list the actors in a given movie document, we are creating duplication.
- However, once the movie is released, the list of actors does not change.
- So duplication on this unchanging information is also perfectly acceptable. 

#####  Case-3: Information that needs to or may change with time.

- Let's use the revenues for a given movie, which is stored within the movie, and the revenues earned per screening.
- Oh, yeah, with said duplication add to be a single value in two locations.
- In this case, we have duplication between the sum store in the movie document and the revenue store in the screening documents used to compute the total sum.
- This type of situation, where we must keep multiple values in sync over time, makes us ask the question is the benefit of having this sum precomputed surpassing the cost and trouble of keeping it in sync?
- If yes, then use this computed pattern.
- If not, don't use it.
- Here, if we want the sum to be synchronized, it may be the responsibility of the application to keep it in sync.
- Meaning, whenever the application writes a new document to the collection or updates the value of an existing document, it must update the sum.
- Alternatively, we could add another application or job to do it.

#### Data Staleness
- Staleness is about facing a piece of data to a user that may have been out of date.
- Why staleness?
	- New events come along at such a fast rate that updating data constantly can cause performance issues.
- Concern
	- Data quality and reliability.

- A common a way to refresh stale data is to use a Change Stream to see what has changed in some documents and derive a list of dependent piece of data to be updated.
- **Change Stream**'s a new application to access and respond to data changes, either in real time or in a delayed mode.
#### Referential Integrity
- Why Referential Integrity?
	- Linking information between documents or tables.
	- No support for cascading deletes.
- Concern?
		- Challenge for correctness and consistency
	
#### Attribute Pattern
- Example-1
	<a href="https://ibb.co/C5cVp9h"><img src="https://i.ibb.co/vxFLCdZ/Screenshot-62.png" alt="Using Attribute pattern" border="0"></a>
	- To search effectively on one of all product fields, you need an index.
	- For example, searching on the capacity for my battery would require an index.
	- Searching on the voltage output of my battery would also require an index.
	- If you have tons of fields, you may have a lot of indexes.
For this case you want to use the attribute pattern.
	- To use the attribute pattern you start by identifying the list of fields you want to transpose.
	- Here we transpose the fields input, output, and capacity.
	- Then for each field in associated value, we create that pair.
	- The name of the keys for those pairs do not matter.

- Example-2
	- Here's an example in which we keep track of the dates when a movie was released in the USA, in Mexico, and France, and when it appears in the San Jose movie festival.
	- One thing to observe with those fields is that they share the same type of value.
	- In this case, the type, date - more conceptually, a release date.
	- What if we want to find all the movies released between two dates across all countries?
	- Using the attribute pattern and transforming the release dates to an array of field pairs, we can change the query to this.
	<a href="https://ibb.co/w43tQmT"><img src="https://i.ibb.co/0r859bk/Screenshot-60.png" alt="Fields sharing common characteristics" border="0"></a>

- Recap
<a href="https://ibb.co/MS328FJ"><img src="https://i.ibb.co/DWSVG0j/Screenshot-61.png" alt="Attribute Pattern" border="0"></a>


### Lab: Apply the Attribute Pattern

**Problem:
	User Story:
The museum we work at has grown from a local attraction to one that is seen as having very popular items.
For this reason, other museums in the World have started exchanging pieces of art with our museum.
Our database was tracking if our pieces are on display and where they are in the museum.
To track the pieces we started exchanging with other museum, we added an array called  events, in which we created an entry for each date a piece was loaned and the museum it was loaned to.**

    {
      "_id": ObjectId("5c5348f5be09bedd4f196f18"),
      "title": "Cookies in the sky",
      "artist": "Michelle Vinci",
      "date_acquisition": ISODate("2017-12-25T00:00:00.000Z"),
      "location": "Blue Room, 20A",
      "on_display": false,
      "in_house": false,
      "events": [{
        "moma": ISODate("2019-01-31T00:00:00.000Z"),
        "louvres": ISODate("2020-01-01T00:00:00.000Z")
      }]
    }
**The problem with this design is that we need to build a new index every time there is a new museum with which we start exchanging pieces. For example, when we started working with The Prado in Madrid, we needed to add this index:**

    { "events.prado" : 1 }

----------

**Task**

**To address this issue, you've decided to change the schema to:**

-   **use a single index on all event dates.**
-   **transform the field that tracks the date when a piece was acquired,  date_acquisition, so that it is also indexed with the values above.
To ensure the validator can verify your solution, use "k" and "v" as field names if needed.**

**To complete this lab:**

-   **Modify the following schema to incorporate the above changes:**
~~~    
{
  "_id": "<objectId>",
  "title": "<string>",
  "artist": "<string>",
  "date_acquisition": "<date>",
  "location": "<string>",
  "on_display": "<bool>",
  "in_house": "<bool>",
  "events": [{
    "moma": "<date>",
    "louvres": "<date>"
  }]
}
~~~   
-   **Save your new schema to a file named**  **pattern_attribute.json**.
    
-   **Validate your answer on  _Windows_  by running in the  **CMD**  **shell:****
    ~~~
    validate_m320 pattern_attribute --file pattern_attribute.json
    ~~~
-   **Validate your answer on  _MacOS_  and  _Linux_  by running:**
    ~~~
    ./validate_m320 pattern_attribute --file pattern_attribute.json
    ~~~
**After running this script you will either be given a validation code or an error message indicating what might be missing in your file.
When you get the validation code, paste it in the text box below and click the submit button**

*Answer:*
~~~
{
  "_id": "<objectId>",
  "title": "<string>",
  "artist": "<string>",
  "location": "<string>",
  "on_display": "<bool>",
  "in_house": "<bool>",
  "events": [
    {"k":"<string>", "v":"<date>"},
    {"k":"<string>", "v":"<date>"}
  ]
}
~~~
> Validation Code


#### Extended Reference Pattern
<a href="https://ibb.co/VHR9ZsG"><img src="https://i.ibb.co/pWcw8ST/Screenshot-65.png" alt="Extended Reference Pattern" border="0"></a>
<a href="https://ibb.co/3hZCwYs"><img src="https://i.ibb.co/GPS5X3J/Screenshot-66.png" alt="Managing Duplication" border="0"></a>
<a href="https://ibb.co/fCGvGfD"><img src="https://i.ibb.co/kh616jm/Screenshot-68.png" alt="Extended Reference for Many-to-one relationship" border="0"></a><br /><a target='_blank' href='https://imgbb.com/'></a><br />



#### Subset Pattern
- If working set is too big, then...
	- Add RAM
	- Scale with Sharding
	- Reduce the size of **working set**

-  There are some information that we don't need to use that often in our documents.
- For example, in a movies collection, most of the time users want to see the top actors and the top reviews, rather than all of them.
- As for the fields like comments, quote, and release-- it's also unlikely that you need all of them, most of the time.
- We could keep only 20 of the cast members, the main actors-- and also 20 of each of those comments, quote, release, and reviews.
- The rest of the information can go into a separate collection.
- It means if you start with a document that has all the info in it, a field that has a **one-to-one relationship**-- for example, the full script-- could be moved to a new collection. And you could access this information through the  $lookup operator.
- As for a field that has a **one-to-N** relationship, you can move most of those objects to another collection and keep only a subset of the N relationship in the main document.
- The result on the working set is that each document has been split into the part that is frequently accessed and the part that is rarely accessed.
- Now that the documents are smaller, the whole working set can fit in memory.
- And we have additional memory to bring in the data we only need to rotate.

<a href="https://ibb.co/vk5F1DJ"><img src="https://i.ibb.co/kXn7KMh/Screenshot-69.png" alt="Subset Pattern Recap" border="0"></a>


### Lab: Apply the Subset Pattern
**Problem:
You are the lead developer for an online organic recycled clothing store. Consider the following user story:**

**User Story:**

**Due to the growing number of environmentally-conscious consumers, our store's inventory has increased exponentially. We now also have an increasingly large pool of makers and suppliers.
We recently found that our shopping app is getting slower due to the fact that the frequently-used documents can no longer all fit in RAM. This is happening largely due to having all product reviews, questions, and specs stored in the same document, which grows in size as reviews and ratings keep coming in.
To resolve this issue, we want to reduce the amount of data immediately available to the user in the app and only load additional data when the user asks for it.
Currently a typical document in our data base looks like this:**
~~~
{
  "_id": ObjectId("5c9be463f752ec6b191c3c7e"),
  "item_code": "AS45OPD",
  "name": "Recycled Kicks",
  "maker_brand": "Shoes From The Gutter",
  "price": 100.00,
  "description": "These amazing Kicks are made from recycled plastics and
  fabrics.They come in a variety of sizes and are completely unisex in design.
  If your feet don't like them within the first 30 days, we'll return your
  money no questions asked.
  ",
  "materials": [
    "recycled cotton",
    "recycled plastic",
    "recycled food waste",
  ],
  "country": "Russia",
  "image": "https:///www.shoesfromthegutter.com/kicks/AS45OPD.img",
  "available_sizes": {
      "mens": ["5", "6", "8", "8W", "10", "10W", "11", "11W", "12", "12W"],
      "womens": ["5", "6", "7", "8", "9", "10", "11", "12"]
  },
  "package_weight_kg": 2.00,
  "average_rating": 4.8,
  "reviews": [{
      "author": "i_love_kicks",
      "text": "best shoes ever! comfortable, awesome colors and design!",
      "rating": 5
    },
    {
      "author": "i_know_everything",
      "text": "These shoes are no good because I ordered the wrong size.",
      "rating": 1
    },
    "..."
  ],
  "questions": [{
      "author": "i_love_kicks",
      "text": "Do you guys make baby shoes?",
      "likes": 1223
    },
    {
      "author": "i_know_everything",
      "text": "Why do you make shoes out of garbage?",
      "likes": 0
    },
    "..."
  ],
  "stock_amount": 10000,
  "maker_address": {
    "building_number": 7,
    "street_name": "Turku",
    "city": "Saint-Petersburg",
    "country": "RU",
    "postal_code": 172091
  },
  "makers": ["Ilya Muromets", "Alyosha Popovich", "Ivan Grozniy", "Chelovek Molekula"],
}
~~~
----------

**Task**

**To address this user story, you must modify the following schema so that the working set can fit in RAM:**
~~~
{
  "_id": "<objectId>",
  "item_code": "<string>",
  "name": "<string>",
  "maker_brand": "<string>",
  "price": "<decimal>",
  "description": "<string>",
  "materials": ["<string>"],
  "country": "<string>",
  "image": "<string>",
  "available_sizes": {
    "mens": ["<string>"],
    "womens": ["<string>"]
  },
  "package_weight_kg": "<decimal>",
  "average_rating": "<decimal>",
  "reviews": [{
    "author": "<string>",
    "text": "<string>",
    "rating": "<int>"
  }],
  "questions": [{
    "author": "<string>",
    "text": "<string>",
    "likes": "<int>"
  }],
  "stock_amount": "<int>",
  "maker_address": {
    "building_number": "<string>",
    "street_name": "<string>",
    "city": "<string>",
    "country": "<string>",
    "postal_code": "<string>"
  },
  "makers": ["<string>"]
}
~~~

**You should accomplish this task by completing the following steps:**

-   **Remove the the maker address and the list of makers from the schema with the intention of moving those pieces of information to a separate collection.**
    
-   **Replace the  reviews  and  questions  fields with  top_five_reviews  and  top_five_questions  respectively.**
    
-   **Save your modified schema to a file named  **pattern_subset.json**.**
    
-   **Validate your answer on  _Windows_  by running in the  **CMD**  shell:**
~~~    
validate_m320 pattern_subset --file pattern_subset.json
~~~
 
-   **Validate your answer on  _MacOS_  and  _Linux_  by running:**
~~~    
./validate_m320 pattern_subset --file pattern_subset.json
~~~
**After running this script you will either be given a validation code or an error message indicating what might be missing in your file.
When you get the validation code, paste it in the text box below and click submit.**

**Enter answer here:**
~~~
{
    "_id": "<objectId>",
    "item_code": "<string>",
    "name": "<string>",
    "maker_brand": "<string>",
    "price": "<decimal>",
    "description": "<string>",
    "materials": [
        "<string>"
    ],
    "country": "<string>",
    "image": "<string>",
    "available_sizes": {
        "mens": [
            "<string>"
        ],
        "womens": [
            "<string>"
        ]
    },
    "package_weight_kg": "<decimal>",
    "average_rating": "<decimal>",
    "top_five_reviews": [{
        "author": "<string>",
        "text": "<string>",
        "rating": "<int>"
    }],
    "top_five_questions": [{
        "author": "<string>",
        "text": "<string>",
        "likes": "<int>"
    }],
    "stock_amount": "<int>"
}
~~~
> Validation Code

#### Computed Patterns
- Common Computations/Transformations on Data
	- Mathematical Computations
		- Eg. sum, median, average,etc.
		- We can save performance by calculating result when we get new piece of data. The result is stored and used when next piece of data is recieved .
		- This can save thousands of  computations and reads.
	
	- Fan out Computations
		- Means doing many tasks to represent one logical task.
		- **Fan out on reads**
			-  to return appropriate data the query must fetch data from different locations.
		
		- **Fan out on writes**
			-  Every logical write operation translate to several writes to different documents.
			- In doing so, the read does not have to fan out anymore as data is preorganized at write time.
			- Why to use Fan out on writes?
				- In case system has plenty of time when information arrives compared to acceptable latency of returning data on read operation the  preparing the data at write time makes lot of sense.
				- This pattern is not a good option if system is write bound(Lots of writes)
				- Eg: Social networking system for sharing photos, where system copies a new photo on each followers own page, this way system does not have to spend resources assembling data when user logs in. Everything would be available from a single operation. 
	- Roll-up Operations
		- Merge data together
		- For eg. grouping time based data, grouping in a parent category
		- Often seeing in reporting for hourly, or yearly summaries.
		- When to apply computed patterns?
			- Overuse of resources like CPU
			- Reduce latency for read operations.

<a href="https://ibb.co/qWhC6gr"><img src="https://i.ibb.co/SdHcSm3/Screenshot-71.png" alt="Computed Pattern" border="0"></a>

### Lab: Apply the Computed Pattern

**Problem:
User Story
Our city is going green, and we're reassessing our power plant. A lot of residents are switching to solar panels for their source of electricity and the power plant needs to be both ecologically friendly and flexible in its distribution of electricity throughout the city.
A number of residents are on board with the Green New Deal and are promptly installing solar panels on their homes, which reduces the consumption of electricity from the power plant.
In some cases, the new solar homes produce more energy than they use, thus selling the excess energy back to the power plant.
Our database is tracking the following data about each building in the city: how much energy in kilowatts (kW) per hour it produces (if any), how much energy it consumes, and how much energy needs to be supplemented daily.
In order to make our plant more flexible in adapting to the growing number of solar-powered homes and to project out to the happy date when there won't be a need for a city power plant, our computer algorithms are analyzing consumption data by city zones one day at a time.**
~~~
{
  "_id": ObjectId("5c9414f25e6aff2b8870a2d0"),
  "address": {
    "building number": "742",
    "street name": "Evergreen Terrace",
    "city": "Springfield",
    "state": "MA",
    "zip code": "01111"
  },
  "owner": "Homer J. Simpson",
  "zone": 13,
  "date": ISODate("2019-03-21T23:03:31.197Z"),
  "kW per day": {
    "consumption": 10,
    "self-produced": 7,
    "city-supplemented": 3
  }
}
~~~
**The problem with this design is that we need to calculate zone totals from each building's report every day, which is a lot of unnecessary calculation during reads.
Every zone has anywhere from 100 to 900 buildings and we will create a  **zone**  collection using the consumption data that comes in from every unit in each zone daily.**

----------

**Task**

**To address this issue, we've decided to implement the Computed Pattern. We will create a separate collection that stores zone-based summaries on zone consumption, production, and city-supplemented energy metrics, which we will calculate and overwrite whenever new data comes into the system.
To complete this lab:**

-   **Modify the following schema to represent the schema we will store in our computed collection. Remove any fields we do not need to store in our computed collection:**
    ~~~
    {
      "_id": "<objectId>",
      "address": {
        "building number": "<string>",
        "street name": "<string>",
        "city": "<string>",
        "state": "<string>",
        "zip code": "<string>"
      },
      "owner": "<string>",
      "zone": "<int>",
      "date": "<date>",
      "kW per day": {
        "consumption": "<int>",
        "self-produced": "<int>",
        "city-supplemented": "<int>"
      }
    }
    ~~~
-   **Save the updated schema to a file named  **pattern_computed.json**.**
    
-   **Validate your answer on  _Windows_  by running in the  **CMD**  shell:**
    ~~~
    validate_m320 pattern_computed --file pattern_computed.json
    ~~~
    
-   **Validate your answer on  _MacOS_  and  _Linux_  by running:**
    ~~~
    ./validate_m320 pattern_computed --file pattern_computed.json
    ~~~

**After running this script, you will either be given a validation code or an error message indicating what might be missing in your file.
When you get the validation code paste it in the text box below and click submit.**

**Enter answer here:**
*Answer:*
~~~
{
  "_id": "<objectId>",
  "zone": "<int>",
  "date": "<date>",
  "kW per day": {
    "consumption": "<int>",
    "self-produced": "<int>",
    "city-supplemented": "<int>"
  }
}

~~~
> Validation Key

#### Bucket Pattern
- This pattern is particularly useful in storing data from IoT devices. 
- For e.g. 
	- Let's say we have 10 million temperature sensors, and each sensor is sending us a piece of data every minute, the temperature it's measuring. This is 36 billion pieces of information per hour. Trying to store each piece in a document may not work as we get too many documents to manage.
	- On the other end, if we keep one document per device, each document is likely to reach the maximum size of 16 megabytes after a while.
	- Even if it does not reach that size, it may still be unmanageable and not very efficient to handle these large documents.
	
	<a href="https://ibb.co/bztV482"><img src="https://i.ibb.co/0h5L0dF/Screenshot-72.png" alt="Suggestions" border="0"></a>
- Collaborative Platform 
	- Consider a collaborative platform. The platform will have chat windows that we'll call channels.
	-	One solution is to keep one document per message. However, this may become messy and may require more processing to organize the messages together.
	- An alternative is to keep all the messages for a given chat in a single document. That may work, but over time, each document may become bigger than what we want, as we care more for a recent message than the old one.
	-	Again, the middle solution will be to arrange the messages in the bucket-- All messages for a channel for a given day.
	- One nice thing about this design is that it's very easy to delete documents older than x days, or then archive them.

	- We can use zone sharding and move my old messages to a shard that resides on cheaper hardware.
	<a href="https://ibb.co/m975FyG"><img src="https://i.ibb.co/zXwVH2r/Screenshot-73.png" alt="Collaborative Platform" border="0"></a>
- Column Oriented Data
	- A column-based database stores information per column instead of per row or document.
	- So when you want to process only one column across all the documents, like it is frequently done in analytics, it's very efficient, as it brings in memory only the data it needs.
	- As you know, MongoDB brings in memory full documents.
	- If we have a use case that will benefit a lot from a column-based approach, we could install a separate database next to MongoDB to handle this use case.
	- However, in order to make our deployment model as streamlined as possible, adding different technology for this alone will come with an unnecessary technical complexity.
	If we were to store all the info for a given device in each document, we could query across variables. However, this ability is not needed.
	- But what is really needed is to process all data regarding one field at the time, meaning one column, for example, the intensity of light.
	- We may want to break up the measurement in many documents so processing one type of data does not require bringing everything in memory.
	- When processing light, we bring these documents in memory.
	- When processing pressure, we bring these documents in memory.
	- Note that both sides are using the bucket pattern.
	- It is just that the one on the right has a structure more similar to columns.
	- Also, know that information for the device, like the location and anything else about the device, has been left out of the documents on the right side and put somewhere else.
	- If you need to search or aggregate results on these fields, you should keep them in the documents.
	- Again, this is not going to be as optimal as a column-based database system.
	- However, it's a good work-around if you don't want to install two database systems.

		<a href="https://ibb.co/j59srTJ"><img src="https://i.ibb.co/BPHFCtn/Screenshot-74.png" alt="Column Oriented Data" border="0"></a>
- Summary
					- <a href="https://ibb.co/NCx3hDF"><img 		src="https://i.ibb.co/8sMB3vm/Screenshot-75.png" alt="Bucket Pattern" border="0"></a>
	
### Lab: Apply the Bucket Pattern

**Problem:
In this lab, we will be applying the Bucket Pattern to address an IoT use case scenario.
**Scenario:**
You've been called in to help improve the performance profile of an application that provides a dashboard and reporting information of cell tower quality service metrics.
This application collects a set of metric information sent directly from cell towers.
Each cell tower emits a message similar to the following, in 1 minute intervals:**
~~~
{
  "date": "2019-05-01T17:23:43.042Z",
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "coordinates": [
    -8.5837984085083,
    41.161823159286136
  ],
  "established_calls": 50013,
  "dropped_calls": 1231,
  "data_in_gb": 142,
  "data_out_gb": 481
}
~~~
**The information sent in each message corresponds to:**

-   **the current longitude and latitude of the cell tower -  coordinates**
-   **a unique identifier of the cell tower -  celltower_id**
-   **the date of the measurement -  date**
-   **the following accumulated counters, accumulated since last reboot:**
    -   **the number of established cell phone calls -  established_calls**
    -   **the number of dropped cell phone calls -  dropped_calls**
    -   **the amount of inbound data traffic -  data_in_gb**
    -   **the amount of outbound data traffic -  data_out_gb**

**This system needs to support the following operational requirements:**

-   **Be capable of storing measurements for at least 500 cell towers**
-   **Be able to produce cell tower cumulative reports on one or all four of the accumulated counters, with the following specifications:**
    -   **Each of these reports consists of a plotted graph of the last 24 hours for each metric, with a granularity of 5 minutes**
    -   **The 95 percentile request latency expected for this report is 100ms**

**Current Implementation:**

**The current implementation is to store documents that are very similar to the emitted messages, having all messages stored in a  measurements  collection:**
~~~
db.measurements.find({"cell_tower.id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC"})
{
  "_id": ObjectId("5cd3587395f8fbc3fab3092e"),
  "date_received": ISODate("2019-05-01T17:24:00.000Z"),
  "message_date": ISODate("2019-05-01T17:23:43.042Z"),
  "cell_tower": {
    "location": {
      "type": "Point",
      "coordinates": [
          -8.5837984085083,
           41.161823159286136
      ]
    },
    "id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC"
  },
  "established_calls": 50013,
  "dropped_calls": 1231,
  "data_in_gb": 142,
  "data_out_gb": 481
}
...
~~~
**This implementation is expected to fill out the database servers disk space in one month and there is no budget left for hardware improvements in the next 12 months.**

**The current implementation is unable to produce the reports within the required**  **100ms**.

**Your Solution:**

**In order to resolve the issues that the current approach is facing, you need to come up with a schema design alternative that allows for:**

-   **an increase in the number of IoT devices and associated workload growth**
-   **all report generation to comply with the expected SLA of less than 100ms**
-   **allow for the application to report status for the last 24 hours, with a granularity of 5 minutes**

**Using your pattern knowledge, consider the following three choices for the implementation:**

**A:**
~~~
One document per hour:

{
  "_id": ObjectId("5cd3587395f8fbc3fab3092e"),
  "date": ISODate("2019-05-01T17:00:00.000Z"),
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "established_calls": {
    "minutes": [
      98, 262, 266, 106, 254, 109, 3, 32, 257, 199,
      194, 209, 251, 269, 175, 42, 240, 169, 166, 149,
      238, 43, 128, 119, 120, 134, 267, 87, 228, 56,
      198, 9, 203, 281, 266, 91, 210, 55, 91, 118,
      203, 283, 74, 19, 222, 37, 18, 249, 149, 76,
      165, 29, 44, 94, 277, 253, 79, 100, 182, 127
    ],
    "sum": 9072
  },
  "dropped_calls": {
    "minutes": [
      0, 1, 0, 0, 1, 0, 0, 2, 0, 0,
      0, 1, 0, 0, 0, 0, 0, 0, 0, 0,
      0, 0, 0, 0, 1, 0, 0, 0, 0, 0,
      0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
      0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
      0, 0, 0, 0, 0, 0, 0, 0, 0, 0
    ],
    "sum": 6
  },
  "data_in_gb": {
    "minutes": [
      5, 12, 30, 0, 24, 12, 12, 5, 16, 5,
      4, 18, 6, 2, 13, 7, 11, 2, 8, 30, 25,
      7, 4, 27, 2, 30, 0, 17, 17, 5,
      9, 19, 10, 4, 13, 1, 4, 3, 3, 28,
      12, 8, 1, 21, 6, 4, 29, 23, 3, 16,
      0, 30, 20, 17, 2, 13, 15, 12, 16, 6
    ],
    "sum": 704
  },
  "data_out_gb": {
    "minutes":[
      43, 100, 59, 40, 7, 57, 61, 3, 94, 84, 37,
      67, 25, 80, 40, 34, 8, 20, 69, 66,
      94, 71, 85, 95, 54, 65, 35, 26, 33,
      42, 19, 42, 72, 45, 100, 17, 96, 53, 50, 91,
      34, 79, 45, 34, 51, 96, 90, 5, 12, 30, 50,
      4, 67, 21, 54, 17, 6, 91, 19, 36
    ],
    "sum": 3020
  }
}
~~~
**B:**
~~~
One document per day per metric:

{
  "_id": ObjectId("5cd3587395f8fbc3fab30934"),
  "date": ISODate("2019-05-01"),
  "metric:": "established_calls",
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "measurements": {
    "0": [98, 262, 266, 106, 254, 109, 3, 32, 257, 199, ... ],
    "1": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    "2": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    ...
    "23": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
  },
  "sum": 9072
}
{
  "_id": ObjectId("5cd3587395f8fbc3fab30933"),
  "date": ISODate("2019-05-01"),
  "metric:": "dropped_calls",
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "measurements": {
    "0": [ 0, 1, 0, 0, 1, 0, 0, 2, 0, 0, 0, 1, 0, 0, 0, ... ],
    "1": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    "2": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    ...
    "23": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...]

  },
  "sum": 6
}
{
  "_id": ObjectId("5cd3587395f8fbc3fab30932"),
  "date": ISODate("2019-05-01"),
  "metric:": "data_in_gb",
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "measurements": {
    "0": [ 5, 12, 30, 0, 24, 12, 12, 5, 16, 5, 4, 18, 6 ...],
    "1": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    "2": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    ...
    "23": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
  },
  "sum": 704
}
{
  "_id": ObjectId("5cd3587395f8fbc3fab30931"),
  "date": ISODate("2019-05-01"),
  "metric:": "data_out_gb",
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "measurements": {
    "0":[ 43, 100, 59, 40, 7, 57, 61, 3, 94, 84, 37, 67 ... ],
    "1": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    "2": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
    ...
    "23": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 ...],
  },
  "sum": 3020
}

 ~~~
**C:**
~~~
One document per day:

{
  "_id": ObjectId("5cd3587395f8fbc3fab3092e"),
  "date": ISODate("2019-05-01"),
  "celltower_id": "BBA87930-4A72-4D77-B238-2BA5899C9BEC",
  "established_calls": {
    "0": 9072,
    "1": 0,
    "2": 0,
    ...
    "23": 0,
  },
  "dropped_calls": {
    "0": 6,
    "1": 0,
    "2": 0,
    ...
    "23": 0
  },
  "data_in_gb": {
    "0": 704,
    "1": 0,
    "2": 0,
    ...
    "23": 0,
  },
  "data_out_gb": {
    "0": 3020,
    "1": 0,
    "2": 0,
    ...
    "23": 0,
  }
}
~~~
**Which of the above solutions provide a valid approach for solving the problems experienced by the application?**

**Check all answers that apply:**
- **A**
- **B**
- **C**

*Answer:*
- **A**
- **B**


#### Schema Versioning Pattern

- <a href="https://ibb.co/HG8mxgf"><img src="https://i.ibb.co/CVGxM03/Screenshot-77.png" alt="Application Lifecycle" border="0"></a>
- <a href="https://ibb.co/Q8TJSZk"><img src="https://i.ibb.co/M8tgwbf/Screenshot-79.png" alt="Schema Versioning Pattern" border="0"></a>

### Lab: Apply the Schema Versioning Pattern

**Problem:
Which of the following scenarios are best suited for applying the Schema Versioning Pattern?**

**Scenario A:**
**Your team was assigned to upgrade the current schema with additional fields and transforming the type of different fields without bringing the system down for this upgrade. However, all documents need to be updated to the new shape quickly.**

**Scenario B:**
**The performance of your application became suboptimal over time. Your team has identified that the most commonly used collection could profit from embedding additional information from other collections using the Subset and Computed Patterns. All documents in the commonly-used collection will need to undergo this modification. If possible, you would like the transition to be done without downtime.**

**Scenario C:**
**Your company was bought by its slightly more successful competitor. Thankfully both your and your new owner's applications are flexible enough to handle both document shapes well. You do not have to modify the application or your document shape, but due to the merger, you have to keep documents with different structures in the same collection.**

Check all answers that apply:**

-	Scenario A
-	Scenario B
-	Scenario C

*Answer:*
-	Scenario A
-	Scenario B


#### Tree Pattern

- Patterns in Tree pattern for modelling tree structures:
-
|           Supported Operations           	| Parent References 	| Child References 	| Array of Ancestors 	| Materialized Path 	|
|:----------------------------------------:	|:-----------------:	|:----------------:	|:------------------:	|:-----------------:	|
| What are ancestors of node X             	|         ❕         	|         ❕        	|          ✔         	|         ✔         	|
| Who reports to Y                         	|         ✔         	|         ❕        	|          ✔         	|         ❕         	|
| Find all nodes that are under Z          	|         ❕         	|         ✔        	|          ❕         	|         ❕         	|
| Change all categories under N to under P 	|         ✔         	|         ❕        	|          ❕         	|         ❕         	|

 - ✔ - 	emphasizes that the pattern does support the query.
 - ❕ - 	it is possible to support this query, however, it may take more work in the application code, such as the use of multiple queries.

-	<a href="https://ibb.co/cCpm2x0"><img src="https://i.ibb.co/VQhKM3R/Screenshot-85.png" alt="Tree pattern" border="0"></a>


### Lab: Tree Patterns

**Problem:
In this lab, you will be selecting a variant of the  _Tree Pattern_  to improve the performance of a common set of operations for a given application.**

**Scenario:**

**Given your expertise in schema design, you have been called in to help improve the performance of a human resources management application called  _Success Factions_.
This application needs to be able to map a corporate reporting structure, an organization chart, like the one this diagram represents:**

![https://university-courses.s3.amazonaws.com/M320/mongomart_org_chart.png](https://university-courses.s3.amazonaws.com/M320/mongomart_org_chart.png)

**_Success Factions_  was originally designed by only taking into account the information of a single individual's data in the organization chart. Therefore, documents in the  **employees**  collection, where this data is stored, has the following schema:**
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  }
}
~~~

**Task:**

**You have been tasked to improve the schema design of the  **employees**  collection to better support the following set of operations:**

-   **Issue one database request to find the direct manager of a given employee.**
-   **Collect all direct reports of an employee with one single and  _efficient_  query.**
-   **Issue one update operation to change the reporting structure of an employee.**

**Which of the following document notation examples and tree patterns implement all of these pieces of functionality?**

**Check all answers that apply:
- Parent References**
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  },
  "reports_to": {
    "id": "<objectId>",
    "name": "<string>"
  }
}
~~~
- **Child References**
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  },
  "reports": [
    { "id": "<objectId>", "name": "<string>" }
  ]
}
~~~
- **Materialized Paths**
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  },
  "reports_to": "<string>/<string>/<string>"
}
~~~
- **Array of Ancestors**
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  },
  "reports_to": [
    { "id": "<objectId>", "name": "<string>" }
  ]
}
~~~

*Answer:*
 - **Parent References****
~~~
{
  "_id": "<objectId>",
  "name": "<string>",
  "role": "<string>",
  "department": {
    "name": "<string>",
    "id": "<objectId>"
  },
  "reports_to": {
    "id": "<objectId>",
    "name": "<string>"
  }
}
~~~


#### Polymorphic pattern
- <a href="https://ibb.co/xjkPhmb"><img src="https://i.ibb.co/9TDznw0/Screenshot-86.png" alt="Polymorphic Pattern" border="0"></a>


### Lab: Apply the Polymorphic Pattern

**Problem:**

**User Story**

**Our recruiting agency has just picked up a big international client named Techsy. They want us to take over their recruitment infrastructure for the foreseeable future. Techsy has various community outreach and diversity initiative programs that are focused on recruiting which have to be incorporated into our existing system.**

**Before working with Techsy, most of our recruiting documents looked like this:**
~~~
{
  "_id": ObjectId("5caa9e799c0aa5e39686f803"),
  "first_name": "Arya",
  "last_name": "Stark",
  "engineer_level": 3,
  "education": [{
    "level": "BS",
    "subject": "Computer Science"
  }],
  "years_experience": 3,
  "previous_employer": "Hooli",
  "technical": ["C++", "Python", "Java", "Golang", "MongoDB", "MySQL", "Bash/Shell"],
  "non-technical": {
    "languages": ["English"],
    "other": ["fencing", "dance", "horseback riding", "management"]
  },
  "candidate_notes": "A fierce warrior and survivor. Will do amazingly on any team."
}
~~~
**The following two documents are additional types of documents we are now getting from the Techsy team:**
~~~
[{
  "_id": ObjectId("5caa9e819c0aa5e39686f804"),
  "first_name": "Ginevra",
  "last_name": "Weasley",
  "program_affiliation": "Veteran Outreach Apprenticeship",
  "team_placement": "Server",
  "start_date": ISODate("2017-01-31T00:00:00.000Z"),
  "end_date": ISODate("2018-01-31T00:00:00.000Z"),
  "education": "App Bootcamp",
  "extend_offer": "yes",
  "technical_skills": ["JS", "ReactJS", "NodeJS", "bash/shell", "HTML5", "CSS", "Ruby on Rails", "Python"],
  "non-technical_skills": {
    "languages": ["English"],
    "other": ["Magic"]
  },
  "notes": "Ginny has been an amazing part of the team during the apprenticeship and we would love to extend her an offer to join us more permanently."
}, {
  "_id": ObjectId("5caa9e899c0aa5e39686f805"),
  "name": {
    "first": "Rincewind",
    "last": "TheWizzard"
  },
  "program_affiliation": "Techsy Summer Internship ",
  "team_placement": "Data Optimization",
  "start_date": ISODate("2019-06-01T00:00:00.000Z"),
  "end_date": ISODate("2019-08-31T00:00:00.000Z"),
  "education": [{
      "level": "BA Major",
      "subject": "Philosophy"
    },
    {
      "level": "BS Minor",
      "subject": "Artificial Intelligence"
    }
  ],
  "extend_offer": "yes",
  "skills": {
    "technical": ["R", "Django", "JavaScript", "Matlab", "HTML5", "CSS", "Mathematica", "LaTeX", "Java", "JIRA"],
    "languages": ["English", "Chimeran", "Vanglemesht", "Sumtri", "Black Oroogu", "High Borogravian", "beTrobi"],
    "other": []
  },
  "notes": "Rinecewind has great ideas, but he is a bit of a mess. We still like him a lot and would love to keep him on board."
}]
~~~
**Our recruiters need to be able to search all the candidates available in the system, meaning that we have to add the Techsy records into our  **candidates**  collection. After adding their records, we will need to be able to differentiate between the different document shapes.**

----------

**Task**

**We've decided that the name format for all the varying document shapes can be structured as two separate fields:  first_name  and  last_name. However, the rest of the differences are not that easy to consolidate.
To address this, we will use the Polymorphic Pattern to identify the shape of each document depending on which recruiting source it came from. We will make the following modifications to our current schemas:**

-   **Across all documents, we will use two fields for name:  first_name  and  last_name.**
-   **Across all documents, we will use a field called  recruiting_source  as the indicative field for the document shape.**
-   **In documents that already have an existing  program_affiliation  field, we will use the value of this field as the  recruiting_source.**
-   **In documents that don't have  program_affiliation, we will add a new field called  recruiting_source.**
-   **All documents should indicate whether an offer was extended via the  extend_offer  field.**

**To complete this task, complete the following steps:**

-   **Modify the following schemas to incorporate the previously-stated changes:**
    ~~~
    [{
      "_id": "<objectId>",
      "first_name": "<string>",
      "last_name": "<string>",
      "engineer_level": "<int>",
      "education": [{
        "level": "<string>",
        "subject": "<string>"
      }],
      "years_experience": "<int>",
      "previous_employer": "<string>",
      "technical": ["<string>"],
      "non-technical": {
        "languages": ["<string>"],
        "other": ["<string>"]
      },
      "candidate_notes": "<string>"
    }, {
      "_id": "<objectId>",
      "first_name": "<string>",
      "last_name": "<string>",
      "program_affiliation": "<string>",
      "team_placement": "<string>",
      "start_date": "<date>",
      "end_date": "<date>",
      "education": "<string>",
      "extend_offer": "<string>",
      "technical_skills": ["<string>"],
      "non-technical_skills": {
        "languages": ["<string>"],
        "other": ["<string>"]
      },
      "notes": "<string>"
    }, {
      "_id": "<objectId>",
      "name": {
        "first": "<string>",
        "last": "<string>"
      },
      "program_affiliation": "<string>",
      "team_placement": "<string>",
      "start_date": "<date>",
      "end_date": "<date>",
      "education": [{
          "level": "<string>",
          "subject": "<string>"
        },
        {
          "level": "<string>",
          "subject": "<string>"
        }
      ],
      "extend_offer": "<string>",
      "skills": {
        "technical": ["<string>"],
        "languages": ["<string>"],
        "other": ["<string>"]
      },
      "notes": "<string>"
    }]
    ~~~
-   **Save the updated schemas to a file named  **pattern_polymorphic.json**, with each schema definition separated by a comma and the full solution enclosed in array brackets.**
    
-   **Validate your answer on  _Windows_  by running in the  **CMD**  shell:**
    ~~~
    validate_m320 pattern_polymorphic --file pattern_polymorphic.json
     ~~~
-   **Validate your answer on  _MacOS_  and  _Linux_  by running:**
	~~~    
    ./validate_m320 pattern_polymorphic --file pattern_polymorphic.json
	~~~
**After running this script, you will either be given a validation code or an error message indicating what might be missing in your file.
When you get the validation code, paste it in the text box below and click submit.
Enter answer here:**
> Validation Key
~~~
[{
    "_id": "<objectId>",
    "first_name": "<string>",
    "last_name": "<string>",
    "recruiting_source": "<string>",
    "years_experience": "<int>",
    "previous_employer": "<string>",
    "extend_offer": "<string>",
    "engineer_level": "<int>",
    "education": [{
        "level": "<string>",
        "subject": "<string>"
    }],
    "technical": ["<string>"],
    "non-technical": {
        "languages": ["<string>"],
        "other": ["<string>"]
    },
    "candidate_notes": "<string>"
}, {
    "_id": "<objectId>",
    "first_name": "<string>",
    "last_name": "<string>",
    "recruiting_source": "<string>",
    "extend_offer": "<string>",
    "team_placement": "<string>",
    "start_date": "<date>",
    "end_date": "<date>",
    "education": "<string>",
    "technical_skills": ["<string>"],
    "non-technical_skills": {
        "languages": ["<string>"],
        "other": ["<string>"]
    },
    "notes": "<string>"
}, {
    "_id": "<objectId>",
    "first_name": "<string>",
    "last_name": "<string>",
    "recruiting_source": "<string>",
    "extend_offer": "<string>",
    "team_placement": "<string>",
    "start_date": "<date>",
    "end_date": "<date>",
    "education": [{
        "level": "<string>",
        "subject": "<string>"
    }, {
        "level": "<string>",
        "subject": "<string>"
    }],
    "skills": {
        "technical": ["<string>"],
        "languages": ["<string>"],
        "other": ["<string>"]
    },
    "notes": "<string>"
}]
~~~

#### Other Patterns
##### Approximation Pattern
- Used to reduce no. of resources needed to perform some write operations.
- Avoid performing an  operation often.
- <a href="https://ibb.co/ZMPTWNH"><img src="https://i.ibb.co/hmj8DCX/Screenshot-90.png" alt="Example of approximation Pattern" border="0"></a>
- <a href="https://ibb.co/QdBCmqD"><img src="https://i.ibb.co/5FDnczK/Screenshot-91.png" alt="Approximation Pattern" border="0"></a>

##### Outlier Pattern
- Used to handle potential outliers in the data.
- Keeping focus on the most frequent cases.
- <a href="https://ibb.co/KhF46Bj"><img src="https://i.ibb.co/nmzdr4s/Screenshot-92.png" alt="Outlier Pattern" border="0"></a>
#### Summary of Patterns and potential use cases
- Patterns are powerful transformations for your schema.
- Provide a common language for your team.
- More predictable Methodology.
- <a href="https://ibb.co/TY5Bg33"><img src="https://i.ibb.co/CmF5HGG/Annotation-2020-05-25-222636.png" alt="patterns use cases" border="0"></a>

## Chapter 5: Conclusion
### Final Exam: Question 1

**Problem:
**Scenario**
Consider the following information about the operations on a system:**

**Write Operations**

> ![https://university-courses.s3.amazonaws.com/M320/m320-final-question1-writes-v2.png](https://university-courses.s3.amazonaws.com/M320/m320-final-question1-writes-v2.png)
> 
> -   **ID**  - A unique value to identify each operation.
> -   **Description**  - A summary of the action occuring in the application that triggers this operation.
> -   **Type**  - Additional information about whether this operation is an insert operation or an update operation.
> -   **Durability**  - The number of nodes that must write the data to the database for the operation to be considered complete.
> -   **Data Life**  - The amount of time this record will be stored in the database.
> -   **Data Size**  - The number of bytes being written to the database.
> -   **Storage Size**  - The amount of additional storage needed per day to store all new write data for this operation.
> -   **Average Frequency**  - The average number of times this operation occurs per second.
> -   **Peak Frequency**  - The average number of times this operation occurs per second at peak hour frequency.

**Read operations**

> ![https://university-courses.s3.amazonaws.com/M320/m320-final-question1-reads.png](https://university-courses.s3.amazonaws.com/M320/m320-final-question1-reads.png)
> 
> -   **ID**  - A unique value to identify each operation.
> -   **Description**  - A summary of the action occuring in the application that triggers this operation.
> -   **Type**  - Additional information about whether this operation is a real-time operation or an analytics operation.
> -   **Max Latency**  - The maximum acceptable amount of time for the application to wait to receive data from the database.
> -   **Execution Time**  - The average amount of time the operation takes to complete. This is only filled in if the amount of time is longer than 1 second.
> -   **Single Doc Size**  - The average size of the document being retrieved.
> -   **Average Frequency**  - The average number of times this operation occurs per second.
> -   **Peak Frequency**  - The average number of times this operation occurs per second at peak hour frequency.

**Which of the following operations is the one that should be considered most prominently when designing our schema?**

**Choose the best answer:**

- W4 - application records time and user info when an item is viewed.
- W6 - user adds item to cart.
- R1 - user logs into the application.
- R2 - user views a specific item.
- R4 - user views their cart.

*Answer:*
- R2 - user views a specific item.

### Final Exam: Question 2

**Problem:
**Scenario**
We built a very successful navigation application for cell phones. The application has been installed on many devices throughout the world.
Over a long period of time, the application is active on an average of 10 million cell phones, with a maximum peak of 50 million devices at the busiest time of the year. This average value includes all of the peaks.
Each device, when active, sends 100 bytes of data every minute, which the server writes as one operation.
We want to keep the data for one year.
Based on the above numbers, which of the following are true statements regarding the quantification of this workload and the sizing of the database?
To simplify the calculations and conversions in data units, use:**

-   **1,000,000,000 bytes is 1 gigabyte**
-   **1,000,000,000,000 bytes is 1 terabyte**
-   **1,000,000,000,000,000 bytes is 1 petabyte**

**and round all results to 2 significant digits.**

**Check all answers that apply:**

- The size of the data in the database is 2.6 terabytes.
- The size of the data in the database is 530 terabytes.
- The size of the data in the database is 2.6 petabytes.
- The peak write rate is 17,000 writes/second.
- The peak write rate is 170,000 writes/second.
- The peak write rate is 830,000 writes/second.
- The average write rate is 17,000 writes/second.
- The average write rate is 83,000 writes/second.
- The average write rate is 170,000 writes/second.

*Answer:*
- The size of the data in the database is 530 terabytes.
- The peak write rate is 830,000 writes/second.
- The average write rate is 170,000 writes/second.
### Final Exam: Question 3

**Problem:
Given the following  _Collection Entity Diagram_.**

![https://university-courses.s3.amazonaws.com/M320/product_catalog_relationships-1-1_ref_second.png](https://university-courses.s3.amazonaws.com/M320/product_catalog_relationships-1-1_ref_second.png)

**Which of the following pairs of data have a One-to-One relationship between them?**

**Check all answers that apply:**

- stores.name  and  store_details.staff.name
- stores._id  and  store_details.services
- stores.address  and  store_details.manager.name
- stores.name  and  store_details.description
- stores.name  and  stores.city
- store_details.manager.employee_id  and  store_details.staff.employee_id
- store_details.name  and  store_details.staff.name

*Answer:*
- stores.address  and  store_details.manager.name
- stores.name  and  store_details.description
- stores.name  and  stores.city


### Final Exam: Question 4

**Problem:
We are building a social media site where users can share a lot of details about their whereabouts in their lives.
The site will likely have many European customers. For all these users, the GDPR law in Europe cites that a person has the right to be forgotten, meaning all data related to that person should be erased per request of the user.
Which one of the following implementations would be the preferred way to represent the One-to-Many relationship between restaurants and each user's last-visited restaurant to enable easy removal of a user's activity in accordance with GDPR?**

**Choose the best answer:
- Only a  **restaurants**  collection in which all user information is embedded.**	![https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-2.png](https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-2.png)

- **Only a  **users**  collection in which all restaurant information is embedded.**
![https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-4.png](https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-4.png)

- **A  **users**  collection with an extended reference to a  **restaurants**  document.**
![https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-1.png](https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-1.png)

- **A  **restaurants**  collection with references to  **users**  documents.**
![https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-3.png](https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-3.png)
-----------
*Answer:*
- **A  **users**  collection with an extended reference to a  **restaurants**  document.**
![https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-1.png](https://university-courses.s3.amazonaws.com/M320/m320-final-one-to-many-1.png)


### Final Exam: Question 5

**Problem:
Which of the following are recommended ways to model a Many-to-Many relationship between airlines and the cities those airlines fly between?**

**Check all answers that apply:**
- Embedding the airlines that fly to/from a city in each of the corresponding  **cities**  document, without having a separate collection for the airlines.**
![https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-3.png](https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-3.png)

- Referencing the  **airlines**  documents in each of the corresponding  **cities**  documents.
![https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-2.png](https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-2.png)

- Embedding the airlines that fly to/from a city in each of the corresponding  **cities**  document and keeping a separate copy of the  **airlines**  documents.
![https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-1.png](https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-1.png)
--------
*Answer:*
- Referencing the  **airlines**  documents in each of the corresponding  **cities**  documents.
![https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-2.png](https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-2.png)

- Embedding the airlines that fly to/from a city in each of the corresponding  **cities**  document and keeping a separate copy of the  **airlines**  documents.
![https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-1.png](https://university-courses.s3.amazonaws.com/M320/m320-final-many-to-many-1.png)


### Final Exam: Question 6

**Problem:
Consider a system that collects census data on all countries in the world.
Which of the following implementations are recommended ways to model a One-to-Zillions relationship between a person entity and the country entity in which that person was born?**

**Check all answers that apply:**

- Referencing from the Zillions side.
	- In each person document, we reference the corresponding country document.

- Embedding as an array.
	- In the document for a country, we embed all the people born in that country.

- Referencing from the One side.

	- In the document for a country, we have an array of references to the documents of the people who are kept in a separate collection.

*Answer:*
- Referencing from the Zillions side.
	- In each person document, we reference the corresponding country document.



### Final Exam: Question 7

**Problem:
**Scenario**
Your team just hired a new Data Architect, and her amazing ideas are gaining traction with the company leadership.
You already updated your application to be able to handle the new data organization in your database. Now you have been tasked with implementing her proposed new data organization approach to your database with minimum downtime for the users of the application.
Which pattern solution is best suited for this situation?**

**Choose the best answer:**

- The Attribute Pattern
- The Bucket Pattern
- The Subset Pattern
- The Extended Reference Pattern
- The Schema Versioning Pattern

*Answer:*
- The Schema Versioning Pattern




### Final Exam: Question 8

**Problem:
**Scenario**
A new decision maker came aboard your online bookstore team. They want to be able to track which genres are most popular daily. To keep this metric up to date without running massive queries for obtaining it, which of the following schema patterns would you choose to implement?**

**Choose the best answer:**

- The Polymorphic Pattern
- The Computed Pattern
- The Extended Reference Pattern
- The Bucket Pattern
- The Subset Pattern

*Answer:*
- The Computed Pattern





### Final Exam: Question 9

**Problem:
**Scenario**
You work as a developer at a factory. Your factory wants to track the usage statistics of the automatic lighting that was recently installed throughout its facilities. The lights send an update to the database every 10 seconds, but the management is interested in an hourly report instead. Additionally, we are only looking to store this information for at most 5 years, so an easy way to purge old data would be beneficial to our data modeling approach.
Which pattern solution is best suited for this situation?**

**Choose the best answer:**

- The Bucket Pattern
- The Computed Pattern
- The Extended Reference Pattern
- The Polymorphic Pattern
- The Subset Pattern

*Answer:*
 - The Bucket Pattern



### Final Exam: Question 10

**Problem:
**Scenario**
With the digitization of every area of our lives, the famous NYC bodegas (convenience stores) are trying to keep up. Bodegas don't just know everything that goes on in the neighborhood, they also supply all types of household goods, hardware supplies, and groceries. The New York City Bodega Association is looking to create an app that will help them keep track of their unique, versatile inventory and help customers look up whether items are in stock before visiting the bodega.
Which pattern solution is best suited for this application?**

**Choose the best answer:**

- The Polymorphic Pattern
- The Extended Reference Pattern
- The Bucket Pattern
- The Computed Pattern
- The Subset Pattern

*Answer:*
- The Polymorphic Pattern


### Final Exam: Question 11

**Problem:
**Scenario**
You are a developer working on an e-commerce application. Each time your application retrieves an item from the  **inventory**  collection, it also needs to retrieve data about the orders in which this item is present. This leads to additional queries on the  **orders**  collection.
We know that reducing the total number of queries we perform on the system would solve the main performance issue we are seeing at peak time.
Which pattern solution is best suited for this situation?**

**Choose the best answer:**

- The Extended Reference Pattern
- The Polymorphic Pattern
- The Schema Versioning Pattern
- The Bucket Pattern
- The Computed Pattern

*Answer:*
- The Extended Reference Pattern



### Final Exam: Question 12

**Problem:
**Scenario**
As a chemical manufacturer, you tend to keep your factory and your data organized, since dealing with chemicals requires a lot of precision, attention, and safety mechanisms. One of the safety mechanisms in the factory is the documentation about produced chemicals. This documentation is recorded in  **Material Safety Data Sheets**  which are large pdf documents containing safety details about a given chemical. These Data Sheets are part of the documents in the  **inventory**  collection, where other information such as the price, quantity, and warehouse location of the chemical is stored as well.
Keeping track of production, sales, and purchases requires a lot of data manipulation on an hourly basis. You notice that at especially busy times, your inventory tracking application slows down by a lot.
Which pattern solution is best suited for solving this issue?**

**Choose the best answer:**

- The Polymorphic Pattern
- The Subset Pattern
- The Bucket Pattern
- The Computed Pattern
- The Extended Reference Pattern


*Answer:*
- The Subset Pattern
