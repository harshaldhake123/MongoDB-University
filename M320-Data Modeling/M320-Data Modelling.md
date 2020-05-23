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
