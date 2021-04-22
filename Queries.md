# Queries

These example questions use the eCommerce sample data and the Shakespeare data.


# Prerequisite

Download the Shakespeare data, create an index mapping as below and insert the data.

https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json


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

How many times do the words `brothers`, `blood` appear in the `shakespeare` data?

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

How many times do the words `brothers` and `blood` appear in the `shakespeare` data?

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

How many times does the phrase `brothers blood` appear in the `shakespeare` data?

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

Find how many lines there are, where `Prince Henry` is mentioned or is a speaker in the `shakespeare` data?

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

## Term Level queries


## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/term-level-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/full-text-queries.html



https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query-phrase.html


How many times is `Hamlet` mentioned as a play in the `shakespeare` index?

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



How many times is `Alls well that ends well` mentioned as a play in the `shakespeare` index?

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


How many lines do kings `Henry V` and `Henry VI` have?

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


How many lines do all the kings have in the `shakespeare` index?

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


How many times do people called `Henry` enter the scene?

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

How many speechs number in range between `400` and `402` ?

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






# Write and execute a search query that is a Boolean combination of multiple queries and filters

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/compound-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html



## solution

|Occur|Description|
|----|----|
| must|The clause (query) must appear in matching documents and will contribute to the score.|
|filter|The clause (query) must appear in matching documents. However unlike must the score of the query will be ignored. Filter clauses are executed in filter context, meaning that scoring is ignored and clauses are considered for caching.|
|should|The clause (query) should appear in the matching document.|
|must_not|The clause (query) must not appear in the matching documents. Clauses are executed in filter context meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of 0 for all documents is returned.|



How many times is `Desdemona` mentioned in `Othello` but not by the leading character `Othello` himself?

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

Filter the previous query results to only show those that also mention the word `sweet`

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

# Highlight the search terms in the response of a query

In the spoken lines of the play, highlight the word `Hamlet` starting the highlight with `"#aaa#` and ending it with `#bbb#`

## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-highlighting.html

```json
GET shakespeare/_search
{
    "query": {
        "match": {
            "text_entry":  "Hamlet"
        }
    }, 
    "highlight": {
        "fields":  { 
          "text_entry": {
            "pre_tags": "#aaa#", 
            "post_tags": "#bbb#"
        }
      }
    }
}
```


# Sort the results of a query by a given set of requirements

Return all of `Othellos` lines in reverse order.

## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-sort.html

```json
GET shakespeare/_search
{
    "query": {
        "term": {
            "speaker": "OTHELLO"
          }
    }, 
    "sort": [
      {
        "speech_number": {
          "order": "desc"
        }
      }
    ]
}
```

# Implement pagination of the results of a search query

Paginate the `Othello` play, `20` speech lines per page, stating from line `40`.

What is the first line on this page?

## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-from-size.html

```json
GET shakespeare/_search
{
    "size": 20,
    "from": 40,
    "query": {
        "term": {
            "play_name": "Othello"
          }
    }, 
    "sort": [
      {
        "speech_number": {
          "order": "asc"
        }
      }
    ]
}

// Output

      {
        "_source" : {
          "text_entry" : "Will you think so?"
        }
      }
```

# Apply fuzzy matching to a query

Using fuzzy query, show that a mis-type of the speaker name `OTHELO` still returns the real name `OTHELLO` lines
## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-fuzzy-query.html

```json
GET shakespeare/_search
{
  "query": {
    "fuzzy": {
      "speaker": {
        "value": "OTHELO"
      }
    }
  }
}
```

> Note: this is like the "DId you mean x?" suggestions you see on the web.

# Define and use a search template

Create and use a search template that returns the lines of a person in a play.

Show all lines that belong to `Attendant` in the play `Cymbeline`

## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html


```json
POST _scripts/get_lines
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "play_name": "{{play_name}}"
              }
            },
            {
              "term": {
                "speaker": "{{ speaker }}"
              }
            }
          ]
        }
      }
    }
  }
}
```

```json
GET _scripts/get_lines

// output

{
  "_id" : "get_lines",
  "found" : true,
  "script" : {
    "lang" : "mustache",
    "source" : """{"query":{"bool":{"must":[{"term":{"play_name":"{{play_name}}"}},{"term":{"speaker":"{{ speaker }}"}}]}}}""",
    "options" : {
      "content_type" : "application/json; charset=UTF-8"
    }
  }
}
```

Test

```json
GET _search/template?filter_path=*.*.*.text_entry
{
    "id": "get_lines", 
    "params": {
        "play_name": "Cymbeline",
        "speaker" : "Attendant"
    }
}

// output

{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "text_entry" : "Please you, sir,"
        }
      },
      {
        "_source" : {
          "text_entry" : "Her chambers are all lockd; and theres no answer"
        }
      },
      {
        "_source" : {
          "text_entry" : "That will be given to the loudest noise we make."
        }
      }
    ]
  }
}
```




# Write and execute a query that searches across multiple clusters

#TODO:  Need a couple of clusters, though the theory is easy

> Note: just split the remote clusters with commas
> Then append the index to each with a colon

local_index,remote1:index1,remote2:index2

```json
GET local_index,remote_cluster1:that_remote_index,remote_cluster2:that_other_remote_index
{
  "query" : {
    "match_all" : {}
  }
}
```
