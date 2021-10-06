# Searching Data

:warning: These example questions use the eCommerce sample data and the Shakespeare data.


# Prerequisite

:scroll: Download the Shakespeare data, create an index mapping as below and insert the data.

:arrow_down: https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json

(this is also contained in the data folder of this repo)

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

:bulb: `text_entry` is the field we are querying.  `match` is the type of query.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-match-query.html

> Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.
>
> The match query is the standard query for performing a full-text search, including options for fuzzy matching.

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

> **operator**
> (Optional, string) Boolean logic used to interpret text in the query value. Valid values are:
>
> **OR** (Default)
> For example, a query value of capital of Hungary is interpreted as capital OR of OR Hungary.
> **AND**
> For example, a query value of capital of Hungary is interpreted as capital AND of AND Hungary.


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

> Like the match query but used for matching exact phrases or word proximity matches.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-match-query-phrase.html

> The match_phrase query analyzes the text and creates a phrase query out of the analyzed text.

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-multi-match-query.html

> The multi-field version of the match query.

> The multi_match query builds on the match query to allow multi-field queries

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

## Term Level / Phrase queries

> You can use term-level queries to find documents `based on precise values` in structured data. Examples of structured data include date ranges, IP addresses, prices, or product IDs.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/term-level-queries.html


> The full text queries enable you to search analyzed text fields such as the body of an email. The query string is processed using the same analyzer that was applied to the field during indexing.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/full-text-queries.html

> The match_phrase query analyzes the text and creates a phrase query out of the analyzed text.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-match-query-phrase.html


:question: How many times is `Hamlet` mentioned as a play in the `shakespeare` index?

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-terms-query.html

> Returns documents that contain one or more `exact` terms in a provided field.

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

> The `terms` query is the same as the `term` query, except you can search for multiple values.

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

> Note: the terms have to match exactly, so have to be in uppercase here as in the index field data.

> Answer: 2003

</details>
<hr>

:question: How many lines do all the kings have in the `shakespeare` index?

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-wildcard-query.html

> Returns documents that contain terms matching a wildcard pattern.

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-regexp-query.html

> Returns documents that contain terms matching a regular expression.

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

:question: How many speechs range in number between `400` and `402` ?

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-range-query.html

> Returns documents that contain terms within a provided range.

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/compound-queries.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/query-dsl-bool-query.html



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

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/async-search.html

> The async search API let you asynchronously execute a search request, monitor its progress, and retrieve partial results as they become available.

## POST - Submit async search APIedit
> Executes a search request asynchronously. It accepts the same parameters and request body as the `search API`.
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

# Write and execute metric and bucket aggregations

We will be using the `kibana_sample_data_ecommerce` index data for these examples.

Show only the aggs results, and not all of the matches: 
https://www.elastic.co/guide/en/elasticsearch/reference/7.2/returning-only-agg-results.html


## Metric Aggregations

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html

### Common
- Avg Aggregation
- Max Aggregation
- Min Aggregation
- Sum Aggregation
- Stats Aggregation
- Extended Stats Aggregation

### Not so common
- Cardinality Aggregation
- Geo Bounds Aggregation
- Geo Centroid Aggregation
- Median Absolute Deviation Aggregation
- Percentiles Aggregation
- Percentile Ranks Aggregation
- Scripted Metric Aggregation
- Top Hits Aggregation
- Value Count Aggregation
- Weighted Avg Aggregation

<hr>

:question: Pull the number of sales, Max, Min, Average and total sales for the American customers.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics-extendedstats-aggregation.html

> Note: We use `"size": 0` to only return the aggs results and not all the actual matching documents.

> Note: We still need to use the `filter_path` to drill down to our aggs results.

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size": 0,
  "query": {
    "match": {
      "geoip.country_iso_code": "US"
    }
  },
  "aggs": {
    "cart_stats": {
      "extended_stats": {
        "field": "taxful_total_price"
      }
    }
  }
}

\\ output

{
  "aggregations" : {
    "cart_stats" : {
      "count" : 1206,
      "min" : 6.98828125,
      "max" : 344.0,
      "avg" : 76.16962064676616,
      "sum" : 91860.5625,
      "sum_of_squares" : 9216415.566009521,
      "variance" : 1840.3245174012998,
      "std_deviation" : 42.89900368774664,
      "std_deviation_bounds" : {
        "upper" : 161.96762802225945,
        "lower" : -9.628386728727122
      }
    }
  }
}
```
</details>
<hr>

## Bucket Aggregations

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html


- Adjacency Matrix Aggregation
- Auto-interval Date Histogram Aggregation
- Children Aggregation
- Composite Aggregation
- Date Histogram Aggregation
- Date Range Aggregation
- Diversified Sampler Aggregation
- Filter Aggregation
- Filters Aggregation
- Geo Distance Aggregation
- GeoHash grid Aggregation
- GeoTile Grid Aggregation
- Global Aggregation
- Histogram Aggregation
- IP Range Aggregation
- Missing Aggregation
- Nested Aggregation
- Parent Aggregation
- Range Aggregation
- Reverse nested Aggregation
- Sampler Aggregation
- Significant Terms Aggregation
- Significant Text Aggregation
- Terms Aggregation

:question: Display all sales per day

<details>
  <summary>View Solution (click to reveal)</summary>

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      }
    }
  }
}

\\ output


{
  "aggregations" : {
    "date_hist" : {
      "buckets" : [
        {
          "key_as_string" : "2021-03-25T00:00:00.000Z",
          "key" : 1616630400000,
          "doc_count" : 146
        },
        {
          "key_as_string" : "2021-03-26T00:00:00.000Z",
          "key" : 1616716800000,
          "doc_count" : 153
        },
        {
          "key_as_string" : "2021-03-27T00:00:00.000Z",
          "key" : 1616803200000,
          "doc_count" : 143
        },
        ...
```
</details>
<hr>


# Write and execute aggregations that contain sub-aggregations


:question: Display all sales per day broken down by sales category

<details>
  <summary>View Solution (click to reveal)</summary>

> - The `date_histogram` is the Y-axis
> - Then you need to group by category
> - then Sum the sales (X-axis)

Create the `date_histogram`
```json 
todo?
```

Aggregate by the `category` grouping - here you are testing your aggregation.

You should also note the number of categories, so you don't `size` the aggregation too small later on.

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size": 0,
  "aggs": {
    "cat_keyword": {
      "terms": {
        "field": "category.keyword"
      }
    }
  }
}

// output
{
  "aggregations" : {
    "cat_keyword" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Men's Clothing",
          "doc_count" : 2024
        },
        {
          "key" : "Women's Clothing",
          "doc_count" : 1903
        },
        {
          "key" : "Women's Shoes",
          "doc_count" : 1136
        },
        {
          "key" : "Men's Shoes",
          "doc_count" : 944
        },
        {
          "key" : "Women's Accessories",
          "doc_count" : 830
        },
        {
          "key" : "Men's Accessories",
          "doc_count" : 572
        }
      ]
    }
  }
}

```

Put it all together

```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist_y_axis": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      },
      "aggs": {
        "group_by_category": {
          "terms": {
            "field": "category.keyword",
            "size": 8
          },
          "aggs": {
            "total_sales_price": {
              "sum": {
                "field": "taxful_total_price"
              }
            }
          }
        }
      }
    }
  }
}
```

</details>
<hr>

BONUS QUESTION: Order the sales price in descending order

<details>
  <summary>View Solution (click to reveal)</summary>

In the `group_by_category` agg, we need to order by the `total_sales_price` agg.


```json
GET kibana_sample_data_ecommerce/_search?filter_path=aggregations
{
  "size":0,
  "aggs": {
    "date_hist_y_axis": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "1d",
        "min_doc_count": 1
      },
      "aggs": {
        "group_by_category": {
          "terms": {
            "field": "category.keyword",
            "order": {
              "total_sales_price": "desc"
            },
            "size": 8
          },
          "aggs": {
            "total_sales_price": {
              "sum": {
                "field": "taxful_total_price"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>
<hr>



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
