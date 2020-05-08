
- [M121 - The MongoDB Aggregation Framework](#m121---the-mongodb-aggregation-framework)
    + [Aggregation Framework:](#aggregation-framework-)
  * [Chapter 1: Basic Aggregation - $match and $project](#chapter-1--basic-aggregation----match-and--project)
    + [Points to remember:](#points-to-remember-)
    + [Lab - $match](#lab----match)
    + [Lab - Changing Document Shape with $project](#lab---changing-document-shape-with--project)
    + [Lab - Computing Fields](#lab---computing-fields)
    + [Optional Lab - Expressions with $project](#optional-lab---expressions-with--project)
  * [Chapter 2: Basic Aggregation - Utility Stages](#chapter-2--basic-aggregation---utility-stages)
    + [Points to remember:](#points-to-remember--1)
    + [Lab - Using Cursor-like Stages](#lab---using-cursor-like-stages)
    + [Lab - Bringing it all together](#lab---bringing-it-all-together)
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
	- $map takes three arguements: 
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
