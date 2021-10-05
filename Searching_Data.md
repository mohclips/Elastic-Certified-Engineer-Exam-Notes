# Searchig Data

:warning: These example questions use the eCommerce sample data and the Shakespeare data.


# Prerequisite

:scroll: Download the Shakespeare data, create an index mapping as below and insert the data.

:arrow_down: https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json


```json
PUT /shakespeare
{
 "mappings": {
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
  }
 }
}
```

```bash
$ curl -u "elastic:Password01" -s -H "Content-Type: application/x-ndjson" -XPUT localhost:9200/shakespeare/_bulk --data-binary "@shakespeare_6.0.json"; echo
```


# Write and execute a search query for terms and/or phrases in one or more fields of an index

## Full Text queries

:question: How many times do the words `brothers`, `blood` appear in the `shakespeare` data?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "brothers blood"
      }
    }
  }
}
```

> Answer: 761

</details>
<hr>

:question: How many times do the words `brothers` and `blood` appear in the `shakespeare` data?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "brothers blood",
        "operator": "and"
      }
    }
  }
}
```

> Answer: 4

</details>
<hr>

:question: How many times does the phrase `brothers blood` appear in the `shakespeare` data?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase": {
      "text_entry": {
        "query": "brothers blood"
      }
    }
  }
}
```

> Answer: 3

</details>
<hr>

:question: Find how many lines there are, where `Prince Henry` is mentioned or is a speaker in the `shakespeare` data?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "multi_match" : {
      "query":    "prince henry", 
      "fields": [ "text_entry", "speaker" ],
      "type": "phrase"
    }
  }
}
```

> Answer: 22

</details>
<hr>

## Term Level queries

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/term-level-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/full-text-queries.html


https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query-phrase.html


:question: How many times is `Hamlet` mentioned as a play in the `shakespeare` index?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Hamlet"
      }
    }
  }
}
```

> Answer: 4244 times

</details>
<hr>

:question: How many times is `Alls well that ends well` mentioned as a play in the `shakespeare` index?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Alls well that ends well"
      }
    }
  }
}
```

> Answer: 3083

</details>
<hr>

:question: How many lines do kings `Henry V` and `Henry VI` have?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "terms": {
      "speaker": ["KING HENRY V", "KING HENRY VI"]
    }
  }
}
```

> Note: the terms have to match exactly, so have to be in uppercase here.

> Answer: 2003

</details>
<hr>

:question: How many lines do all the kings have in the `shakespeare` index?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "wildcard": {
      "speaker": "KING*"
    }
  }
}
```

> Answer: 7250

</details>
<hr>

:question: How many times do people called `Henry` enter the scene?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "regexp": {
      "text_entry.keyword": {
        "value": "Enter.*HENRY.*",
        "flags": "ALL"
      }
    }
  }
}
```

> Answer: 33

</details>
<hr>

:question: How many speechs number in range between `400` and `402` ?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "query": {
    "range": {
      "speech_number": {
        "gte": 400,
        "lte": 402
      }
    }
  }
}
```

> Answer: 10

</details>
<hr>




# Write and execute a search query that is a Boolean combination of multiple queries and filters

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/compound-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html



|Occur|Description|
|----|----|
| must|The clause (query) must appear in matching documents and will contribute to the score.|
|filter|The clause (query) must appear in matching documents. However unlike must the score of the query will be ignored. Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.|
|should|The clause (query) should appear in the matching document.|
|must_not|The clause (query) must not appear in the matching documents. Clauses are executed in filter context meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of 0 for all documents is returned.|



:question: How many times is `Desdemona` mentioned in `Othello` but not by the leading character `Othello` himself?

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "size": 200,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "play_name": "Othello"
          }
        },
        {
          "wildcard": {
            "text_entry.keyword": "*Desdemona*"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "speaker": "OTHELLO"
          }
        }
      ]
    }
  }
}    
```

</details>
<hr>

:question: Filter the previous query results to only show those that also mention the word `sweet`


<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET shakespeare/_search
{
  "size": 200,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "play_name": "Othello"
          }
        },
        {
          "wildcard": {
            "text_entry.keyword": "*Desdemona*"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "speaker": "OTHELLO"
          }
        }
      ],
      "filter": {
        "term": {
          "text_entry": "sweet"
        }
      }
    }
  }
} 
```
</details>
<hr>


# Write an asynchronous search

https://www.elastic.co/guide/en/elasticsearch/reference/current/async-search.html

> The async search API let you asynchronously execute a search request, monitor its progress, and retrieve partial results as they become available.

## POST - Submit async search APIedit
> Executes a search request asynchronously. It accepts the same parameters and request body as the search API.
>
>Returns an `id`.

:warning: These are easy to write but hard to do in the lab as you need so much data to do a slow query, to get the `POST` to return an `id`.

It is better to be aware of the concepts and be able to look this up in the online manuals in the exam.

```json
POST /kibana_sample_data_logs/_async_search?size=0
{
  "sort": [
    { "@timestamp": { "order": "asc" } }
  ],
  "aggs": {
    "sale_date": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1d"
      }
    }
  }
}

// output 

{
  "id" : "FmRldE8zREVEUzA2ZVpUeGs2ejJFUFEaMkZ5QTVrSTZSaVN3WlNFVmtlWHJsdzoxMDc=", 
  "is_partial" : true, 
  "is_running" : true, 
  "start_time_in_millis" : 1583945890986,
  "expiration_time_in_millis" : 1584377890986,
  "response" : {
    "took" : 1122,
    "timed_out" : false,
...
```


## GET async searchedit
> The get async search API retrieves the results of a previously submitted async search request given its `id`. 
>
> If the Elasticsearch security features are enabled, the access to the results of a specific async search is restricted to the user or API key that submitted it.


```json
GET /_async_search/FmRldE8zREVEUzA2ZVpUeGs2ejJFUFEaMkZ5QTVrSTZSaVN3WlNFVmtlWHJsdzoxMDc=
```

## GET async search statusedit
> The get async search status API, without retrieving search results, shows only the status of a previously submitted async search request given its `id`. 
> 
> If the Elasticsearch security features are enabled, the access to the get async search status API is restricted to the monitoring_user role.


```json
GET /_async_search/status/FmRldE8zREVEUzA2ZVpUeGs2ejJFUFEaMkZ5QTVrSTZSaVN3WlNFVmtlWHJsdzoxMDc=
```

## DELETE async searchedit
> You can use the delete async search API to manually delete an async search by `id`. 
>
> If the search is still running, the search request will be cancelled. Otherwise, the saved search results are deleted.

```json
DELETE /_async_search/FmRldE8zREVEUzA2ZVpUeGs2ejJFUFEaMkZ5QTVrSTZSaVN3WlNFVmtlWHJsdzoxMDc=
```




# Write and execute a query that searches across multiple clusters

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-search.html

#TODO:  Need a couple of clusters, though the theory is easy

> Note: just split the remote clusters with commas

> Then append the index to each with a colon

local_index,remote1:index1,remote2:index2

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET local_index,remote_cluster1:that_remote_index,remote_cluster2:that_other_remote_index
{
  "query" : {
    "match_all" : {}
  }
}
```
</details>
<hr>
