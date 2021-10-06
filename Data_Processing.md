# Data Processing

#  Define a mapping that satisfies a given set of requirements

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/mapping.html

> Mapping is the process of defining how a document, and the fields it contains, are stored and indexed.

> Each document is a collection of fields, which each have their own data type. When mapping your data, you create a mapping definition, which contains a list of fields that are pertinent to the document. 

> A mapping definition also includes metadata fields, like the `_source` field, which customize how a document’s associated metadata is handled.


## Dynamic mapping
> Dynamic mapping allows you to experiment with and explore data when you’re just getting started. Elasticsearch adds new fields automatically, just by indexing a document. You can add fields to the top-level mapping, and to inner object and nested fields.

## Explicit mapping
> Explicit mapping allows you to precisely choose how to define the mapping definition, such as:
> 
> - Which string fields should be treated as full text fields.
> - Which fields contain numbers, dates, or geolocations.
> - The format of date values.
> - Custom rules to control the mapping for dynamically added fields.

:question: 1. Create a new index mapping called `henry4` that matches the following requirements:

- speaker: keyword
- line_id: keyword and not aggregateable 
- speech_number: integer

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/mapping-types.html


### Mapping numeric identifiers
>Not all numeric data should be mapped as a numeric field data type. Elasticsearch optimizes numeric fields, such as integer or long, for range queries. However, keyword fields are better for term and other term-level queries.
>
>Identifiers, such as an ISBN or a product ID, are rarely used in range queries. However, they are often retrieved using term-level queries.
>
>Consider mapping a numeric identifier as a keyword if:
>
> - You don’t plan to search for the identifier data using range queries.
> - Fast retrieval is important. term query searches on keyword fields are often faster than term searches on numeric fields.
> 
> If you’re unsure which to use, you can use a multi-field to map the data as both a keyword and a numeric data type.




```json
PUT /henry4
{
 "settings": {
   "number_of_replicas": 0
 },
 "mappings": {
   "properties": {
    "speaker": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
  }
 }
}
```

:bulb: #FIXME: I think `line_id` should be a keyword

</details>
<hr/>

:question: 2. Using the previous ingested `shakespeare` index, re-index the data into the new one called `henry4` that only contains the lines for the play `Henry IV`

<details>
  <summary>View Solution (click to reveal)</summary>

The best way to do this is to write the term query first, check that contains what you want, then convert the query into the `_reindex`

```json
POST _reindex
{
  "source": { "index": "shakespeare",
    "query": {
      "term": {
        "play_name": "Henry IV"
      }
    }
  },
  "dest":   { "index": "henry4" }
}

// output

{
  "took" : 2844,
  "timed_out" : false,
  "total" : 3205,
  "updated" : 0,
  "created" : 3205,
  "deleted" : 0,
  "batches" : 4,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```
</details>
<hr/>

3. verify that the data only contains `HENRY IV` play lines.

#TBC


#  Define and use a custom analyzer that satisfies a given set of requirements

and

# Define and use multi-fields with different data types and/or analyzers

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/mapping-types.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/analyzer.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/analysis-standard-analyzer.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/analysis-custom-analyzer.html

:question: 1. Write a custom analyzer that changes the name of `PRINCE HENRY` to `WAYWARD PRINCE HAL` in the `speaker` field, add this to a new index called `henry4_hal`

- Create a new index, 
- with a mapping on the speaker field 
- that utilises an analyser 
- to rename the princes name if it matches.

<details>
  <summary>View Solution (click to reveal)</summary>


## Test the analyser

```json
POST _analyze
{
  "char_filter": {
      "type": "pattern_replace",
      "pattern": "PRINCE HENRY",
      "replacement": "WAYWARD PRINCE HAL"
  },
  "text": [
    "PRINCE WILLIAM",
    "PRINCE HENRY",
    "PRINCE HARRY"
  ]
}

// output

{
  "tokens" : [
    {
      "token" : "PRINCE WILLIAM",
      "start_offset" : 0,
      "end_offset" : 14,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "WAYWARD PRINCE HAL",
      "start_offset" : 15,
      "end_offset" : 27,
      "type" : "word",
      "position" : 101
    },
    {
      "token" : "PRINEC HARRY",
      "start_offset" : 28,
      "end_offset" : 40,
      "type" : "word",
      "position" : 202
    }
  ]
}

```

## Put it all together

- mappings -> `speaker` -> `"analyzer": "wayward_son_analyser"`

- settings -> analysis -> `"wayward_son_analyser"` -> `"char_filter": ["rename_filter"]`

- `"rename_filter"` -> `"pattern_replace"`

```json
PUT /henry4_hal
{
  "mappings": {
    "properties": {
      "speaker": {
        "type": "text",
        "analyzer": "wayward_son_analyser",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "line_id": {
        "type": "integer"
      },
      "speech_number": {
        "type": "integer"
      }
    }
  },
  "settings": {
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "wayward_son_analyser": {
          "type":      "custom", 
          "tokenizer": "standard",
          "char_filter": [
            "rename_filter"
          ]
        }
      },
      "char_filter": {
        "rename_filter": {
          "type": "pattern_replace",
          "pattern": "PRINCE HENRY",
          "replacement": "WAYWARD PRINCE HAL"
        }
      }
    }
  }
}
```
</details>
<hr/>

:question: 2. re-index `henry4` into `henry4_hal`

<details>
  <summary>View Solution (click to reveal)</summary>

```json
POST _reindex
{
  "source": { "index": "shakespeare",
    "query": {
      "term": {
        "play_name": "Henry IV"
      }
    }
  },
  "dest":   { "index": "henry4_hal" }
}
```
</details>
<hr/>

:question: 3. verify by querying the `henry4_hal` index for the speaker `HAL`, `WAYWARD` and `PRINCE HENRY`

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET henry4_hal/_search
{
  "query": {
    "term": {
      "speaker": "HAL"
    }
  }
}
```

> NOTE: Oddly you can't see the `HAL` or `WAYWARD` in the returned data, but you can search for it.
> What you get returned is the original data `PRINCE HENRY`

#TBC check why this is.
</details>
<hr/>






# Use the Reindex API and Update By Query API to reindex and/or update documents

## Part 1
:question: Reindex the `accounts-raw` index into `accounts-2021`.

:question: Then reindex `accounts-2021` into `accounts-female` index where only the female account holders are present.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html

> :warning: 
Reindex requires _source to be enabled for all documents in the source index.

:warning: check your templates, to make sure they are not forcing `"_source": { "enabled": false },` as this will break reindexing.

```json
POST _reindex
{
  "source": { "index": "accounts-raw"  },
  "dest":   { "index": "accounts-2021" }
}

GET accounts-2021/_count?filter_path=count

// Output 

{
  "count" : 1000
}
```

reindex into `accounts-female`

:bulb: do the term query first, then once you are happy with the output, convert it into a `_reindex`

```json
POST _reindex
{
  "source": { "index": "accounts-raw",
    "query": {
      "term": {
        "gender.keyword": "F"
      }
    }
  },
  "dest":   { "index": "accounts-female" }
}
```

Check
```json
GET accounts-female/_count?filter_path=count

// Output 

{
  "count" : 493
}
```

Check again
```json
GET /accounts-female/_search?filter_path=*.*.*.gender

// Output 

{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "gender" : "F"
        }
      },
      {
        "_source" : {
          "gender" : "F"
        }
      },
    ...
```
</details>

## Part 2
:question: Give all female account holders in `accounts-2021` a 25% bonus increase on their balance :)

<details>
  <summary>View Solution (click to reveal)</summary>

Get two example docs
```json
GET /accounts-raw/_search?q=gender:F&size=2
```

Note down those ids and get the balances
```json
GET /accounts-raw/_doc/_mget?filter_path=*.*.balance
{
    "ids" : ["13", "25"]
}

//Output

{
  "docs" : [
    {
      "_source" : {
        "balance" : 32838
      }
    },
    {
      "_source" : {
        "balance" : 40540
      }
    }
  ]
}
```

Update the accounts - take note of the number of `updated` docs
```json
POST accounts-2021/_update_by_query
{
  "script": {
    "source": "ctx._source.balance=ctx._source.balance*1.25",
    "lang": "painless"
  },
  "query": {
    "term": {
      "gender": "F"
    }
  }
}


// Output

{
  "took" : 369,
  "timed_out" : false,
  "total" : 493,
  "updated" : 493,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

Get those same ids again to check the balances have increased
```json
GET /accounts-2021/_doc/_mget?filter_path=*.*.balance
{
    "ids" : ["13", "25"]
}

// Output

{
  "docs" : [
    {
      "_source" : {
        "balance" : 41047.5
      }
    },
    {
      "_source" : {
        "balance" : 50675.0
      }
    }
  ]
}
```

</details>
<hr>

# Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

:question: Apply a pipeline called `accounts-ingest` to the data in `accounts-2021` with the following requirements:

- Add a `tag` called `pipeline_ingest` to show that the document was ingested via the pipeline 
- Combine the `firstname` and `lastname ` fields into a new field called `full_name`
- Increase the `balance` of holders that are `female` and `39 years old and over` by 5%.


**Hint**: Use the simulate API to test your code

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest-apis.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/accessing-data-in-pipelines.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest-processors.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/script-processor.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/set-processor.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/append-processor.html


> :warning: The value of ctx is read-only in if conditions.

> :question: Thus use a script to do more complex work

:bulb: it is probably quicker now to do this into the web UI.

## Simulate first

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "append": {
          "field": "tags",
          "value": ["pipeline_ingest"]
        }
      },
      {
        "set": {
          "tag": "set full_name",
          "field": "full_name",
          "value": "{{firstname}} {{lastname}}"
        }
      },
      {
        "script": {
          "tag": "39s and over female bonus",
          "if": """
            if (ctx.age >= 39) { 
              if (ctx.gender=="F") { 
                return true 
              }
            } 
            return false
          """,
          "lang": "painless",
          "source": """
            ctx.balance = ctx.balance*1.05
          """
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "account_number": 10000,
        "balance": 1000000,
        "firstname": "George",
        "lastname": "Cross",
        "age": 92,
        "gender": "M",
        "address": "1 Dog Lane",
        "employer": "Wheatens",
        "email": "george@wheatens.com",
        "city": "London",
        "state": "UK"
      }
    },
    {
      "_source": {
        "account_number": 10001,
        "balance": 1000001,
        "firstname": "Millie",
        "lastname": "Cross",
        "age": 84,
        "gender": "F",
        "address": "1 Dog Lane",
        "employer": "Wheatens",
        "email": "millie@wheatens.com",
        "city": "London",
        "state": "UK"
      }
    }
  ]
}
```

Here you will get a lot of output, make sure it matches what you expect to see.


Now copy the working pipeline

```json
PUT _ingest/pipeline/accounts-ingest
{
  "description" : "pipeline to account ingest",
  "processors": [
      {
        "append": {
          "field": "tags",
          "value": ["pipeline_ingest"]
        }
      },
      {
        "set": {
          "tag": "set full_name",
          "field": "full_name",
          "value": "{{firstname}} {{lastname}}"
        }
      },
      {
        "script": {
          "tag": "39s and over female bonus",
          "if": """
            if (ctx.age >= 39) { 
              if (ctx.gender=="F") { 
                return true 
              }
            } 
            return false
          """,
          "lang": "painless",
          "source": """
            ctx.balance = ctx.balance*1.05
          """
        }
      }
    ]
}
```

Then reindex the data with that new pipeline

```json
POST accounts-2021/_update_by_query?pipeline=accounts-ingest

// Output - check the `updated` field

{
  "took" : 651,
  "timed_out" : false,
  "total" : 1000,
  "updated" : 1000,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

### Check some docs

Pick a doc id

```json
POST accounts-raw/_search?filter_path=*.*._id
{
  "size": 1, 
  "query": { 
    "bool": { 
      "must": [
        { "match": { "gender.keyword":   "F"}}
      ],
      "filter": [ 
        { "range": { "age": { "gte": "39" }}}
      ]
    }
  }
}

// Output 

{
  "hits" : {
    "hits" : [
      {
        "_id" : "25"
      }
    ]
  }
}
```


Original data
```json
GET accounts-raw/_doc/25?filter_path=*.balance,*.firstname,*.lastname

// Output 

{
  "_source" : {
    "balance" : 40540,
    "firstname" : "Virginia",
    "lastname" : "Ayala",
    "age" : 39
  }
}
```

Updated data
```json
GET accounts-2021/_doc/25?filter_path=*.balance,*.full_name,*.tags

// Output 

{
  "_source" : {
    "tags" : [
      "pipeline_ingest"
    ],
    "full_name" : "Virginia Ayala",
    "balance" : 42567.0
  }
}
```

```
40540 * 1.05 = 42567
```

</details>
<hr>

# Configure an index so that it properly maintains the relationships of nested arrays of objects

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html

:question: 1. Using the below data, create an index with a mapping that allows for relationships to be queried.

```json
PUT henry4_r/_doc/_bulk
{"index":{"_index":"henry4_r","_id":"0"}}
{"name":"KING HENRY IV","relationship":[{"name":"PRINCE HENRY","type":"father"}]}
{"index":{"_index":"henry4_r","_id":"1"}}
{"name":"FALSTAFF","relationship":[{"name":"PRINCE HENRY","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"2"}}
{"name":"HOTSPUR","relationship":[{"name":"PRINCE HENRY","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"3"}}
{"name":"WESTMORELAND","relationship":[{"name":"KING HENRY IV","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"4"}}
{"name":"WESTMORELAND","relationship":[{"name":"KING HENRY IV","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"5"}}
{"name":"EARL OF WORCESTER","relationship":[{"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"6"}}
{"name":"GLENDOWER","relationship":[{"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"7"}}
{"name":"EARL OF DOUGLAS","relationship":[{"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"8"}}
{"name":"MORTIMER","relationship":[{"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"9"}}
{"name":"VERNON","relationship":[{"name":"EARL OF WORCESTER","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"10"}}
{"name":"NORTHUMBERLAND","relationship":[{"name":"HOTSPUR","type":"father"}, {"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"11"}}
{"name":"SIR WALTER BLUNT","relationship":[{"name":"KING HENRY IV","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"12"}}
{"name":"GADSHILL","relationship":[{"name":"PRINCE HENRY","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"13"}}
{"name":"ARCHBISHOP OF YORK","relationship":[{"name":"KING HENRY IV","type":"foe"}]}
{"index":{"_index":"henry4_r","_id":"14"}}
{"name":"POINS","relationship":[{"name":"FALSTAFF","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"15"}}
{"name":"BARDOLPH","relationship":[{"name":"FALSTAFF","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"16"}}
{"name":"PETO","relationship":[{"name":"FALSTAFF","type":"friend"}]}
{"index":{"_index":"henry4_r","_id":"17"}}
{"name":"PRINCE HENRY","relationship":[{"name":"FALSTAFF","type":"friend"}, {"name":"PETO","type":"friend"},{"name":"BARDOLPH","type":"friend"},{"name":"POINS","type":"friend"},{"name":"GADSHILL","type":"friend"}]}
```

<details>
  <summary>View Solution (click to reveal)</summary>

The important part here is to add the `"type": "nested"` so that the data is not flattened when ingested.

So anywhere you have more than one item in the relationship list, it will not be found if you do not use the `nested` term. see docs.

```json
PUT henry4_r
{
  "mappings": {
    "properties": {
      "relationship": {
        "type": "nested" 
      }
    }
  }
}

```
</details>
<hr/>

:question: 2. Then query all people that are a `foe` of `KING HENRY IV`

<details>
  <summary>View Solution (click to reveal)</summary>

You need to be using a `nested` query here as well.  Otherwise it will not work.

```json
GET henry4_r/_search
{
  "query": {
    "nested": {
      "path": "relationship",
      "query": {
        "bool": {
          "must": [
            { "match": { "relationship.name": "KING HENRY IV" }},
            { "match": { "relationship.type":  "foe" }} 
          ]
        }
      }
    }
  }
}
// output

{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "name" : "EARL OF WORCESTER"
        }
      },
      {
        "_source" : {
          "name" : "GLENDOWER"
        }
      },
      {
        "_source" : {
          "name" : "EARL OF DOUGLAS"
        }
      },
      {
        "_source" : {
          "name" : "MORTIMER"
        }
      },
      {
        "_source" : {
          "name" : "ARCHBISHOP OF YORK"
        }
      },
      {
        "_source" : {
          "name" : "NORTHUMBERLAND"
        }
      },
      {
        "_source" : {
          "name" : "HOTSPUR"
        }
      }
    ]
  }
}
```

</details>
<hr/>

:question: 3. Show all friends of `FALSTAFF`


<details>
  <summary>View Solution (click to reveal)</summary>

There should be four.  This is where the nesting comes into play as `FALSTAFF` himself decribes `PRINCE HENRY` as his only friend. But other people describe `FALSTAFF` as their friend.

```
PRINCE HENRY -> FALSTAFF
FALSTAFF -> PRINCE HENRY
POINS -> FALSTAFF
BARDOLPH -> FALSTAFF
PETO -> FALSTAFF
```

# query

```json
GET henry4_r/_search?filter_path=*.*.*.name
{
  "query": {
    "nested": {
      "path": "relationship",
      "query": {
        "bool": {
          "must": [
            { "match": { "relationship.name": "FALSTAFF" }},
            { "match": { "relationship.type":  "friend" }} 
          ]
        }
      }
    }
  }
}

// output

{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "name" : "POINS"
        }
      },
      {
        "_source" : {
          "name" : "BARDOLPH"
        }
      },
      {
        "_source" : {
          "name" : "PETO"
        }
      },
      {
        "_source" : {
          "name" : "PRINCE HENRY"
        }
      }
    ]
  }
}
```

</details>
<hr/>
