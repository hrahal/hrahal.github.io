---
layout: post
title: Elasticsearch Intro
---

ElasticSearch is an open source, RESTful search engine built on top of Apache Lucene and released under an Apache license. It is Java-based and can search and index document files in diverse formats.

ElasticSearch has been compared to Apache Solr and offers several notable features:

 - Provides a scalable search solution.
 - Performs near-real-time searches.
 - Provides support for multi-tenancy.
 - An index can be easily recovered in a case of a server crash.
 - Uses Javascript Object Notation (JSON) and Java application program interfaces (APIs).
 - Automatically indexes JSON documents.
 - Each index can have its own settings.
 - Searches can be done with Lucene-based querystrings.

Indices and types
-------
Every time you store data in _Elasticsearch_ it gets saved inside an **index** which has a **type**. compared to _MongoDB_ an index is similar to a database, and a type similar to a collection. Compared to _SQL_ an index would be like a database, and a type like a table.

convention: 
```
localhost:9200/{index}/{type}/
```

**Important note** different types living in the same index cannot have the same field name with different config or field type


For example the following two documents can't co-exist since they share the same index, and both have a _city_ attribute of different types, _string_ and _object_ respectively 

```
localhost:9200/test/users/1
```
```
{
    "city": "cityID123"
}
```
```
localhost:9200/test/city/1
```
```
{
    "city": {
        "name": "Toronto"
    }
}
```

When developing with elasticsearch there are 3 main steps we have to consider. **Mapping**, **Indexing**, and **Searching** data

### 1. Mapping

Mapping is used to define how elastic should store and index a particular document and it's fields. 

However if no mapping was introduced to a specific field on pre-index time, elastic will [dynamically](https://www.elastic.co/guide/en/elasticsearch/guide/current/dynamic-mapping.html) add a **generic** type to that field. Although this may sound tempting, it is NOT! since generic types are very basic and do not meet the query expectations most of the time.

Moving forward with this tutorial we will base our examples on the following data schema: 

```
{
    "first_name": "bam",
    "last_name": "margera",
    "gender": "male",
    "age": 36
}
``` 

So to make things more efficient we're gonna create the index, type and mapping for the schema in one request. Something that looks like the following: 

```
PUT localhost:9200/test/

{
    "mappings": {
        "users": {
            "properties": {
                "age": {
                    "type": "long"
                },
                "first_name": {
                    "type": "string"
                },
                "gender": {
                    "type": "string"
                },
                "level": {
		            "type": "string"
		        },
                "last_name": {
                    "type": "string"
                }
            }
        }
    }
}

```
so creating an _Index_ called test, a _type_ called users with 5 fields that it contains.

note that field types can have the following values: _string_, _date_, _long_, _double_, _boolean_, _ip_, _object_, _nested_, _geo_point_, _geo_shape_

If everything goes well, we should get the following response: 

```
{
  "acknowledged": true
}
```

Now that we told elasticsearch what kind of data we want to insert, let's go ahead and index/store it.

### 2. Indexing

Indexing/storing is the process of inserting data to ES to make it **searchable** using the _Index API_.

So let's  index 3 simple documents: 

```
POST localhost:9200/test/users/
{
    "first_name": "Bam",
    "last_name": "Margera",
    "gender": "male",
    "level": "super awesome",
    "age": 36
}

POST localhost:9200/test/users/

{
    "first_name": "Stephanie",
    "last_name": "Hodge",
    "gender": "female",
    "level": "awesome",
    "age": 34
}

POST localhost:9200/test/users/

{
    "first_name": "Johnny",
    "last_name": "Knoxville",
    "gender": "male",
    "level": "awesome",
    "age": 45
}

```

on success of any of the following docs, we should see a response like this: 

```
{
  "_index": "test",
  "_type": "users",
  "_id": "AVRQDOka0YBBUjDwpzQQ",
  "_version": 1,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": true
}
```

where __id_ is a generated id by ES that is 20 character long, URL-safe, Base64-encoded GUID string.

we can also specify our own id after the type like this: 

```
POST localhost:9200/test/users/MyID123
{
    "first_name": "Bam",
    "last_name": "Margera",
    "gender": "male",
    "level": "super awesome",
    "age": 36
}
```

Now what we have our data indexed let's move forward to query it.

### 3. Searching

In this section we will cover ES _Queries_, _Filters_, and _Aggregations_ for search

To search in a specific _index_ and _type_ the following convention is used: 

```
POST localhost:9200/test/users/_search
```

so now by hitting this request, the response will look like: 

```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "test",
        "_type": "users",
        "_id": "AVRQQlCE0YBBUjDwpzQZ",
        "_score": 1,
        "_source": {
          "first_name": "Bam",
          "last_name": "Margera",
          "gender": "male",
          "level": "super awesome",
          "age": 36
        }
      },
      **... the other 2 docs go here**
    ]
  }
}
```

By looking at this response we can see that the data that we inserted is found inside the hits.hits _array_ included inside the __source_ _object_

and since we didn't actually specify anything to search for we'll get a __score_ of 1 for all docs. 

on the top level hits.total is the total number of the docs we using an empty search query. and max_score is the maximum score a document can take is a specific query. in out case it's 1 since no query was specified. 

in the __shards.total_ value is the number of lucene Indexed that elasticsearch created for that index. The default number is always 5 unless we specify otherwise on index creation time. more about shards is explained [here](https://www.elastic.co/guide/en/elasticsearch/guide/current/_add_an_index.html) 

#### a. Queries 

Queries is what we use to get results with **scoring** (relevance)

To ask a question like
 
 - level = "super awesome"

Using the _match_ query for **full-text** that is used on analyzed fields, we would write: 

```
POST localhost:9200/test/users/_search

{
    "query": {
        "match": {
            "level": "super awesome"
        }
    }
}
``` 

the response will be: 

```
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.2712221,
    "hits": [
      {
        "_index": "test",
        "_type": "users",
        "_id": "AVRQQlCE0YBBUjDwpzQZ",
        "_score": 0.2712221,
        "_source": {
          "first_name": "Bam",
          "level": "super awesome",
          ...
        }
      },
      {
        "_index": "test",
        "_type": "users",
        "_id": "AVRQRtYW0YBBUjDwpzQa",
        "_score": 0.09848769,
        "_source": {
          "first_name": "Stephanie",
          "level": "awesome",
          ...
        }
      },
      {
        "_index": "test",
        "_type": "users",
        "_id": "AVRQRx-E0YBBUjDwpzQf",
        "_score": 0.09848769,
        "_source": {
          "first_name": "Johnny",
          "level": "awesome",
          ...
        }
      }
    ]
  }
}
```

As we can see that the user _Bam_ scored the highest of 0.2712221 since his level was "super awesome ", whereas _Stephanie_ &  _Johnny_ scored an equal 0.09848769 for their level was just "awesome"


where as for **exact values** on _non-analyzed_ fields, _numbers_, _dates_, and _Booleans_, it's better to use the _Term Query_ :

 - age = 36

```
POST localhost:9200/test/users/_search

{
     "query": {
        "term": {
            "age": 36
        }
    }
}
```

this query will return **only** Bam

To combine more than one query together we can use the _Query clause_  to find: 

 - level = "super awesome" AND "age" < 40

```
POST localhost:9200/test/users/_search

{
     "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "level": "super awesome"
                    }
                },
                {
                    "range": {
                        "age": {
                            "lt": 40
                        }
                    }
                }
            ]
        }
    }
}
```

Where _must_ is and array that implies **AND**. _bool_ also supports _should_ implying **OR**, and must_not.

Moreover we used the _range_ query with age "lt" **less than** 40. where _range_ also supports "lte", "gt", and "gte" 

#### b. Filters 

Filters are **non-scoring** queries that can be used if the score has no importance. It's returns a _boolean_ that answers with yes or no **where the score is always = 1**

so executing the following filter has no significance on the score, but will return only 2 docs: 

```
POST localhost:9200/test/users/_search

{
    "filter": {
       "match": {
            "gender": "male"
        }
    }
}
``` 

Whereas combining this with a previous query:

 - level = "super awesome" AND only return gender = "male"

```
POST localhost:9200/test/users/_search

{
     "query": {
        "match": {
            "level": "super awesome"
        }
    },
     "filter": {
        "match": {
            "gender": "male"
        }
    }
}
```
 
This will return only 2 users Bam and Johnny **scoring** 0.2712221 and 0.09848769 respectively. Where Bam has a more relevant level than Johnny.

Although this works fine but it is **bad for performance** since it will execute the query first then apply the filter returned results. 

To force ES to apply the filter before in order limit the number of docs then apply the query we should wrap everything in a bool clause then add the filter next to **must**: 


```
POST localhost:9200/test/users/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "level": "super awesome"
                    }
                }
            ],
            "filter": {
                "match": {
                    "gender": "male"
                }
            }
        }    
    }
}
``` 

<br>
**more more more...**

 - level = "super awesome", and age < 40 but only return gender = "male"

we would write:

```
POST localhost:9200/test/users/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "level": "super awesome"
                    }
                },
                {
                    "range": {
                        "age": {
                            "lt": 40
                        }
                    }
                }
            ],
            "filter": {
                "match": {
                    "gender": "male"
                }
            }
        }    
    }
}
```
 
This will return only 1 user Bam **scoring**  1.0253175.

**Important note** we can also combine more than 1 filter using the bool 

So as ES states it: "As a general rule, use query clauses for full-text search or for any condition that should affect the relevance score, and use filters for everything else."


#### c. Aggregations

[Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) is a big part of elasticseach it is used to calculate stats about our data. Divided into 3 different types: 

 - Metrics Aggregations
 - Bucket Aggregations
 - Pipeline Aggregations
 
 In this tutorial I'm gonna cover the _Term Aggregations_ which is a part of the Bucket Aggregations. 

 - how many females and males We,v got in our Index/type ?

we can write the following: 

```
POST localhost:9200/test/users/_search

{
    "size": 0,
    "aggs" : {
        "genders" : {
            "terms" : { "field" : "gender" }
        }
    }
}
```

We set "size" = 0 since we don't want to see any search results. just the aggs results. "aggs" is a predefined ES property, followed by "genders" which is a property that we can freely name. We can call it "xyz" if we want.

"terms" implies that that we are performing a term aggregation which specifies the field name that we want to agg > genders. 

response:

```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "genders": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "male",
          "doc_count": 2
        },
        {
          "key": "female",
          "doc_count": 1
        }
      ]
    }
  }
}
``` 

what we want is every thing inside the bucket array which tells us that we have, 1 female and 2 males. 

The power off aggs is that it can be combines with any filter/query. 

so using the last filter we did, we can simply say: 

```
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "level": "super awesome"
                    }
                },
                {
                    "range": {
                        "age": {
                            "lt": 40
                        }
                    }
                }
            ],
            "filter": {
                "match": {
                    "gender": "male"
                }
            }
        }    
    },
    "aggs" : {
        "genders" : {
            "terms" : { "field" : "gender" }
        }
    }
}
```

This will return Bam with a male count = 1 : )

TADAH! 
