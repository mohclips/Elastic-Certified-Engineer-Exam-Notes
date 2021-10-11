# George Bridgeman Exercises

See https://georgebridgeman.com/exercises/olympic-data/olympics-01/

## Prerequisites

Download CSV from here:
https://www.kaggle.com/heesoo37/120-years-of-olympic-history-athletes-and-results/version/2

Upload into `olympic-events` vai the `data visualiser`

## Skip Exercises 01-03

We don't need these anymore, as we use the data visualiser from July 2021 onwards.

## Exercise 04
Validate that the data was imported correctly by using a single API call to show the index name, index health, number of documents, and the size of the primary store. The details in the response must be in that order, with headers, and for the new index only.

```json
GET _cat/indices/olympic-events?v&h=index,health,docs.count,pri.store.size

// output

index          health docs.count pri.store.size
olympic-events yellow     271116         30.9mb
```

## Exercise 05
The cluster health is yellow. Use a cluster API that can explain the problem.

```json
GET _cluster/allocation/explain

// output

{
  "index" : "olympic-events",
  "shard" : 0,
  "primary" : false,
  "current_state" : "unassigned",
  "unassigned_info" : {
    "reason" : "INDEX_CREATED",
    "at" : "2021-10-11T09:51:47.628Z",
    "last_allocation_status" : "no_attempt"
  },
  "can_allocate" : "no",
  "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions" : [
    {
      "node_id" : "DRGgT8z3SEOPFPqlioq2TQ",
      "node_name" : "esnode",
      "transport_address" : "172.19.0.2:9300",
      "node_attributes" : {
        "xpack.installed" : "true",
        "transform.node" : "false"
      },
      "node_decision" : "no",
      "weight_ranking" : 1,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "a copy of this shard is already allocated to this node [[olympic-events][0], node[DRGgT8z3SEOPFPqlioq2TQ], [P], s[STARTED], a[id=y89Tkcc4T2ahl7nQiZIhxg]]"
        }
      ]
    }
  ]
}
```

## Exercise 06
Change the cluster or index settings as required to get the cluster to a green status.


## view settings

```json
GET olympic-events?filter_path=*.settings

// output

{
  "olympic-events" : {
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "olympic-events",
        "creation_date" : "1633945907613",
        "number_of_replicas" : "1",
        "uuid" : "K-A5nA5BTgGw3gtdH5rh6w",
        "version" : {
          "created" : "7130299"
        }
      }
    }
  }
}

```
### Fix

```json
PUT olympic-events/_settings
{
  "number_of_replicas": 0
}
```

### Check Health

```json
GET _cat/indices/olympic-events?v&h=index,health,docs.count,pri.store.size

//output

index          health docs.count pri.store.size
olympic-events green      271116         30.9mb
```


## Exercise 07
Look at how Elasticsearch has applied very general-purpose mappings to the data. Why has it chosen to use a text type for the Age field? Find all unique values for the Age field; there are less than 100 unique values for the Age field. Look for any suspicious values.


### View

```json
GET olympic-events?filter_path=*.mappings.properties.Age

// output

{
  "olympic-events" : {
    "mappings" : {
      "properties" : {
        "Age" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

### Query

- Find all `unique values` suggests an `aggregation` `(aggs)`
- Set results docs `size:0` as we don't want to see the actual docs that matched
- Use `terms` to group the Ages (unique values)
- Set terms results `size:100` as it says in the question
- Order `desc` so we can see any anomolies.

```json
GET olympic-events/_search?filter_path=aggregations
{
  "size": 0, 
  "aggs": {
    "age_check": {
      "terms": {
        "field": "Age",
        "size": 100,
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}

// output

"aggregations" : {
    "age_check" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "NA",
          "doc_count" : 9474
        },
        {
          "key" : "97",
          "doc_count" : 1
        },
        {
          "key" : "96",
          "doc_count" : 1
        },
        {
          "key" : "88",
          "doc_count" : 3
        },
    ...

```

Suspicious values = `NA`

## Exercise 08
We will be deleting data in the next exercise; making a backup is always prudent. Without making any changes to the data, reindex the olympic-events index into a new index called olympic-events-backup.

```json
POST _reindex?wait_for_completion=false
{
  "source": {"index": "olympic-events"},
  "dest": {"index": "olympic-events-backup"}
}


// output
{
  "task" : "DRGgT8z3SEOPFPqlioq2TQ:2409291"
}
```

### Check task is complete

```json
GET _tasks/DRGgT8z3SEOPFPqlioq2TQ:2409291

// output

{
  "completed" : false,
  "task" : {
    "node" : "DRGgT8z3SEOPFPqlioq2TQ",
    "id" : 2409291,
    "type" : "transport",
    "action" : "indices:data/write/reindex",
    "status" : {
      "total" : 271116,
      "updated" : 0,
      "created" : 96000,
      "deleted" : 0,
      "batches" : 97,
      "version_conflicts" : 0,
      "noops" : 0,
      "retries" : {
        "bulk" : 0,
        "search" : 0
      },
      "throttled_millis" : 0,
      "requests_per_second" : -1.0,
      "throttled_until_millis" : 0
    },
    "description" : "reindex from [olympic-events] to [olympic-events-backup][_doc]",
    "start_time_in_millis" : 1633948147137,
    "running_time_in_nanos" : 15183290753,
    "cancellable" : true,
    "headers" : { }
  }
}

// try again

GET _tasks/DRGgT8z3SEOPFPqlioq2TQ:2409291?filter_path=completed,response

// output

{
  "completed" : true,
  "response" : {
    "took" : 46605,
    "timed_out" : false,
    "total" : 271116,
    "updated" : 0,
    "created" : 271116,
    "deleted" : 0,
    "batches" : 272,
    "version_conflicts" : 0,
    "noops" : 0,
    "retries" : {
      "bulk" : 0,
      "search" : 0
    },
    "throttled" : "0s",
    "throttled_millis" : 0,
    "requests_per_second" : -1.0,
    "throttled_until" : "0s",
    "throttled_until_millis" : 0,
    "failures" : [ ]
  }
}
```



### Check health etc

```json
GET _cat/indices/olympic-events*?v 

// output

health status index                 uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   olympic-events-backup Z6Em5SiQTFm-bexwSTzTIQ   1   1     164947            0     43.4mb         43.4mb
green  open   olympic-events        K-A5nA5BTgGw3gtdH5rh6w   1   0     271116            0     30.9mb         30.9mb

```

WTF? they dont match!


Run a `_count` on each index

```json

GET _cat/count/olympic-events

// output

1633948517 10:35:17 271116

GET _cat/count/olympic-events-backup

// output

1633948529 10:35:29 271116
```

Oh now they do match, try again...

```json
GET _cat/indices/olympic-events*?v 

// output

health status index                 uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   olympic-events-backup Z6Em5SiQTFm-bexwSTzTIQ   1   1     271116            0     41.4mb         41.4mb
green  open   olympic-events        K-A5nA5BTgGw3gtdH5rh6w   1   0     271116            0     30.9mb         30.9mb

```

So, there we have it. Interesting, no?


## Exercise 09
The Height and Weight fields suffer from the same problem as the Age field. Later exercises will require numeric-type queries for these fields so we want to exclude any document we can’t use in our analyses. In a single request, delete all documents from the olympic-events index that have a value of NA for either the Age, Height or Weight field.


Query first to make sure it works properly

```json
GET olympic-events/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "Age": {
              "value": "NA"
            }
          }
        },
        {
          "term": {
            "Height": {
              "value": "NA"
            }
          }
        },
        {
          "term": {
            "Weight": {
              "value": "NA"
            }
          }
        }
      ]
    }
  }
}
```

### The actual delete

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/docs-delete-by-query.html


I used a task here again so i didn't get a timeout.

```json

POST olympic-events/_delete_by_query?wait_for_completion=false
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "Age": {
              "value": "NA"
            }
          }
        },
        {
          "term": {
            "Height": {
              "value": "NA"
            }
          }
        },
        {
          "term": {
            "Weight": {
              "value": "NA"
            }
          }
        }
      ]
    }
  }
}

GET _tasks/DRGgT8z3SEOPFPqlioq2TQ:2418803

// output
{
  "completed" : true,
  "task" : {
    "node" : "DRGgT8z3SEOPFPqlioq2TQ",
    "id" : 2418803,
    "type" : "transport",
    "action" : "indices:data/write/delete/byquery",
    "status" : {
      "total" : 64951,
      "updated" : 0,
      "created" : 0,
      "deleted" : 64951,
      "batches" : 65,
      "version_conflicts" : 0,
      "noops" : 0,
      "retries" : {
        "bulk" : 0,
        "search" : 0
      },
      "throttled_millis" : 0,
      "requests_per_second" : -1.0,
      "throttled_until_millis" : 0
    },
    "description" : "delete-by-query [olympic-events]",
    "start_time_in_millis" : 1633949092612,
    "running_time_in_nanos" : 21303050970,
    "cancellable" : true,
    "headers" : { }
  },
  "response" : {
    "took" : 21302,
    "timed_out" : false,
    "total" : 64951,
    "updated" : 0,
    "created" : 0,
    "deleted" : 64951,
    "batches" : 65,
    "version_conflicts" : 0,
    "noops" : 0,
    "retries" : {
      "bulk" : 0,
      "search" : 0
    },
    "throttled" : "0s",
    "throttled_millis" : 0,
    "requests_per_second" : -1.0,
    "throttled_until" : "0s",
    "throttled_until_millis" : 0,
    "failures" : [ ]
  }
}
```

Note: `task.status.deleted = 64951`


## Exercise 10
Notice how the Games field contains both the Olympic year and season. Create an ingest pipeline called split_games that will split this field into two new fields - year and season - and remove the original Games field.

:bulb: I strongly suggest you do this in Kibana

# Get a test document
```json
GET olympic-events/_search?filter_path=hits.hits
{
  "size": 1, 
  "query": {
    "match_all": {}
  }
}

// output

{
  "hits" : {
    "hits" : [
      {
        "_index" : "olympic-events",
        "_type" : "_doc",
        "_id" : "8sfEbnwBlYfrBPPFIjEe",
        "_score" : 1.0,
        "_source" : {
          "NOC" : "MEX",
          "Sex" : "M",
          "City" : "London",
          "Weight" : "70",
          "Name" : "Javier Corts Granados",
          "Sport" : "Football",
          "Year" : 2012,
          "Games" : "2012 Summer",
          "Event" : "Football Men's Football",
          "Height" : "171",
          "Team" : "Mexico",
          "ID" : 23206,
          "Medal" : "Gold",
          "Season" : "Summer",
          "Age" : "23"
        }
      }
    ]
  }
}
```
Use this as your test data in kibana

### Kibana

`Stack Management -> Ingest Node Pipelines -> Create Pipeline`


- Create each processor at a time
- Test the document against the newly created processor and check the results.
- There are numerous ways to skin a cat - mine follows below
- There is a `Year` and `Season` feild alreasy, but we have been asked to create `year` and `season` (all lowercase)

```json
PUT _ingest/pipeline/split_games
{
  "description": "Create an ingest pipeline called split_games that will split this field into two new fields - year and season - and remove the original Games field.",
  "processors": [
    {
      "split": {
        "field": "Games",
        "separator": "\\s+",
        "target_field": "_split_games"
      }
    },
    {
      "set": {
        "field": "year",
        "value": "{{{_split_games.0}}}"
      }
    },
    {
      "set": {
        "field": "season",
        "value": "{{{_split_games.1}}}"
      }
    },
    {
      "remove": {
        "field": [
          "_split_games",
          "Games"
        ]
      }
    }
  ]
}
```


## Exercise 11
Ensure your new pipeline is working correctly by simulating it with these values:

1998 Summer
2014 Winter

:bulb: skip this part if you used Kibana in the previous question.  Or have a go anyway...

```json
POST _ingest/pipeline/split_games/_simulate
{
  "docs": [
    {
      "_source": {
        "Games": "1998 Summer"
      }
    },
    {
      "_source": {
        "Games": "2014 Winter"
      }
    }
  ]
}

// output

{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "season" : "Summer",
          "year" : "1998"
        },
        "_ingest" : {
          "timestamp" : "2021-10-11T11:15:46.673505565Z"
        }
      }
    },
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "season" : "Winter",
          "year" : "2014"
        },
        "_ingest" : {
          "timestamp" : "2021-10-11T11:15:46.673512641Z"
        }
      }
    }
  ]
}
```

## Exercise 12
We’ll now start to clean up the mappings. Create a new index called olympic-events-fixed with 1 shard, 0 replicas, and the following mapping:

|Field|	Type|
|---|---|
|athleteId|	integer|
|age|	short|
|height|	short|
|weight|	short|
|athleteName|	text + keyword|
|gender|	keyword|
|team|	keyword|
|noc|	keyword|
|year|	short|
|season|	keyword|
|city|	text + keyword|
|sport|	keyword|
|event|	text + keyword|
|medal|	keyword|


Quicker to do this in the Kibana GUI

`Stack Management -> Index Management -> Templates -> Create template`

```json
{
  "settings": {
    "index": {
      "number_of_replicas": "0"
    }
  },
  "mappings": {
    "properties": {
      "age": {
        "type": "short"
      },
      "athleteId": {
        "type": "integer"
      },
      "athleteName": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "city": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "event": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "gender": {
        "type": "keyword"
      },
      "height": {
        "type": "short"
      },
      "medal": {
        "type": "keyword"
      },
      "noc": {
        "type": "keyword"
      },
      "season": {
        "type": "keyword"
      },
      "sport": {
        "type": "keyword"
      },
      "team": {
        "type": "keyword"
      },
      "weight": {
        "type": "short"
      },
      "year": {
        "type": "short"
      }
    }
  },
  "aliases": {}
}
```


## Exercise 13
Reindex the data in the olympic-events index into the new olympic-events-fixed index created in exercise 12 using the split_games pipeline created in exercise 10.

```json
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "olympic-events"
  },
  "dest": {
    "index": "olympic-events-fixed",
    "op_type": "create",
    "pipeline": "split_games"
  }
}

GET _tasks/DRGgT8z3SEOPFPqlioq2TQ:2492188

// wait for completion

// check results

GET olympic-events-fixed/_search
{
  "query": { "match_all": {}}
}
```



## Exercise 14
Look at the mapping for the olympic-events-fixed index. Notice how Elasticsearch has created new fields. We created the mapping for this index with the same field names as before but we put all the field names in lowercase. Field names are case sensitive, so Age and age are different, distinct fields to Elasticsearch.

Also notice that the new mapping uses athleteId instead of ID, athleteName instead of Name and gender instead of Sex.

We’ll need to correct this by tearing down the new index and reindexing with an additional pipeline to use the correct field names. To save us constantly having to recreate the index with the right mappings, we can leverage index templates.

Create an index template called olympic-events for new indices with a name beginning with olympic-events-. Use the mapping and settings we defined in exercise 12 and configure the mapping so Elasticsearch will throw an exception if a document contains a field not defined in the mapping.

```json
GET olympic-events-fixed?filter_path=*.mappings
```

Take the newly created template and update that for speed.

- update `mappings.dynamic:strict`
- update `index-patterns`

```json
PUT _template/olympic-events
{
  "index_patterns": [ "olympic-events-*"], 
  "settings": {
    "index": {
      "number_of_replicas": "0"
    }
  },
  "mappings": {
    "dynamic" : "strict",
    "properties": {
      "age": {
        "type": "short"
      },
      "athleteId": {
        "type": "integer"
      },
      "athleteName": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "city": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "event": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "gender": {
        "type": "keyword"
      },
      "height": {
        "type": "short"
      },
      "medal": {
        "type": "keyword"
      },
      "noc": {
        "type": "keyword"
      },
      "season": {
        "type": "keyword"
      },
      "sport": {
        "type": "keyword"
      },
      "team": {
        "type": "keyword"
      },
      "weight": {
        "type": "short"
      },
      "year": {
        "type": "short"
      }
    }
  }
}

```



## Exercise 15
Create a new ingest pipeline called reconcile_fields to replace all fields with their correct field names (except for the Games field), then also execute the split_games pipeline.

Again, do this in Kibana GUI.  You can duplicate pipeline processors with the elipsis dots menu at the right hand side.
Also pipelines can all other pipelines.


```json
PUT _ingest/pipeline/reconcile_fields
{
  "processors": [
    {
      "rename": {
        "field": "NOC",
        "target_field": "noc"
      }
    },
    {
      "rename": {
        "field": "Sex",
        "target_field": "sex"
      }
    },
    {
      "rename": {
        "field": "City",
        "target_field": "city"
      }
    },
    {
      "rename": {
        "field": "Weight",
        "target_field": "weight"
      }
    },
    {
      "rename": {
        "field": "Name",
        "target_field": "athleteName"
      }
    },
    {
      "rename": {
        "field": "Sport",
        "target_field": "sport"
      }
    },
    {
      "rename": {
        "field": "Event",
        "target_field": "event"
      }
    },
    {
      "rename": {
        "field": "Height",
        "target_field": "height"
      }
    },
    {
      "rename": {
        "field": "Team",
        "target_field": "team"
      }
    },
    {
      "rename": {
        "field": "ID",
        "target_field": "athleteId"
      }
    },
    {
      "rename": {
        "field": "Medal",
        "target_field": "medal"
      }
    },
    {
      "rename": {
        "field": "Age",
        "target_field": "age"
      }
    },
    {
      "pipeline": {
        "name": "split_games"
      }
    }
  ]
}
```

## Exercise 16
Test your new pipeline with the following document:

```json
{
  "NOC": "ARG",
  "Sex": "M",
  "City": "Los Angeles",
  "Weight": "98",
  "Name": "Ernesto Arturo Alas",
  "Sport": "Shooting",
  "Games": "1984 Summer",
  "Event": "Shooting Men's Free Pistol, 50 metres",
  "Height": "186",
  "Team": "Argentina",
  "ID": 2224,
  "Medal": "NA",
  "Age": "54"
}
```

Do this in the kibana GUI.   You should have done this in the previous step anyway.

```json
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_type": "_doc",
        "_id": "_id",
        "_source": {
          "noc": "ARG",
          "city": "Los Angeles",
          "year": "1984",
          "sex": "M",
          "weight": "98",
          "team": "Argentina",
          "athleteId": 2224,
          "medal": "NA",
          "season": "Summer",
          "athleteName": "Ernesto Arturo Alas",
          "event": "Shooting Men's Free Pistol, 50 metres",
          "sport": "Shooting",
          "age": "54",
          "height": "186"
        },
        "_ingest": {
          "timestamp": "2021-10-11T14:00:58.537616358Z"
        }
      }
    }
  ]
}
```


## Exercise 17
Delete the olympic-events-fixed index.

```json
DELETE olympic-events-fixed
```

## Exercise 18
Reindex the data in the olympic-events index into a new olympic-events-fixed index using the reconcile_fields pipeline. If Elasticsearch throws any exceptions, you may have missed a field in your pipeline.

```json
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "olympic-events"
  },
  "dest": {
    "index": "olympic-events-fixed",
    "op_type": "create",
    "pipeline": "reconcile_fields"
  }
}
```

```json
GET _tasks/DRGgT8z3SEOPFPqlioq2TQ:2516649?filter_path=completed,response.failures

// output

{
  "completed" : true,
  "response" : {
    "failures" : [ ]
  }
}
```

Check mappings

```json
GET olympic-events-fixed/_search
{
  "size": 1,
  "query": {
    "match_all": {}
  }
}

// output

    "hits" : [
      {
        "_index" : "olympic-events-fixed",
        "_type" : "_doc",
        "_id" : "CMfEbnwBlYfrBPPFGx5y",
        "_score" : 1.0,
        "_source" : {
          "noc" : "KEN",
          "city" : "Sydney",
          "year" : "2000",
          "sex" : "M",
          "weight" : "53",
          "team" : "Kenya",
          "Year" : 2000,
          "athleteId" : 20522,
          "medal" : "NA",
          "season" : "Summer",
          "Season" : "Summer",
          "athleteName" : "Kenneth Cheruiyot",
          "event" : "Athletics Men's Marathon",
          "sport" : "Athletics",
          "age" : "26",
          "height" : "173"
        }
      }
    ]

```
Note: that the data from Kaggle contians `Season`, `Year` already so these have been duplicated.  

## Exercise 19
Write a single query to find the name of all Gymnastics events. There are less than 100 Gymnastics event types.

```json
GET olympic-events-fixed/_search?filter_path=aggregations
{
  "query": {
    "match": {
      "sport": "Gymnastics"
    }
  }, 
  "aggs": {
    "get_gymnastics": {
      "terms": {
        "field": "event.keyword",
        "size": 100
      }
    }
  }
}
```

## Exercise 20
Write a single query to find the average weight for male and female competitors in Gymnastics events.


```json
GET olympic-events-fixed/_search?filter_path=aggregations
{
  "query": {
    "match": {
      "sport": "Gymnastics"
    }
  }, 
  "aggs": {
    "get_gymnastics": {
      "terms": {
        "field": "event.keyword",
        "size": 100
      },
      "aggs": {
        "avg_weight": {
          "avg": {
            "field": "weight"
          }
        }
      }
      
    }
    
  }
}
```

## Exercise 21
Write a single query to find the year that each of the 590 unique events first appeared in the Olympic Games, and which events were introduced most recently.

```json
GET olympic-events-fixed/_search?filter_path=aggregations
{
  "aggs": {
    "events": {
      "terms": {
        "field": "event.keyword"
        
      },
      "aggs": {
        "year": {
          "min": {
            "field": "year"
          }
          
        },
        "year_bucket_sort": {
          "bucket_sort": {
            "sort": [
              { "year": { "order": "desc" } } 
            ],
            "size": 10                                
          }
        }
      }
    }
  }
}


// output

{
  "aggregations" : {
    "events" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 182220,
      "buckets" : [
        {
          "key" : "Volleyball Men's Volleyball",
          "doc_count" : 1782,
          "year" : {
            "value" : 1964.0
          }
        },
        {
          "key" : "Basketball Men's Basketball",
          "doc_count" : 2461,
          "year" : {
            "value" : 1936.0
          }
        },
        {
          "key" : "Handball Men's Handball",
          "doc_count" : 2036,
          "year" : {
            "value" : 1936.0
          }
        },
        {
          "key" : "Cycling Men's Road Race, Individual",
          "doc_count" : 2099,
          "year" : {
            "value" : 1924.0
          }
        },
...
```

I found this one a little ambiguous, `which events were introduced` is plural.   So i sorted the year.

## Exercise 22
Write a query to return only the following fields for the 50 tallest athletes in the 2016 Rio de Janeiro Games:

- athleteName
- team
- sport
- age
- height
- weight
- gender

Build the query
```json
GET olympic-events-fixed/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
          "city": "Rio de Janeiro"
          }
        },
        {
          "match": {
            "year": "2016"
          }
        }
      ]
    }
  }
}
```

Add the sort and restrict the size

```json
GET olympic-events-fixed/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "Rio de Janeiro"
          }
        },
        {
          "match": {
            "year": "2016"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "height": {
        "order": "desc"
      }
    }
  ],
  "size": 50
}
```

Restrict the fields

```json
GET olympic-events-fixed/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "Rio de Janeiro"
          }
        },
        {
          "match": {
            "year": "2016"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "height": {
        "order": "desc"
      }
    }
  ],
  "size": 50,
  "_source": false,
  "fields": [
    "athleteName",
    "team",
    "sport",
    "age",
    "height",
    "weight",
    "gender"
  ]
}
```

## Exercise 23
The weight and height fields are in metric. Weight is in kg and height is in cm. Add a scripted field called weightLbs to the previous query to return the weight in lbs. The formula for this is: Weight * 2.2

```json
GET olympic-events-fixed/_search
{
  "script_fields": {
    "weightLbs": {
      "script": {
        "lang": "painless",
        "source": "doc['weight'].value * 2.2"
      }
    }
  },
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "Rio de Janeiro"
          }
        },
        {
          "match": {
            "year": "2016"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "height": {
        "order": "desc"
      }
    }
  ],
  "size": 50,
  "_source": false,
  "fields": [
    "athleteName",
    "team",
    "sport",
    "age",
    "height",
    "weight",
    "gender",
    "weightLbs"
  ]
}


// output

{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "olympic-events-fixed",
        "_type" : "_doc",
        "_id" : "HsjEbnwBlYfrBPPFZp4L",
        "_score" : null,
        "fields" : {
          "weightLbs" : [
            253.00000000000003
          ],
          "weight" : [
            115
          ],
```

:bulb: WTF?! moment.  `115 * 2.2 = 253`, AND NOT `253.00000000000003`  !!!!!



## Exercise 24
Add a scripted field called bmi to the previous query to return the BMI for each athlete, calculated using the following formula: Weight / (Height in m squared)t

```json
GET olympic-events-fixed/_search
{

  "script_fields": {
    "bmi": {
      "script": {
        "lang": "painless",
        "source": """
        doc['weight'].value /  Math.pow( doc['height'].value / 100, 2 )
        """
      }
    },
    "weightLbs": {
      "script": {
        "lang": "painless",
        "source": "doc['weight'].value * 2.2"
      }
    }
  },
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "Rio de Janeiro"
          }
        },
        {
          "match": {
            "year": "2016"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "height": {
        "order": "desc"
      }
    }
  ],
  "size": 50,
  "_source": false,
  "fields": [
    "athleteName",
    "team",
    "sport",
    "age",
    "height",
    "weight",
    "gender",
    "weightLbs"
  ]
}
```




## Exercise 25
Write a query to return the first 50 documents for gold medal athletics events, in descending age order.

```json
GET olympic-events-fixed/_search
{
  "size" :50,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "sport": "Athletics"
          }
        },
        {
          "match": {
            "medal": "Gold"
          }
          
        }
      ]
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}
```

## Exercise 26
Write a query to match swimming events where either:

The athlete’s weight was between 60kg and 70kg
The athlete’s age was less than 20
Enhance the query so the results identify whether the weight, age, or both matched the search criteria.

:bulb:  The question doesnt match the data very well nor the answer.

Try this:

- The athlete’s weight was between 75kg and 90kg
- The athlete’s age was greater than 30
- Enhance the query so the results identify whether the weight, age, or both matched the search criteria.
- First 50 results
- Only return, age, weight, sport.

```json
GET olympic-events-fixed/_search
{
  "size" :50,
  "query": {
    "bool": {
      "minimum_should_match": 1, 
      "should": [
        {
          "range": {
            "weight": {
              "_name" : "weight",
              "gte": 75,
              "lte": 90
            }
          }
        },
        {
          "range": {
            "age": {
              "_name" : "age",
              "gt": 30
            }
          }
        }
      ],
      "filter": [
        {
          "match": {
            "sport": "Swimming"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "weight": {
        "order": "desc"
      }
    }
  ]
,
"_source": ["age", "weight","sport"]
}
```




## Exercise 27
Create a new index for the National Olympic Committees. Call it olympic-noc-regions, with 1 primary shard and 0 replica shards. Ensure it has the following fields:

noc (keyword)
region (keyword)
notes (text)

## Exercise 28
Index the data containing information about the National Olympic Committees using the Bulk API by following these steps:

https://www.kaggle.com/heesoo37/120-years-of-olympic-history-athletes-and-results/version/2?select=noc_regions.csv

Use the data visualiser in Kibana


## Exercise 29
Create an enrich policy and ingest pipeline that uses the enrich processor to add details of the National Olympic Committee to each document in the olympic-events-fixed index. Call the policy olympic-noc-append and the pipeline enrich-noc. Add details of the matching NOC entry to a new field called nocDetails, matching on the noc field.



## Exercise 30
Create a new index called olympic-events-enriched, into which we can reindex the Olympic events but with some enriched fields. Change the mapping settings for the new index so we can add fields dynamically.



## Exercise 31
Reindex the olympic-events-fixed index into olympic-events-enriched, running it through the enrich-noc ingest pipeline. Once complete, verify the new field was added to the olympic-events-fixed index, and populated with details of the associated NOC.
