## M201 - MongoDB Performance

- [Chapter 1 - Introduction](#chapter-1---introduction)
  * [Points to Remember](#points-to-remember)
  * [Lab 1.1 - Install Course Tools and Datasets](#lab-11---install-course-tools-and-datasets)
- [Chapter 2 - MongoDB Indexes](#chapter-2---mongodb-indexes)
  * [Points to Remember](#points-to-remember-1)
  * [Lab 2.1 - Using Indexes to Sort](#lab-21---using-indexes-to-sort)
  * [Lab 2.2 - Optimizing Compound Indexes](#lab-22---optimizing-compound-indexes)
- [Chapter 3 - Index Operations](#chapter-3---index-operations)
  * [Points to Remember](#points-to-remember-2)
  * [Lab 3.1 - Explain Output](#lab-31---explain-output)
- [Chapter 4 - CRUD Optimization](#chapter-4---crud-optimization)
  * [Points to Remember](#points-to-remember-3)
  * [Lab 4.1 - Equality, Sort, Range](#lab-41---equality--sort--range)
  * [Lab 4.2 - Aggregation Performance](#lab-42---aggregation-performance)
- [Chapter 5 - Performance on Clusters](#chapter-5---performance-on-clusters)
  * [Points to Remember](#points-to-remember-4)
- [Final Exam](#final-exam)
  * [Final - Question 1](#final---question-1)
  * [Final - Question 2](#final---question-2)
  * [Final - Question 3](#final---question-3)
  * [Final - Question 4](#final---question-4)
  * [Final - Question 5](#final---question-5)
  * [Final - Question 6](#final---question-6)
  * [Final - Question 7](#final---question-7)

## Chapter 1 - Introduction
### Points to Remember
- MongoDB has search engines that are either very dependent on RAM or completely in memory execution modes.
	- Operations that rely heavily on RAM:
		- Aggregation
		- Index Traversing
		- Write Operations
		- Query Engine
		- Connections(roughly 1mb/conn.)

	- Operations where CPU is utilized:
		- Storage Engine
		- Concurrency Model
		- Page compression
		- Data calculation
		- Aggregation Framework Operations
		- Map Reduce

	-Writing constantly to same document will require each write to block all other writes on same document to comply. In such cases multiple CPUs do not help in performance because threads can't do their work in parallel.


### Lab 1.1 - Install Course Tools and Datasets

**Problem: 
Welcome to your first lab in M201!
In this lab, you will install some tools and load some of the datasets we will use in the course.**
**You can follow the lessons and do the labs using either:**
-  Your private Atlas cluster
-  A local installation of MongoDB on your machine

**Verification:
To confirm that you've successfully completed the required steps to follow the class and do the labs, run the following query on the  **m201**  database from the  **mongo shell**  and paste its output into the submission area below:**

     use m201
     db.people.count({ "email" : {"$exists": 1} })

**Enter answer here:**
*50474*



## Chapter 2 - MongoDB Indexes
### Points to Remember
- Data Storage on Disk:
	- Indexes in MongoDB are stored in b-tree that can be used to find target values with very few comparisons.
	- We can get awesome performance gain using indexes however, this comes with some drawbacks. 
	- With each additional index. write speed is decreased.
	- Also, for each write/update document, one or more b-trees may need to be balanced. Therefore , unnecessary indexes should be avoided.
	- We can specify an index that uses dot notation to index on the fields of our embedded document. In this way, when we use dot notation in our queries, we'll still be able to use an index.
	- We should never index on the field that points to a sub-document. Because doing so, we'd have to query on the entire sub-document.

	- It's much better to use dot notation when querying because we can just query on the fields that we care about in our sub-documents.
	- If you do need to index on more than one field, you can use a compound index.
- Single Field Index:
	- We can use a single field index to query on a single value or to query on a range of values.
	- We can also use single field index for dot notation when using subdocuments.
- Explain 
	- If we ever have a sort that uses more than **32 megabytes**, then the server is going to cancel that query.
	- **Explain** Plan helps to troubleshoot slow queries and optimize performance of queries.
	
	- Sorting using Indexes: 
	- We can sort our queries by using **index prefixes** in our sort predicates.
	- We can filter and sort our queries by splitting up our index prefix between the query and sort predicates.
	- We can sort our documents with an index if our sort predicates inverts our index keys or their on of their predicates.
- Multikey Indexes:
	- For each index document, we can have **at most one** index field whose value is an array.
	
	- If we do create a compound index on two array valued fields, then the index keys will be  cartesian product between the length of both the index fields. This impacts performance of query due to a large no. of indexes.
	- Multikey indexes don't support covered queries.
- Partial Indexes:
	- options:
		- sparse
		- partialFilterExpression
	- With a **sparse index**, we only index documents where the field exists that we're indexing on, rather than creating an index key with a null value.
		- We can achieve the same effect by creating a **partial index** where the filter expression checks for the existence of the field were indexing on.
	- In order to use a partial index, the query must be guaranteed to match a subset of the documents, specified by the filter expression. This is because the server could miss results in the case where matching documents are not indexed.

	- In order for us to trigger an index scan, we need to include the stars predicate in our query that matches our filter expression.
	- Your query predicate must be guaranteed to match a subset of the documents that will match our filter expression.
	- Partial Index Restrictions:
		- we cannot specify both a partialFilterExpression and the sparse option(we can actually make a sparse query with a filter expression).
		
		- We also can't make our _ID index partial, because every document has to have an indexed _ID field.
		- We cannot have our shard key be a partial index.
- Text Indexes:
	- Regular expressions don't have very good performance even with indexes.
	- Therefore text indexes can help in this case. It works similar to multi-key Index fields. Separate index key will be created for each word for that field which are identified by spaces, hyphens as text delimiters.
	- By default, text indexes are case insensitive.
	- Text indexes on large text fields result in more keys to examine, increased index size , increased time to build index and decreased write performance.

	- One strategy to reduce the no. of index keys is to create a **compound index**.
	- **$search** ORs each keyword to be searched. **\$meta:"textScore"** can be used to assign a score to each document based on the relevance to search query.
	~~~
	// Sort the documents by their textScore so that the most relevant documents  
	db.textExample.find({ $text: { $search : "MongoDB best" } }, { score: { $meta: "textScore" } }).sort({ score: { $meta: "textScore" } })
	~~~

- Collations
	- Collations allow users to specify language specific rules for string comparison, such as rules for letter, case, and accent marks.
		 
	 - Can specify a different collation on a index creation. For index, it will override default and collection level collations.
	~~~
	 db.foreign_text.find({ _id: {$exists:1 } }).collation({locale: 'it'})
	 db.foreign_text.aggregate([ {$match: { _id: {$exists:1 }  }}], {collation: {locale: 'es'}})
	 db.foreign_text.createIndex( {name: 1},  {collation: {locale: 'it'}} )
	~~~
	- In order for index to be used by a query, query must match collation of index.
	~~~
	// uses the collection collation (Portuguese)
	db.foreign_text.find( {name: 'Máximo'}).explain()

	// uses the index collation (Italian)
	db.foreign_text.find( {name: 'Máximo'}).collation({locale: 'it'}).explain()
	~~~
	**Collation Properties**

	-   Needed for correctness of text searching
	-   Marginal performance impact
	-   Case insensitive indexes

	 Set  *strength*  to 1 for primary level of comparison (i.e. case insensitive index):

	    db.createCollection( "no_sensitivity", {collation: {locale: 'en', strength: 1}})

	In this case sorting by some text field where docs contain values like  `aaa`  and/or  `AAA`,  `aAa`  etc, will sort the same whether ascending or descending.

- WildCard Indexes:
	- Wildcard indexes are a new index type in MongoDB, which allow you to dynamically create indexes on all fields or a selected subset of fields for each document in a collection.

	- Wildcard indexes **aren't a replacement** for traditional indexes.

	- Indexes are great for query performance, but come at a cost. Each index needs to be maintained. And every document that's written to the database needs to update all the relevant indexes as well.

	- Some workloads have unpredictable access patterns.
		- For eg: sensor data for IoT use cases or weather stations.
	- In such cases, each query may include a combination of an arbitrarily large number of different fields.
	- This can make it very difficult to plan an effective indexing strategy.
	- For these workloads, we needed a way to be able to index on multiple fields without the overhead of maintaining multiple indexes.
	- MongoDB indexes any scalar values associated with a specified path or paths.
	- For fields that are documents, MongoDB descends into the document and creates index keys for each field value pair it finds.
	- For fields that are arrays, MongoDB creates an index key for each value in the array.
	- This is similar to how multi-key indexes handle arrays at the moment.
	- If the array contains sub-documents, MongoDB, again, will descend through those documents and index all field value pairs.
	- If we use a wildcard index, we can assume that certain indexes exist without the need to manually create them.
	- **$\*\*** is a wildcard operator, which tells MongoDB to index everything in the collection.
	- Wildcard index will generate one virtual single field index at query execution time and then the planner will assess them using the standard query plan score.
	
	- WildCard Query Syntax:
		~~~
		db.coll.createIndex({"$**":1})								//Index Everything
		db.coll.createIndex({"a.b.$**":1})							//Index 'a.b' and all subpaths
		db.coll.createIndex({"$**":1},{wildcardProjection:{a:1}})	//Index 'a' and all subpaths
		db.coll.createIndex({"$**":1},{wildcardProjection:{a:0}})	//Index everything but 'a'
		~~~
		
- Covered Query:
	- Wildcard indexes can support a  covered query **only if**  all of the following are true:
		
		-   The query planner selects the wildcard index for satisfying the query predicate.
		-   The query predicate specifies  _exactly_  one field covered by the wildcard index.
		-   The projection explicitly excludes  `_id`  and includes  _only_  the query field.
		-   The specified query field is never an array. 


### Lab 2.1 - Using Indexes to Sort
**Problem:
In this lab you're going to determine which queries are able to successfully use a given index for both filtering and sorting.**

**Given the following index:**
~~~
{ "first_name": 1, "address.state": -1, "address.city": -1, "ssn": 1 }
~~~
**Which of the following queries are able to use it for both filtering and sorting?**
**Check all answers that apply:**
~~~
db.people.find({ "first_name": { $gt: "J" }  }).sort({  "address.city":  -1  })
~~~
~~~
db.people.find({ "address.city": "West Cindy"  }).sort({  "address.city":  -1  })
~~~
~~~
db.people.find({ "address.state": "South Dakota", "first_name": "Jessica"  }).sort({  "address.city":  -1  })
~~~
~~~
db.people.find({ "first_name": "Jessica"  }).sort({  "address.state": 1, "address.city": 1 })
~~~
~~~
db.people.find({ "first_name": "Jessica", "address.state": { $lt: "S"}  }).sort({  "address.state": 1 })
~~~
*Answer:*
- The order of the fields in the query predicate does not matter.
~~~ 
db.people.find({ "address.state": "South Dakota", "first_name": "Jessica"  }).sort({  "address.city":  -1  })
~~~
~~~
db.people.find({ "first_name": "Jessica"  }).sort({  "address.state": 1, "address.city": 1 })
~~~
~~~
db.people.find({ "first_name": "Jessica", "address.state": { $lt: "S"}  }).sort({  "address.state": 1 })
~~~
### Lab 2.2 - Optimizing Compound Indexes

**Problem:
In this lab you're going to examine several example queries and determine which compound index will best service them.**
~~~
> db.people.find({
    "address.state": "Nebraska",
    "last_name": /^G/,
    "job": "Police officer"
  })
~~~
~~~
> db.people.find({
    "job": /^P/,
    "first_name": /^C/,
    "address.state": "Indiana"
  }).sort({ "last_name": 1 })
~~~
~~~
> db.people.find({
    "address.state": "Connecticut",
    "birthday": {
      "$gte": ISODate("2010-01-01T00:00:00.000Z"),
      "$lt": ISODate("2011-01-01T00:00:00.000Z")
    }
  })
~~~
**If you had to build one index on the  **people**  collection, which of the following indexes would best sevice all 3 queries?**

**Choose the best answer:**

~~~
{ "job": 1, "address.state": 1 }
~~~
~~~
{ "job": 1, "address.state": 1, "first_name": 1 }
~~~
~~~
{ "job": 1, "address.state": 1, "last_name": 1 }
~~~
~~~
{ "address.state": 1, "job": 1, "first_name": 1 }
~~~
~~~
{ "address.state": 1, "last_name": 1, "job": 1 }
~~~
~~~
{ "address.state": 1, "job": 1 }
~~~

*Answer:*
~~~
{ "address.state": 1, "last_name": 1, "job": 1 }
~~~

## Chapter 3 - Index Operations
### Points to Remember
- Building Indexes:
	- **Foreground index build**, which was the most performant but had the side effect of locking the entire database for the duration of the index build. This meant that you could **neither read from or write to** the database for the duration of the index build.

	- **Background index builds**,  don't hold a lock on a database but aren't as performant as a foreground index build. The background index build uses an incremental approach that is slower than the foreground index build. Background index build will periodically lock the database but will yield to incoming read and write operations, releasing resources to attend to incoming requests. If the index is larger than the available RAM, the incremental approach can take much longer than a foreground index build. The other downside of background indexes is that the index structure resulting from this type of index build is less efficient than foreground indexes, resulting in less optimal index traversal operations.
	- The new **Hybrid index build** has both the performance of a foreground index build and the nonlocking properties of a background index build, meaning that all database operations can proceed uninhibited for the duration of the build.
		- New in MongoDB 4.2
		- This is now the only way to build an index on MongoDB.
		- Index structure is unchanged.
		- Allows database operations to continue as normal for the duration of the index build.

- **Query Plan**: A series of stages that feed into one another.
- Steps in selecting a Query Plan:
	1. When a fresh query comes into the database for the first time, the server is going to look at all the available indexes on the collection.

	2. From there, it will identify which indexes are viable to satisfy the query. (Candidate indexes).

	3.  From these candidate indexes, the query optimizer can generate candidate plans.

	4. MongoDB has an **empirical query planner**,each of the candidate plans is executed over a short period of time in the trial period and the planner will then see which plan performed best.
 
- Query Plan Eviction from cache:
	- Reasons:
		- If the server is restarted.

		- If the amount of work performed by the first portion of the query exceeds the amount of work performed by the winning plan by a factor of 10.

		- When the indexes are rebuilt.
		- If an index is created or dropped.

- Forcing Indexes with hint():
	- **hint()** overrides the query planner's default index selection to the one specified  in the parenthesis.
	- It should be used with caution because the query planner does a very good job selecting a index itself.
	- It may be used when there are a lot of indexes  on a collection but it is better to just remove any superfluous, unnecessary indexes rather than using hint().

- Resource Allocation for Indexes:
	- Indexes need two basic computational resources:
		1. Disk - to store index information
		2. Memory - to operate with data structures.
	- Determining Index Size

	- Compass displays index size for each collection in a database. Selecting a particular collection will break down index size for each index on that collection. Index size can also be fetched using  `db.stats()`.
	- After an  index created, disk space requirement is a function of data. However, if we  are operating with **separate disks** for indexes and collection, we do need to ensure that disk allocated for indexes is large enough.
	
	- Memory is most intensive resource utilization for index usage. Deployments should be sized to accommodate all indexes in RAM.

	- If there isn't enough space in RAM for index, a lot of disk access will be required to traverse index. Different pages from index on disk will be allocated to memory. As you traverse that space, will slide pages into positions that are no longer in memory, therefore need to allocate those to memory and flush out info to disk.
	- If constantly traversing index then lots of page in/out will take place. 

	- We can specify WT(WiredTiger) cache size at mongo startup:
	~~~
	mongod --dbpath data --wiredTigerCacheSizeGB 1
	~~~
	- Then even though mem on machine is 2G, amount allocated to 				Mongo will only be 1G.
	- To find percentage of index that is cached in memory, 
	~~~
	db.people.stats({indexDetails: true})
	~~~
	- This returns a lot of information, we can look at *cache* entry in *indexDetails* for each index.
	~~~
	"cache" : {
		"bytes currently in the cache" : 3568, 	// how much of index is in RAM
		"bytes read into cache" : 1160,
		...
		"pages read into cache" : 3,						// use to determine hit and pass page ratios
		"pages requested from the cache" : 2,		// use to determine hit and pass page ratios
		...
	~~~
	- `pages requested from the cache` means queries were run using the index and it was traversed, therefore mongo read pages from RAM.

- Edge Cases:
	- Occasional Reports
		- If a BI report doesn't run very often, don't necessarily need entire index that supports it in memory.

		- To mitigate this, don't run query against primaries in replica sets and do not create index on primary. Rather, use secondaries.
	- Right-hand-side index increments
		- Monotonically incrementing fields will cause index to become unbalanced on the right hand side therefore ,it will  grow with new data incoming.
		- If we only need to query on the most recent data, then we only need the most recent right-hand portion in memory, not the whole thing because left side and upper right of it contains older data.

- Basic Benchmarking:
	- Types:
		- Public Test Suite
		- Specific/Private Test Environment
	- **Low Level Benchmarking**

		-   File I/O performance
		-   Scheduler performance
		-   Memory allocation and transfer speed
		-   Thread performance
		-   Database server performance
		-   Transaction isolation

	- **Database Server Benchmarking**

		-   Data set load
		-   Writes per second
		-   Reads per second
		-   Balanced workloads
		-   Read / Write ratio

	- **Distributed Systems Benchmarking**

		Most relevant for Mongo.

		-   Linearization
		-   Serialization
		-   Fault tolerance
		
		Tools: jepson.io, iBench, etc.
- Benchmarking Conditions

	- Most tooling was built for relational db's, not necessarily relevant to MongoDB.
	- Tooling may not capture app-specific use cases.

		-   Hardware
		-   Clients
		-   Load

- **Benchmarking Anti-Patterns**

	-   Database swap replace (comparing mongo to relational db using exact same schema)
	-   Using mongo shell for write and read requests (not reflective of typical app usage)
	-   Using mongoimport to test write response
	-   Local laptop to run tests (use a server)
	-   Using default MongoDB parameters (use production settings - eg authentication, high availability)

### Lab 3.1 - Explain Output

**Problem:
In this lab you're going to determine which index was used to satisfy a query given its explain output.
The following query was ran:**
~~~
> var exp = db.restaurants.explain("executionStats")
> exp.find({ "address.state": "NY", stars: { $gt: 3, $lt: 4 } }).sort({ name: 1 }).hint(REDACTED)
~~~
**Which resulted in the following output:**
~~~
{
  "queryPlanner": {
    "plannerVersion": 1,
    "namespace": "m201.restaurants",
    "indexFilterSet": false,
    "parsedQuery": "REDACTED",
    "winningPlan": {
      "stage": "SORT",
      "sortPattern": {
        "name": 1
      },
      "inputStage": {
        "stage": "SORT_KEY_GENERATOR",
        "inputStage": {
          "stage": "FETCH",
          "inputStage": {
            "stage": "IXSCAN",
            "keyPattern": "REDACTED",
            "indexName": "REDACTED",
            "isMultiKey": false,
            "isUnique": false,
            "isSparse": false,
            "isPartial": false,
            "indexVersion": 1,
            "direction": "forward",
            "indexBounds": "REDACTED"
          }
        }
      }
    },
    "rejectedPlans": [ ]
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 3335,
    "executionTimeMillis": 20,
    "totalKeysExamined": 3335,
    "totalDocsExamined": 3335,
    "executionStages": "REDACTED"
  },
  "serverInfo": "REDACTED",
  "ok": 1
}
~~~
**Given the redacted explain output above, select the index that was passed to hint.**

**Choose the best answer:**
~~~
{ "address.state": 1, "name": 1, "stars": 1 }
~~~
~~~
{ "address.state": 1 }
~~~
~~~
{ "address.state": 1, "stars": 1, "name": 1 }
~~~
~~~
{ "address.state": 1, "name": 1 }
~~~

*Answer:*
~~~
{ "address.state": 1, "stars": 1, "name": 1 }
~~~

## Chapter 4 - CRUD Optimization
### Points to Remember
- Equality, Sort, Range Thumb rule:
	- At the beginning of our index, we should match on **equality** conditions in the query predicate, followed by **sort** conditions, and finally **range** conditions; equality, sort, range.
- Covered Queries:
	
	- Highly performant
	- Satisfied entierly by index keys
	- 0 documents need to be examined
	- Restrictions:
		- We cannot run a index key by explicitly excluding some fields because mongodb does not know what fields to project so it has to scan the documents anyway.
		
		- Also, you can't cover a query if:
			- Any of indexes fields are arrays
			- Any of indexed fields are embedded documents
			- When run against mongos if the index does not contain the shard key.
- Regex Performance:
	- Regex in our query can perform well with some conditions applied.
	- That is, We should have an index on the field to be queried  and regex should be structured where the following characters start at the beginning of the string. 
	
	- This optimization doesnt work if we specify a wildcard regex like: */^.kirby/* because mongo still will have to look for all the index btree branches to search for this regex pattern.

- Aggregation Performance	
	 
	'Realtime' Processed Queries | Batch Processed Queries
	-------- | -----
	 can never have truly real time processing. | Job is generally periodic.
	 Performance is important| Performance not important
	 To provide data to applications| aggregation for analytics 


	- Data moves through pipeline from the first operator to the last. Once the server encounters a stage that is not able to use indexes, all of the following stages will no longer be able to use indexes either.
	- Query optimizer tries its best to detect when a stage can be moved forward so that indexes can be utilized.
	- There's a 100 megabyte limit of RAM usage. You can get around this limit by using *allowDiskUse : true* but it is not recommended as it affects performance critically.
	
	- Output of an  aggregation pipeline has a size limit of 16megabytes. However, this limit does not apply to documents as they flow through the pipeline.


### Lab 4.1 - Equality, Sort, Range

**Problem:
In this lab you're going to use the equality, sort, range rule to determine which index best supports a given query.
Given the following query:**

    db.accounts.find( { accountBalance : { $gte : NumberDecimal(100000.00) }, city: "New York" } )
               .sort( { lastName: 1, firstName: 1 } )

**Which of the following indexes best supports this query with regards to the equality, sort, range rule.**

**Choose the best answer:**
~~~
{ lastName: 1, firstName: 1, city: 1, accountBalance: 1 }
~~~
~~~
{ lastName: 1, firstName: 1, accountBalance: 1, city: 1 }
~~~
~~~
{ accountBalance: 1, city: 1, lastName: 1, firstName: 1 }
~~~
~~~
{ city: 1, lastName: 1, firstName: 1, accountBalance: 1 }
~~~
*Answer:*
~~~
{ city: 1, lastName: 1, firstName: 1, accountBalance: 1 }
~~~
### Lab 4.2 - Aggregation Performance

**Problem:
For this lab, you're going to create an index so that the following aggregation query can be executed successfully.
We assume you have imported the  _restaurants_  dataset in your cluster. If not, it is attached to this lesson, so you can import it following the instructions in the  _Install Course Tools and Datasets_  lesson in Chapter 1.
Before you work on your solution, delete all indexes for the  _restaurants_  dataset:**
~~~
db.restaurants.dropIndexes()
~~~
**Then, run the following query and you will receive an error.**
~~~
db.restaurants.aggregate([
  { $match: { stars: { $gt: 2 } } },
  { $sort: { stars: 1 } },
  { $group: { _id: "$cuisine", count: { $sum: 1 } } }
])
~~~
~~~
{
  "ok": 0,
  "errmsg": "Sort exceeded memory limit of 104857600 bytes, but did not opt in to external sorting. Aborting operation. Pass allowDiskUse:true to opt in.",
  "code": 16819,
  "codeName": "Location16819"
}
~~~
**The task of this lab is to identify why this error is occuring and build an index to resolve the issue.
Keep in mind that there might be several indexes that resolve this error, but we're looking for an index with the smallest number of fields that resolves the error and services this command.**

**For example, if you ran:**
~~~
db.restaurants.createIndex( { foobar: 1 } )
~~~
**to create the index that fixes the error, then you would enter the following document which lists the fields indexed into the text box:**

    { foobar: 1 }

*Answer:*

> { stars : 1}


## Chapter 5 - Performance on Clusters
### Points to Remember
- Performance Considerations in Distributed Systems:
	- Distributed systems in mongo:
		- **Replica Cluster**
		- **Shard Cluster** (horizontal scalability of data)
		
	- Considerations when more than one machine is involved:
		-   Latency
		-   Data is spread across different nodes (copies of data or different sets of data in different shards)
		-   Read implications
		-   Write implications
	- It is recommended to **use Replica Sets in Production.**
	- Benefits of replica set include:
		- Higher availability
		-   offloading eventual consistency data to secondaries, freeing up primary for operational workload
		-   having workload configuration where indexes are on secondary nodes

	- **Sharded cluster**
		- Multiple mongos instances are responsible for routing client application requests to designated nodes.
		-   Config servers contain mapping of shard cluster - where data sits, and configuration of shard cluster.
		-   Shard nodes contain application data (databases, collections, indexes)
		-  major workloads performed here
		-   Shard nodes are also replica sets

		- Considerations before sharding:

			-   Have we reached limits of vertical scaling?
			-   Understand how your data grows and how your data is accessed
			-   Works by defining key based ranges - shard key
			-   It is important to get a good shard key

		- Sources of Latency in a Shard Cluster
			-   Client app talks to mongos(s).
			-   Mongos(s) establish communications with config server to retrieve config information about shard, and with shard nodes to get app-specific data.
			-   Some latency is also caused by replication mechanism within each shard node.

	- How to minimize latency
		-   Co-locate mongos within same server as client application to minimize number of network hops to access shard nodes.
		-   Ensure high bandwidth network connection between mongos and shard nodes.

	- Types of reads in shard cluster

		-   **Scatter Gather**: 
			- Ping all nodes of shard cluster for info corresponding to a given query.
			- pay the latency price for asking every single shard node for data.
			- If not using shard key, then it is forced to do Scatter Gather because mongo cannot determine which shard node has data needed to satisfy client query.
		-   **Routed Queries**: 
			- Ask a single shard node, or small amount of shard nodes for data requested by application.
			- less latency because only talking to one or few shard nodes.
			- Routed Query is possible when using shard key in queries.  
			- When shard key is used, mongos knows exactly which shard(s) contain data relevant for this query.

	- Sorting in shard cluster
		
		- Sorting involves a few hurdles, but transparent to client application. Mongos routes request to shards containing data. Each shard will perform a sort on its own data locally.

		- After local sort(s) complete, final sort-merge occurs in primary shard, then data is returned to client app.
		- Similar steps are needed for $limit and $skip - local limit/skip is performed on each shard that contains portion of the data:
		- When each shard has completed its local limit/skip, final merge of results is performed on primary shard, then results sent back to client app.

- Increasing Write Performance with Sharding
	- Sometimes, we may have too much data or  throughput for single db to handle. Solution is  _scaling_.

	- **Vertical**
		
		-   Server has too few resources (CPU, RAM, I/O).
		-   We can fix this by buying bigger faster machine with more cpu, ram and disk.
		-   But there is a limit to how much of these resources one physical machine can have.
		-   Also, Cost is not linear - buying machine that is 2x as fast, more memory, more disk could be 4x as expensive or more.

	- **Horizontal**
		-   Increase total number of servers.
		-   Split workload between many different machines.
		-   When we reach limit of current setup, add more machines.
		-   Cost scales linearly with performance requirements because of  buying more of the same machine.

	- **Shard Cluster**
		
		-   All reads/writes go via mongos, which must determine which shard to send reads/writes to.
		-   This is achieved with a shard key, which defines how data partitioned across different machines.

		- Shard key is index field or compound of index fields, must exist in every document in collection.
		- It is important to have data  _evenly_  distributed across shards, to evenly distribute load across machines.

	- **Shard Key**

		-   Using shard key, data divided into small  _chunks_.
		-   Each chunk has inclusive lower bound and exclusive upper bound.
		-   Max chunk size is 64M - when chunk reaches close to this size, it gets split.
		-   Multiple chunks exist on each cluster.

	- To have write throughput scale linearly with shards, must consider:
		- Cardinality
		- Frequency
		- Rate of Change
		
	- **_Cardinality_**:

		-   Number of distinct values for a given shard key.
		-   The higher the better.
		-   Determines max number of chunks that can exist in cluster.

		- eg: Building an app that will only be used by those living in new york. If shared on  `address.state`, would only have one chunk - all of new york would go in there, therefore would only have on shard, which defeats the purpose of sharding:

		
		- If we are unable to use a shard key with high cardinality, we can increase cardinality with a compound shard key. Eg, rather than range of values only on state, have range of values on combination of state and last name:
	~~~
	sh.shardCollection('m201.people', {'address.state': 1, last_name: 1})
	~~~
	- **_Frequency_**:

		-   Even distribution of each shard key value is good.
		-   If some values come in more often than others (eg: common last name like "Brow"), won't have even amount of load across cluster, limiting its effectiveness.


		- **HOT SHARD**: 
		-For eg: If most people have last name "Brown", throughput becomes constrained by shard containing those values .

		- **JUMBO CHUNK**:
			- Chunks containing frequently occurring values will grow larger. 
			- Usually when chunk is close to its max, will be split. 
			- BUT if chunk is created where lower and upper bound are the same, this chunk is no longer eligible for splitting.

			- Jumbo chunks reduce effectiveness of horizontal scaling because they cannot be moved between shards.

		- Frequency issue is mitigated with good compound shard key, in this example:
		~~~
		sh.shardCollection('m201.people', {'last_name': 1, _id: 1})
		~~~
		- Keys at beginning of shard key should have high cardinality, compounding key helps to distribute the frequency of popular values.

	- **_Rate of Change_**:

		-   Consider how shard key value changes over time.
		-   Avoid monotonically (always increasing or always decreasing) increasing or decreasing values. 
		- For eg - do not shard ObjectId because newer created ObjectId's are always higher in value than previously created values. 
		- This will make all writes go to last shard - i.e. shard which contains upper bound of max key.
		- If shard key was monotonically decreasing, all writes would go to first shard - i.e. shard which contains lower bound of min key.

		- **NOTE**: We can have monotonically increasing value in compound shard key, as long as it's not in the first field, as per this example. Having monotonically increasing value as second key in compound shard key is good -> increases total cardinality since its guaranteed to be unique.
		~~~
		sh.shardCollection('m201.people', {'last_name': 1, _id: 1})
		~~~
	- Bulk Writes to Shard Cluster

		- We must specify if writes should be ordered or unordered:
			~~~
			db.collection.bulkWrite(
				[
					<operation 1>,
					<operation 2>,
					...
				],
				{ordered: <boolean>}
			)
			~~~

		- When  `{ordered: true}`, server executes each operation in sequence, waiting for previous operation to succeed before executing next. If any operation fails, process halts and error is reported to client.

		- When  `{ordered: false}`, server can execute all operations in parallel.

		- With sharded cluster, ordered bulk writes can cause performance issues because mongo have to wait for the last operation to complete before next can be executed. 
		- With replica set, this won't be so bad because all databases on one machine. But in sharded cluster, each shard is on a separate machine so it has more latency waiting for each write to succeed.

		- With unordered bulk write on sharded cluster, all operations can be executed in parallel, getting more benefit of distributed nature of shard cluster.

		- **NOTE**: Mongos must deserialize the multiple write operations to appropriate nodes.

- Reading from Secondaries

	- We can specify a read preference.
	- By default, it's  `primary`  - all reads/writes go to primary server.
	- Writes can only be routed to the `primary`
	~~~
	db.people.find().readPref("primary")
	~~~
	Other readPrefs are:
	~~~
	db.people.find().readPref("primary")
	db.people.find().readPref("primaryPreferred")
	db.people.find().readPref("secondary") 						// relevant to performance
	db.people.find().readPref("secondaryPreferred")		// relevant to performance
	db.people.find().readPref("nearest")							// relevant to performance
	~~~
	- If   `readPref("secondary")`, all reads are routed to secondary.

	- If   `readPref("seoncdaryPrefered")`, reads will try to go to secondary, but if none available, routed to primary.
	- If  `readPref("nearest")`, will read from member that has lowest network latency.
	
	- **Eventual Consistency**
		- When reading from secondary, we might be reading *stale* data because data is asynchronously replicated to secondaries as writes occur on primary.

	- **Strong Consistency**
		- All writes go to primary, so when reading from primary, guaranteed to be reading latest state of data.

	- When Reading from a Secondary is a **GOOD Idea**

		- Offloading Work
			- When running analytics/report against data. 
			
			- This tends to be resource intensive and long running. We don't want this running on primary because will slow down reads/writes from operational application. 
			- It is assumed that batch report doesn't expect most up-to-date data anyways so some stale data is fine.

		- Local Reads
			- Good for geographically distributed replica sets.
			- For eg - If we have two app servers, one on west coast of US and one on east coast, have one secondary on west coast and another on east coast.
			
			- Use  `nearest`  in this case to reduce latency, IF clients are ok with reading stale data.
	- When Reading from a Secondary is a **BAD Idea**

		-   In general, Secondaries exist to provide high availability, not to increase performance. 
		- Sharding increases read/write capacity by distributing read/write operations across a group of machines.
		- We can use this instead of trying to increase performance by reading secondary.
	-   Providing extra capacity for reads: Misconception is that if primary is overworked with writes, can offload some work by reading from secondary -> FALSE because as writes come in to primary, they're replicated to secondaries, so all members of replica set have roughly same amount of write traffic.
	-   Reading from secondaries in shard cluster should *not* be done. This may return incomplete or duplicate data due to in progress chunk migrations.

- Replica sets with different indexes
	- This practice is not usual and offers only a few use cases:
		1. Specific analytics on secondary nodes
		2. Reporting on delayed consistency of data
		3. Text Search 
	
		
	- Requirements for different indexes on replica set:
		- Prevent such secondary node from becoming primary by setting their priority=0, or set as Hidden Node or simply delayed secondary nodes.
		- This is because if primary node steps down, then the app will begin communicating with members whose indexes are not designed to serve its queries. This may cause major performance hit.

- Aggregation Pipeline on a Sharded Cluster

	- Eg-1: Where collection is sharded on state, and aggregation is grouping by state.

		- This is relatively easy because all matched restaurants will be located in the same shard. So mongos can route the aggregation query to this shard, shard can compute entire aggregation, and return results to mongos.

	- Eg-2: Where results will be located across multiple shards.

		- In this case, each shard will have to do some computing, then all the individual shard results need to be sent to one place where they can be  _merged_  together.

		- Pipeline needs to be split, mongo determines which stages need to be executed on each shard, and which stages need to be executed on a single shard to merge results.

		Generally merging occurs on a  _random_  shard, except for the following stages that must be merged on the  _primary_  shard:

		-   `$out`
		-   `$facet`
		-   `$lookup`
		-   `$graphLookup`

	- Performance Implication

		- If we run above operations frequently that require merging on primary shard, that shard will receive more load than the others, reducing benefits of horizontal scaling. 

		- We can mitigate this by using better machine (more ram, faster cpu, etc.) for primary shard.
	- Aggregation Optimizations

	- Server will attempt to optimize aggregation pipeline automatically, whether sharding is present or not.

	- Match before Sort

		- We should move match stage ahead of sort to reduce number of documents that need to be sorted. This is particularly useful with sharding because it reduces the amount of data that need to be transferred across the wire for merging the results to be sorted.


## Final Exam
### Final - Question 1
**Problem:
Which of these statements is/are true?**

**Check all answers that apply:**

- Creating an ascending index on a monotonically increasing value creates index keys on the right-hand side of the index tree.
- Covered queries can sometimes still require some of your documents to be examined.
- Write concern has no impact on write latency.
- You can index multiple array fields in a single document with a single compound index.
- A collection scan has a logarithmic search time.

*Answer:*
- Creating an ascending index on a monotonically increasing value creates index keys on the right-hand side of the index tree.


### Final - Question 2

**Problem:
Which of the following statements is/are true?**

**Check all answers that apply:**

- It's important to ensure that secondaries with indexes that differ from the primary not be eligible to become primary.
- Indexes can be walked backwards by inverting their keys in a sort predicate.
- It's important to ensure that your shard key has high cardinality.
- Partial indexes can be used to reduce the size requirements of the indexes.
- Indexes can decrease insert throughput.

*Answer:*
- It's important to ensure that secondaries with indexes that differ from the primary not be eligible to become primary.
- Indexes can be walked backwards by inverting their keys in a sort predicate.
- It's important to ensure that your shard key has high cardinality.
- Partial indexes can be used to reduce the size requirements of the indexes.
- Indexes can decrease insert throughput.

### Final - Question 3

**Problem:
Which of the following statements is/are true?**

**Check all answers that apply:**

- Collations can be used to create case insensitive indexes.
- By default, all MongoDB user-created collections have an _id index.
- Background index builds block all reads and writes to the database that holds the collection being indexed.
- It's common practice to co-locate your  mongos  on the same machine as your application to reduce latency.
- MongoDB indexes are markov trees.

*Answer:*
- Collations can be used to create case insensitive indexes.
- By default, all MongoDB user-created collections have an _id index.
- It's common practice to co-locate your  mongos  on the same machine as your application to reduce latency.

### Final - Question 4

**Problem:
Which of the following statements is/are true?**

**Check all answers that apply:**

- Indexes are fast to search because they're ordered such that you can find target values with few comparisons.
- Under heavy write load you should scale your read throughput by reading from secondaries.
- When you index on a field that is an array it creates a partial index.
- On a sharded cluster, aggregation queries using  $lookup  will require a  **merge**  stage on a  **random**  shard.
- Indexes can solve the problem of slow queries.

*Answer:*
 - Indexes are fast to search because they're ordered such that you can find target values with few comparisons.
- Indexes can solve the problem of slow queries.



### Final - Question 5

**Problem:
Which of the following statements is/are true?**

**Check all answers that apply:**

- Compound indexes can service queries that filter on a  _prefix_  of the index keys.
- By default, the  explain()  command will execute your query.
- If no indexes can be used then a collection scan will be necessary.
- Compound indexes can service queries that filter on any  _subset_  of the index keys.
- Query plans are removed from the plan cache on index creation, destruction, or server restart.

*Answer:*
- Compound indexes can service queries that filter on a  _prefix_  of the index keys.
- If no indexes can be used then a collection scan will be necessary.
- Query plans are removed from the plan cache on index creation, destruction, or server restart.

### Final - Question 6

**Problem:
Which of the following statements is/are true?**

**Check all answers that apply:**

- An index doesn't become multikey until a document is inserted that has an array value.
- Indexes can only be traversed forward.
- Running performance tests from the  mongo  shell is an acceptable way to benchmark your database.
- The ideal ratio between  nReturned  and  totalKeysExamined  is 1.
- You can use the  --wiredTigerDirectoryForIndexes  option to place your indexes on a different disk than your data.

*Answer:* 
- An index doesn't become multikey until a document is inserted that has an array value.
- The ideal ratio between  nReturned  and  totalKeysExamined  is 1.
- You can use the  --wiredTigerDirectoryForIndexes  option to place your indexes on a different disk than your data.


### Final - Question 7

**Problem:
Given the following indexes:**

1.  `{ categories: 1, price: 1 }`
2.  `{ in_stock: 1, price: 1, name: 1 }`

**The following documents:**

1. `{ price: 2.99, name: "Soap", in_stock: true, categories: ['Beauty','Personal Care'] }`

2.  `{ price: 7.99, name: "Knife", in_stock: false, categories: ['Outdoors'] }`

**And the following queries:**

1.  `db.products.find({ in_stock: true, price: { $gt: 1, $lt: 5 }  }).sort({  name: 1 })`
2.  `db.products.find({ in_stock: true })`
3.  `db.products.find({ categories: 'Beauty'  }).sort({  price: 1 })`

**Which of the following is/are true?**

**Check all answers that apply:**

- Index #1 would provide a sort to query #3.
- Index #2 can be used by both query #1 and #2.
- There would be a total of 4 index keys created across all of these documents and indexes.
- Index #2 properly uses the equality, sort, range rule for query #1.

*Answer:*
- Index #1 would provide a sort to query #3.
- Index #2 can be used by both query #1 and #2.
- Index #2 properly uses the equality, sort, range rule for query #1.


 