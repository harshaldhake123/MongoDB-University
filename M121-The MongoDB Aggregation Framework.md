
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



## Chapter 5: 

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
	 - whenNotMatched  ("**insert/discard/fail**")
	 -  whenMatched ("**merge**"|replace|keepExistig)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ4NDk0MjcyLC0xMzEyMTU4NzA0LC04MD
EyMjk2NjQsNDE2Mzc4MzA4XX0=
-->