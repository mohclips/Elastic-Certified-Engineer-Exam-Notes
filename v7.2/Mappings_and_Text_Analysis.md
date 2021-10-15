# Mappings and Text Analysis

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping-types.html


#  Define a mapping that satisfies a given set of requirements

:question: 1. Create a new index mapping called `henry4` that matches the following requirements:

- speaker: keyword
- line_id: keyword and not aggregateable 
- speech_number: integer

<details>
  <summary>View Solution (click to reveal)</summary>

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
</details>
<hr/>

:question: 2. Using the previous ingested `shakespeare` index, re-index the data into the new one called `henry4` that only contains the lines for the play `Henry IV`

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping-types.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analyzer.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-standard-analyzer.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html

:question: 1. Write a custom analyzer that changes the name of `PRINCE HENRY` to `WAYWARD PRINCE HAL` in the `speaker` field, add this to a new index called `henry4_hal`

<details>
  <summary>View Solution (click to reveal)</summary>

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

You need to be using a `nested` quesry here as well.  Otherwise it will not work.

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

There should be four.  This is where the nesting comes into play.

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
