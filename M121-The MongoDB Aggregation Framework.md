
# M121 - The MongoDB Aggregation Framework




### Aggregation Framework:
	

   - Pipelines are always an array of one or more stages.
 - Stages are composed of one or more aggregation operators or expressions.	
 -   Expressions may take a single argument or an array of arguments.

 -  **Conventions**
	1. Field Path: "fieldName"    ("\$numberOfMoons")
	2. System Variable: "\$\$UPPERCASE"    ("\$\$CURRENT")
	3. User Variable: "\$\$foo"



	  

## Chapter 1: Basic Aggregation - $match and $project
### Points to remember:
- **$match:**
	- A **\$match** stage may contain a **$text** query operator, but it must be the first stage in the pipeline.
	- -$match should come early in an aggregation pipeline.
	- You cannot use **$where** with match.
	-  $match uses the **same** query syntax as find.
- **$project:**
	- Once we specify one field to retain, we must specify all fields we want to retain. The 		   _id field is the only exception to this.
	- Beyond simply removing and retaining fields, $project also lets us add new fields.
	- $project can be used as many times as required within an Aggregation Pipeline.
	- $project can be used to reassign values to existing field names and to derive entirely new fields.
- **$map:**
	- $map lets us iterate over an array, element by element, performing some transformation on each element. The result of that transformation will be returned in the same place as the original element. 
	- $map takes three arguments: 
	   1. *input*
	   2.  *as*(optional, but if omitted, each element must be referred as 	*$$this*)
	   3. *in*
### Lab - $match

**Problem: Help MongoDB pick a movie our next movie night! Based on employee polling, we've decided that potential movies must meet the following criteria.**

-   **imdb.rating  is at least 7**
-   **genres  does not contain "Crime" or "Horror"**
-   **rated  is either "PG" or "G"**
-   **languages  contains "English" and "Japanese"**

**Assign the aggregation to a variable named  pipeline**

    var pipeline = [{
        $match: {
            "imdb.rating": {
                $gte: 7
            },
            genres: {
                $nin: ["Crime", "Horror"]
            },
            rated: {
                $in: ["PG", "G"]
            },
            languages: {
                $all: ["English", "Japanese"]
            }
        }
    }]


### Lab - Changing Document Shape with $project
**Problem: Our first movie night was a success. Unfortunately, our ISP called to let us know we're close to our bandwidth quota, but we need another movie recommendation!
Using the same  $match  stage from the previous lab, add a  $project  stage to only display the the title and film rating (title  and  rated  fields).**

        MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline = [{
         $match: {
             "imdb.rating": {
                 $gte: 7
             },
             genres: {
                 $nin: ["Crime", "Horror"]
             },
             rated: {
                 $in: ["PG", "G"]
             },
             languages: {
                 $all: ["English", "Japanese"]
             }
         }
     }]
     MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate(pipeline)

### Lab - Computing Fields
**Problem: Our movies dataset has a lot of different documents, some with more convoluted titles than others. If we'd like to analyze our collection to find movie titles that are composed of only one word, we  *could*  fetch all the movies in the dataset and do some processing in a client application, but the Aggregation Framework allows us to do this on the server!
Using the Aggregation Framework, find a count of the number of movies that have a title composed of one word. To clarify, "Cinderella" and "3-25" should count, where as "Cast Away" would not.**

    MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline = [{
        $project: {
            _id: 0,
            "movSize": {
                "$size": {
                    "$split": ["$title", " "]
                }
            }
        }
    }, {
        $match: {
            movSize: 1
        }
    }]
    MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate(pipeline)



### Optional Lab - Expressions with $project
**Problem: Let's find how many movies in our  *movies*  collection are a "labor of love", where the same person appears in  cast,  directors, and  writers.**

        db.movies.aggregate([{
        $match: {
            cast: {
                $elemMatch: {
                    $exists: true
                }
            },
            directors: {
                $elemMatch: {
                    $exists: true
                }
            },
            writers: {
                $elemMatch: {
                    $exists: true
                }
            }
        }
    }, {
        $project: {
            _id: 0,
            cast: 1,
            directors: 1,
            writers: {
                $map: {
                    input: "$writers",
                    as: "writer",
                    in : {
                        $arrayElemAt: [{
                                $split: ["$$writer", " ("]
                            },
                            0
                        ]
                    }
                }
            }
        }
    }, {
        $project: {
            labor_of_love: {
                $gt: [{
                        $size: {
                            $setIntersection: ["$cast", "$directors", "$writers"]
                        }
                    },
                    0
                ]
            }
        }
    }, {
        $match: {
            labor_of_love: true
        }
    }, {
        $count: "labors of love"
    }])




## Chapter 2: Basic Aggregation - Utility Stages

### Points to remember:
- **$addFields:**
		- By combining $project with $addFields, we remove the annoyance of explicitly needing to remove or retain fields.
		- This is a style choice and can prevent having to repeatedly specify which fields to retain in larger pipelines when performing many various calculations.

- **$geoNear:**
	- must be the first stage in pipeline.
	- $geoNear requires that the collection to perform the aggregation to have only one 2dsphere geoIndex.
	- If using 2dsphere, distance is returned in metres, else if using legacy coordinates, the distance is returned in radians.
	- Arguments:(required)
		- near
		- distanceField
		- Spherical
	- Optional:
		- minDistance
		- maxDistance
		- query
		- includeLocs
		- limit
		- num
		- distanceMultiplier 
- Cursor-like Stages:
	- **$limit**
	- **$skip**
	- **$count**
	- **$sort**
- **$sample**
- Methods:
  1. {$sample : { $size: <N, how many documents>}}
	  -  [*WHEN N <=5% of total **AND** source collection>=100 documents **AND** $sample is first stage*]
	2. {$sample : { $size: <N, how many documents>}}
		- For all other conditions


###  Lab - Using Cursor-like Stages

**Problem: MongoDB has another movie night scheduled. This time, we polled employees for their favorite actress or actor, and got these results:**

    favorites = [
      "Sandra Bullock",
      "Tom Hanks",
      "Julia Roberts",
      "Kevin Spacey",
      "George Clooney"]
**For movies released in the  **USA**  with a  tomatoes.viewer.rating  greater than or equal to 3, calculate a new field called num_favs that represets how many  **favorites**  appear in the  cast  field of the movie.**
**Sort your results by  num_favs,  tomatoes.viewer.rating, and  title, all in descending order.**
**What is the  title  of the  **25th**  film in the aggregation result?****
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline = [{
        $match: {
            "tomatoes.viewer.rating": {
                $gte: 3
            },
            "countries": {
                $in: ["USA"]
            },
            "cast": {
                $exists: true
            }
        }
    },
    {
        $addFields: {
            "num_favs": {
                $size: {
                    $setIntersection: ["$cast", favorites]
                }
            }
        }
    },
    {
        $sort: {
            "num_favs": -1,
            "tomatoes.viewer.rating": -1,
            "title": -1
        }
    },
    {
        $project: {
            _id: 0,
            title: 1
        }
    },
    {
        $skip: 24
    },
    {
        $limit: 1
    }
]
~~~

    MongoDB  Enterprise  Cluster0 - shard - 0: PRIMARY > db.movies.aggregate(pipeline, { allowDiskUse:  true }).pretty()
    { "title" : "The Heat" }
Answer: *The Heat*

### Lab - Bringing it all together

**Problem: Calculate an average rating for each movie in our collection where English is an available language, the minimum  *imdb.rating*  is at least 1, the minimum  *imdb.votes*  is at least 1, and it was released in  *1990* or after. You'll be required to  rescale (or  _normalize_) *imdb.votes*.
The formula to rescale  *imdb.votes*  and calculate  *normalized_rating*  is included as a handout.
What film has the lowest  normalized_rating?**

~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline=[{
    $match: {
        "languages": "English",
        "imdb.rating": {$gte: 1},
        "imdb.votes": {$gte: 1},
        "year": {$gte: 1990}
    }
}, {
    $addFields: {
        "scaled_votes": {
            $add: [
                1, {
                    $multiply: [
                        9, {
                            $divide: [{
                                $subtract: ["$imdb.votes", 5]
                            }, {
                                $subtract: [1521105, 5]
                            }]
                        }
                    ]
                }
            ]
        }
    }
}, {
    $addFields: {
        "normalized_rating": {
            $avg: ["$scaled_votes", "$imdb.rating"]
        }
    }
}, {
    $sort: {
        "normalized_rating": 1
    }
}, {
    $project: {
        "title": 1,
        "normalized_rating": 1
    }
},{
	$limit:  1
}];
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate(pipeline,{allowDiskUse:true}).pretty()
~~~
~~~
{
        "_id" : ObjectId("573a13ccf29313caabd837cb"),
        "title" : "The Christmas Tree",
        "normalized_rating" : 1.0507662218131615
}
~~~
Answer: *The Christmas Tree*

## Chapter 3: Core Aggregation - Combining Information
### Points to remember:
- **$group**

	- _id is where we specify what incoming documents should be 	grouped on.
	- We can use all accumulator expressions within group.
	- Group can be used multiple times within a pipeline.
	- It may be necessary to sanitize incoming data.

- **$unwind**

	- $unwind only works on Array values.
	- Two forms of $unwind: 
					1. Short form
					2. Long form
	- Using unwind on large collections may lead to performance issues. 
- **$lookup**
	- Arguments: 
		 1. from ()
		 2. localField ()
		 3. foreignField ()
		 4. as () 
- **$graphLookup**
	- Arguments:
		1. from
	    2. startWith 
	      3. connectFromField 
	      4. connectToField
	      5. as
	      6. maxDepth
	      7. depthField
	      8. restrictSearchWithMatch

### Lab - $group and Accumulators

**Problem: In the last lab, we calculated a normalized rating that required us to know what the minimum and maximum values for  *imdb.votes*  were. These values were found using the  $group  stage!**
**For all films that won at least 1 Oscar, calculate the standard deviation, highest, lowest, and average  imdb.rating. Use the  **sample**  standard deviation expression.
**HINT**  - All movies in the collection that won an Oscar begin with a string resembling one of the following in their  awards  field**

~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate([{
    $match: {
        "awards": {
            $exists: true
        }
    }
}, {
    $match: {
        "awards": {
            $regex: /Won \d+ Oscar/
        }
    }
}, {
    $group: {
        _id: null,
        "highest_rating": {
            $max: "$imdb.rating"
        },

        "lowest_rating": {
            $min: "$imdb.rating"
        },
        "average_rating": {
            $avg: "$imdb.rating"
        },
        "deviation": {
            $stdDevSamp: "$imdb.rating"
        }
    }
}, {
    $project: {
        "highest_rating": 1,
        "lowest_rating": 1,
        "average_rating": 1,
        "deviation": 1
    }
}, ])
~~~
~~~
{ "_id" : null, "highest_rating" : 9.2, "lowest_rating" : 4.5, "average_rating" : 7.527024070021882, "deviation" : 0.5988145513344498 }
~~~

### Lab - $unwind

**Problem: Let's use our increasing knowledge of the Aggregation Framework to explore our movies collection in more detail. We'd like to calculate how many movies every  *cast*  member has been in and get an average  imdb.rating  for each  cast  member.**

**What is the name, number of movies, and average rating (truncated to one decimal) for the cast member that has been in the most number of movies with  *English*  as an available language?**
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline=[{
    $unwind: "$cast"
}, {
    $unwind: "$languages"
}, {
    $match: {
        languages: "English"
    }
}, {
    $group: {
        _id: "$cast",
        "numFilms": {
            $sum: 1
        },
        "average": {
            $avg: "$imdb.rating"
        },
    }
}, {
    $project: {
        _id: 1,
        numFilms: 1,
        average: {
            $trunc: ["$average", 1]
        }
    }
}, {
    $sort: {
        numFilms: -1
    }
}, {
    $limit: 1
}]
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate(pipeline,{allowDiskUse:true})
~~~
*Answer:*
~~~

{ "_id" : "John Wayne", "numFilms" : 107, "average" : 6.4 }
~~~




### Lab - Using $lookup
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline=[{
    $match: {
        airplane: /747|380/
    }
}, {
    $lookup: {
        from: "air_alliances",
        foreignField: "airlines",
        localField: "airline.name",
        as: "alliance"
    }
}, {
    $unwind: "$alliance"
}, {
    $group: {
        _id: "$alliance.name",
        count: {
            $sum: 1
        }
    }
}, {
    $sort: {
        count: -1
    }
}]
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.air_routes.aggregate(pipeline,{allowDiskUse:true})

~~~
~~~
{ "_id" : "SkyTeam", "count" : 16 }
{ "_id" : "OneWorld", "count" : 11 }
{ "_id" : "Star Alliance", "count" : 11 }
~~~
*Answer*
- SkyTeam

### Lab -

**Problem:**
**Now that you have been introduced to  $graphLookup, let's use it to solve an interesting need. You are working for a travel agency and would like to find routes for a client! For this exercise, we'll be using the  **air_airlines**,  **air_alliances**, and  **air_routes**  collections in the  **aggregations**  database.**
**Determine the approach that satisfies the following question in the most efficient manner:
Find the list of all possible distinct destinations, with at most one layover, departing from the base airports of airlines from Germany, Spain or Canada that are part of the "OneWorld" alliance. Include both the destination and which airline services that location. As a small hint, you should find  **158**  destinations.
Select the correct pipeline from the following set of options:**
*Answer:*
     
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.air_alliances.aggregate([{
  $match: { name: "OneWorld" }
}, {
  $graphLookup: {
    startWith: "$airlines",
    from: "air_airlines",
    connectFromField: "name",
    connectToField: "name",
    as: "airlines",
    maxDepth: 0,
    restrictSearchWithMatch: {
      country: { $in: ["Germany", "Spain", "Canada"] }
    }
  }
}, {
  $graphLookup: {
    startWith: "$airlines.base",
    from: "air_routes",
    connectFromField: "dst_airport",
    connectToField: "src_airport",
    as: "connections",
    maxDepth: 1
  }
}, {
  $project: {
    validAirlines: "$airlines.name",
    "connections.dst_airport": 1,
    "connections.airline.name": 1
  }
},
{ $unwind: "$connections" },
{
  $project: {
    isValid: { $in: ["$connections.airline.name", "$validAirlines"] },
    "connections.dst_airport": 1
  }
},
{ $match: { isValid: true } },
{ $group: { _id: "$connections.dst_airport" } }
])
~~~


## Chapter 4: Core Aggregation - Multidimensional Grouping
- **$facet**
	-    The  $facet  stage allows several sub-pipelines to be executed to produce multiple facets.
	-   The  $facet  stage allows the applications to generate several different facets with one single database request.
	- The  $facet  stage allows other stages to be included on the sub-pipelines, except for:
		-   $facet
		-   $out
		-   $geoNear
		-   $indexStats
		-   $collStats

	- Also, the sub-pipelines, defined for each individual facet, cannot share their output accross other parallel facets. 
	- Each sub-pipeline will receive the same input data set but does not share the result dataset with parallel facets.

- **$sortByCount**
- Just like a $group stage followed by a $sort (desc).
	- Single query facets are supported by the new aggregation pipeline stage $sortByCount.
	- except for **$out**, just like any other Aggregation pipelines, output of this stage can be used as input for downstream stages to manipulate dataset accordingly.
- **\$bucket** 
- Must always have atleast 2 values for boundaries.
- 
	- Arguments:
		- groupBy
		- boundaries (all boundaries should have same datatype)
		- default
- **$bucketAuto**
	- Will try to distribute documents evenly across all buckets.
	- adhere buckets to a numerical series set by the granularity option.(Renard Series, Power of 2, E-series, etc.) 
	 - Arguments:
		 - groupBy
		 - buckets
		 - output
		 - granularity
	- Cardinality of **groupBy** may impact even distribution and no. of buckets.
	- Specifying a **granularity**requires the expression groupBy to resolve to a numeric value.

### Lab - $facets
**Problem:
How many movies are in both the top ten highest rated movies according to the  imdb.rating  and the  metacritic  fields? We should get these results with exactly one access to the database.
**Hint:**  What is the  _intersection_?**
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> var pipeline = [{
    $match: {
        "metacritic": {
            $gt: 0
        },
        "imdb.rating": {
            $gt: 0
        }
    }
}, {
    $facet: {
        "topTenImdb": [{
            $sort: {
                "imdb.rating": -1
            }
        }, {
            $limit: 10
        }],
        "topTenMeta": [{
            $sort: {
                "metacritic": -1
            }
        }, {
            $limit: 10
        }]
    }
}, {
    $project: {
        "commonMovies": {
            $size: {
                $setIntersection: ["$topTenImdb", "$topTenMeta"]
            }
        }
    }
}]
~~~
~~~
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.movies.aggregate(pipeline,{allowDiskUse:true})
{ "commonMovies" : 1 }

~~~
*Answer*: 1



## Chapter 5: Miscellaneous Aggregation

- **$redact**
	- **\$\$KEEP** and **$$PRUNE** automatically apply to all levels below evaluated level.
	- **$$DESCEND** retains current level and evaluates next level down.
2	- $redact is not for restricting access to a collection.

- **$out**
	- Must be the last stage in a pipeline.
	- cant be used within a $facet.
	- Mongodb will create a collection of name specified / replace if collection already exists.
	- new collection will be created in the same database.
	- Must honor existing indexes if specifying an existing collection.
	-  Will not create / replace collection if pipeline errors.
	- The existing collection that is to be replaced should not be sharded.

-  **$merge**
	- can output to a collection in same or different database.
	- Arguments:
	- into(target collection)
	- on
				-  "_id" (if *on* not specified)
			-  ["_id", "shard key(s)"]
	 - whenNotMatched  (**"insert"|"discard"|"fail"**)
	 -  whenMatched ("**"merge"|"replace"|"keepExisting"|"fail"**)



- **Views**
	- Contain no data themselves, created on demand.
	- Horizontal slicing performed using $match stage.
	 - Vertical slicing performed using $project or other shaping stage.
	- No write operations
	- No index operations(create, Update)
	- No renaming	(immutable)
	- No mapReduce
	- No \$text	(\$text can only be used in first stage of a pipeline)
	- No \$geoNear(\$geoNear can only be used in first stage of a pipeline)
	- Collation restrictions	(do not inherit collations of source collection)
	- find() operation with projection operators not allowed. Eg-
			- $
			- $elemMatch
			- $slice
			- $meta
	- View Definitions are public.
	- Sensitive info must not be referred in view.
	
## Final Exam

### Final: Question 1

**Problem:
Consider the following aggregation pipelines:**

-   **Pipeline 1**
~~~
db.coll.aggregate([
  {"$match": {"field_a": {"$gt": 1983}}},
  {"$project": { "field_a": "$field_a.1", "field_b": 1, "field_c": 1  }},
  {"$replaceRoot":{"newRoot": {"_id": "$field_c", "field_b": "$field_b"}}},
  {"$out": "coll2"},
  {"$match": {"_id.field_f": {"$gt": 1}}},
  {"$replaceRoot":{"newRoot": {"_id": "$field_b", "field_c": "$_id"}}}
])
~~~
-   **Pipeline 2**
~~~
db.coll.aggregate([
  {"$match": {"field_a": {"$gt": 111}}},
  {"$geoNear": {
    "near": { "type": "Point", "coordinates": [ -73.99279 , 40.719296 ] },
    "distanceField": "distance"}},
  {"$project": { "distance": "$distance", "name": 1, "_id": 0  }}
])
~~~
-   **Pipeline 3**
 
~~~
db.coll.aggregate([
  {
    "$facet": {
      "averageCount": [
        {"$unwind": "$array_field"},
        {"$group": {"_id": "$array_field", "count": {"$sum": 1}}}
      ],
      "categorized": [{"$sortByCount": "$arrayField"}]
    },
  },
  {
    "$facet": {
      "new_shape": [{"$project": {"range": "$categorized._id"}}],
      "stats": [{"$match": {"range": 1}}, {"$indexStats": {}}]
    }
  }
])
~~~

**Which of the following statements are correct?**
Check all answers that apply:
- **Pipeline 2**  is incorrect because  $geoNear  needs to be the first stage of our pipeline
- **Pipeline 1**  is incorrect because you can only have one  $replaceRoot  stage in your pipeline
- **Pipeline 3**  fails because  $indexStats  must be the first stage in a pipeline and may not be used within a  $facet
- **Pipeline 1**  fails since  $out  is required to be the last stage of the pipeline
- **Pipeline 2**  fails because we cannot project  distance  field
- **Pipeline 3**  executes correctly
- **Pipeline 3**  fails since you can only have one  $facet  stage per pipeline


*Answer:*
 - **Pipeline 2**  is incorrect because  $geoNear  needs to be the first stage of our pipeline
- **Pipeline 3**  fails because  $indexStats  must be the first stage in a pipeline and may not be used within a  $facet
- **Pipeline 1**  fails since  $out  is required to be the last stage of the pipeline

### Final: Question 2

**Problem:
Consider the following collection:**
~~~
db.collection.find()
{
  "a": [1, 34, 13]
}
~~~
**The following pipelines are executed on top of this collection, using a mixed set of different expression accross the different stages:**

-   **Pipeline 1**
~~~
db.collection.aggregate([
  {"$match": { "a" : {"$sum": 1}  }},
  {"$project": { "_id" : {"$addToSet": "$a"}  }},
  {"$group": { "_id" : "", "max_a": {"$max": "$_id"}  }}
])
~~~
-   **Pipeline 2**
~~~
db.collection.aggregate([
    {"$project": { "a_divided" : {"$divide": ["$a", 1]}  }}
])
~~~
-   **Pipeline 3**
~~~
db.collection.aggregate([
    {"$project": {"a": {"$max": "$a"}}},
    {"$group": {"_id": "$$ROOT._id", "all_as": {"$sum": "$a"}}}
])
~~~
**Given these pipelines, which of the following statements are correct?**

**Check all answers that apply:**

- **Pipeline 2**  is incorrect since  $divide  cannot operate over field expressions
- **Pipeline 2**  fails because the  $divide  operator only supports numeric types
- **Pipeline 1**  will fail because  $max  can not operator on  _id  field
- **Pipeline 3**  is correct and will execute with no error
- **Pipeline 1**  is incorrect because you cannot use an accumulator expression in a  $match  stage.

*Answer:*
- **Pipeline 2**  fails because the  $divide  operator only supports numeric types
- **Pipeline 3**  is correct and will execute with no error
- **Pipeline 1**  is incorrect because you cannot use an accumulator expression in a  $match  stage.

### Final: Question 3

**Problem:**
**Consider the following collection documents:**
~~~
db.people.find()
{ "_id" : 0, "name" : "Bernice Pope", "age" : 69, "date" : ISODate("2017-10-04T18:35:44.011Z") }
{ "_id" : 1, "name" : "Eric Malone", "age" : 57, "date" : ISODate("2017-10-04T18:35:44.014Z") }
{ "_id" : 2, "name" : "Blanche Miller", "age" : 35, "date" : ISODate("2017-10-04T18:35:44.015Z") }
{ "_id" : 3, "name" : "Sue Perez", "age" : 64, "date" : ISODate("2017-10-04T18:35:44.016Z") }
{ "_id" : 4, "name" : "Ryan White", "age" : 39, "date" : ISODate("2017-10-04T18:35:44.019Z") }
{ "_id" : 5, "name" : "Grace Payne", "age" : 56, "date" : ISODate("2017-10-04T18:35:44.020Z") }
{ "_id" : 6, "name" : "Jessie Yates", "age" : 53, "date" : ISODate("2017-10-04T18:35:44.020Z") }
{ "_id" : 7, "name" : "Herbert Mason", "age" : 37, "date" : ISODate("2017-10-04T18:35:44.020Z") }
{ "_id" : 8, "name" : "Jesse Jordan", "age" : 47, "date" : ISODate("2017-10-04T18:35:44.020Z") }
{ "_id" : 9, "name" : "Hulda Fuller", "age" : 25, "date" : ISODate("2017-10-04T18:35:44.020Z") }
~~~
**And the aggregation pipeline execution result:**
~~~
db.people.aggregate(pipeline)
{ "_id" : 8, "names" : [ "Sue Perez" ], "word" : "P" }
{ "_id" : 9, "names" : [ "Ryan White" ], "word" : "W" }
{ "_id" : 10, "names" : [ "Eric Malone", "Grace Payne" ], "word" : "MP" }
{ "_id" : 11, "names" : [ "Bernice Pope", "Jessie Yates", "Jesse Jordan", "Hulda Fuller" ], "word" : "PYJF" }
{ "_id" : 12, "names" : [ "Herbert Mason" ], "word" : "M" }
{ "_id" : 13, "names" : [ "Blanche Miller" ], "word" : "M" }
~~~
**Which of the following pipelines generates the output result?
Choose the best answer:**
*Answer:*

~~~
var pipeline = [{
    "$project": {
      "surname_capital": { "$substr": [{"$arrayElemAt": [ {"$split": [ "$name", " " ] }, 1]}, 0, 1 ] },
      "name_size": {  "$add" : [{"$strLenCP": "$name"}, -1]},
      "name": 1
    }
  },
  {
    "$group": {
      "_id": "$name_size",
      "word": { "$push": "$surname_capital" },
      "names": {"$push": "$name"}
    }
  },
  {
    "$project": {
      "word": {
        "$reduce": {
          "input": "$word",
          "initialValue": "",
          "in": { "$concat": ["$$value", "$$this"] }
        }
      },
      "names": 1
    }
  },
  {
    "$sort": { "_id": 1}
  }
]
~~~


### Final: Question 4

**Problem:
$facet  is an aggregation stage that allows for sub-pipelines to be executed.**
~~~
var pipeline = [
  {
    $match: { a: { $type: "int" } }
  },
  {
    $project: {
      _id: 0,
      a_times_b: { $multiply: ["$a", "$b"] }
    }
  },
  {
    $facet: {
      facet_1: [{ $sortByCount: "a_times_b" }],
      facet_2: [{ $project: { abs_facet1: { $abs: "$facet_1._id" } } }],
      facet_3: [
        {
          $facet: {
            facet_3_1: [{ $bucketAuto: { groupBy: "$_id", buckets: 2 } }]
          }
        }
      ]
    }
  }
]
~~~
**In the above pipeline, which uses  $facet, there are some incorrect stages or/and expressions being used.
Which of the following statements point out errors in the pipeline?**

**Check all answers that apply:**
- a  $multiply  expression takes a document as input, not an array.
- can not nest a  $facet  stage as a sub-pipeline.
- facet_2  uses the output of a parallel sub-pipeline,  facet_1, to compute an expression
- $sortByCount  cannot be used within  $facet  stage.
- a  $type  expression does not take a string as its value; only the BSON numeric values can be specified to identify the types.

*Answer:*
- can not nest a  $facet  stage as a sub-pipeline.
- facet_2  uses the output of a parallel sub-pipeline,  facet_1, to compute an expression
 
### Final: Question 5

Problem:

Consider a company producing solar panels and looking for the next markets they want to target in the USA. We have a collection with all the major  **cities**  (more than 100,000 inhabitants) from all over the World with recorded number of sunny days for some of the last years.

A sample document looks like the following:

db.cities.findOne()
{
"_id": 10,
"city": "San Diego",
"region": "CA",
"country": "USA",
"sunnydays": [220, 232, 205, 211, 242, 270]
}

 COPY

The collection also has these indexes:

db.cities.getIndexes()
[
{
  "v": 2,
  "key": {
    "_id": 1
  },
  "name": "_id_",
  "ns": "test.cities"
},
{
  "v": 2,
  "key": {
    "city": 1
  },
  "name": "city_1",
  "ns": "test.cities"
},
{
  "v": 2,
  "key": {
    "country": 1
  },
  "name": "country_1",
  "ns": "test.cities"
}
]

 COPY

We would like to find the cities in the USA where the minimum number of sunny days is 200 and the average number of sunny days is at least 220. Lastly, we'd like to have the results sorted by the city's name. The matching documents may or may not have a different shape than the initial one.

We have the following query:

var pipeline = [
    {"$addFields": { "min": {"$min": "$sunnydays"}}},
    {"$addFields": { "mean": {"$avg": "$sunnydays" }}},
    {"$sort": {"city": 1}},
    {"$match": { "country": "USA", "min": {"$gte": 200}, "mean": {"$gte": 220}}}
]
db.cities.aggregate(pipeline)

 COPY

However, this pipeline execution can be optimized!

Which of the following choices is still going to produce the expected results and likely improve the most the execution of this aggregation pipeline?

**Attempts Remaining:**Correct Answer

Choose the best answer:
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NzgwMTYxNDgsLTc2NTczMTc2NSw4NT
YxMTA5NTIsLTEzMzk2MDA3NzQsLTEzMTIxNTg3MDQsLTgwMTIy
OTY2NCw0MTYzNzgzMDhdfQ==
-->