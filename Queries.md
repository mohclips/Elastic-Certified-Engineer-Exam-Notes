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

Find how many lines there are where `Prince Henry` is mentioned or is a speaker in the `shakespeare` data?

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


## Term Level queries


## solution

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/term-level-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/full-text-queries.html



https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query-phrase.html









# Write and execute a search query that is a Boolean combination of multiple queries and filters


# Highlight the search terms in the response of a query


# Sort the results of a query by a given set of requirements


# Implement pagination of the results of a search query


# Apply fuzzy matching to a query


# Define and use a search template


# Write and execute a query that searches across multiple clusters


