# M320: Data Modeling
## Chapter 1: Introduction to Data Modelling
### Important Points
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

