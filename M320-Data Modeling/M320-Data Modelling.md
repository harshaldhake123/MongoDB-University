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
