# Data Management

# Define an index that satisfies a given set of requirements

:question: Create an index with the following requirements:

- **Name**: accounts-raw
- **Shards**: 1
- **Replicas**: 0

<details>
  <summary>View Solution (click to reveal)</summary>

```json
PUT accounts-raw
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

Check details:

```json
GET accounts-raw/_settings

// Output 

{
  "accounts-raw" : {
    "settings" : {
      "index" : {
        "creation_date" : "1618482826116",
        "number_of_shards" : "1",
        "number_of_replicas" : "0",
        "uuid" : "uHiOrOZsQUC7oAyeIzZKUg",
        "version" : {
          "created" : "7020099"
        },
        "provided_name" : "accounts-raw"
      }
    }
  }
}
```
</details>
<hr>

# Use the Data Visualizer to upload a text file into Elasticsearch

This is a bit of point and click.

https://www.elastic.co/blog/importing-csv-and-log-data-into-elasticsearch-with-file-data-visualizer

You have to make sure that Machine Learning is switch on (default) or this functionality will be missing.

In elasticsearch.yml: `xpack.ml.enabled=true`



1. Download accounts data from github repo

    https://github.com/mohclips/Elastic-Certified-Engineer-Exam-Notes/blob/main/data/accounts.csv

2. Import into an index called `accounts-import`


# Define and use an index template for a given pattern that satisfies a given set of requirements

## Part 1

:question: Create an index template called `accounts-tmpl` with the following requirements:

    "account_number" : integer,
    "balance" : long,
    "firstname" : keyword,
    "lastname" : keyword,
    "age" : integer,
    "gender" : keyword,
    "address" : text,
    "employer" : text / keyword,
    "email" : keyword,
    "city" : keyword,
    "state" : keyword


<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html

> Templates are only applied at index creation time. Changing a template will have no impact on existing indices. When using the create index API, the settings/mappings defined as part of the create index call will take precedence over any matching settings/mappings defined in the template.

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html

> text, keyword, date, long, double, boolean or ip.

> object or nested.

> geo_point, geo_shape, or completion.

### Text vs Keyword

- text - a string broken up into individual words, that are searchable.
- keyword - a fixed string (not broken up) such as email addresses.  Only searchable by their _exact_ value.  Usually used in `filters`.

> Loosely: Text is a bunch of text,  Keyword is a single 'word' used as a 'key'


```json
PUT _template/accounts-tmpl
{
  "index_patterns": ["accounts-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
    "account_number" : { "type": "integer" },
    "balance" :  { "type": "integer" },
    "firstname" :  { "type": "keyword" },
    "lastname" :  { "type": "keyword" },
    "age" :  { "type": "integer" },
    "gender" :  { "type": "keyword" },
    "address" :  { "type": "text" },
    "employer" :  { "type": "keyword" },
    "email" :  { "type": "keyword" },
    "city" :  { "type": "keyword" },
    "state" :  { "type": "keyword" }
    }
  }
}
```

:warning: `"_source": { "enabled": false },` will BREAK any re_indexing you may want to do later on.

</details>
<hr>

## Part 2

:question: Import the account data into a new index called `accounts-new`

<details>
  <summary>View Solution (click to reveal)</summary>

```bash
$ curl -u "elastic:Password01" -s -H "Content-Type: application/x-ndjson" -XPUT localhost:9200/accounts-new/_bulk --data-binary "@accounts.json"; echo
```

Check the mappings of this new index

```json
GET accounts-new\_mapping

// Output 

{
  "accounts-new" : {
    "mappings" : {
      "_source" : {
        "enabled" : false
      },
      "properties" : {
        "account_number" : {
          "type" : "integer"
        },
        "address" : {
          "type" : "text"
        },
        "age" : {
          "type" : "integer"
        },
        "balance" : {
          "type" : "integer"
        },
        "city" : {
          "type" : "keyword"
        },
        "email" : {
          "type" : "keyword"
        },
        "employer" : {
          "type" : "keyword"
        },
        "firstname" : {
          "type" : "keyword"
        },
        "gender" : {
          "type" : "keyword"
        },
        "lastname" : {
          "type" : "keyword"
        },
        "state" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

### Test
How many account holders work for `Zork`

```json
POST accounts-new\_search?filter_path=hits.total.value
{
  "query": {
    "match": {
      "employer": "Zork"
    }
  }
}

// Output 

{
  "hits" : {
    "total" : {
      "value" : 1
    }
  }
}
```
</details>
<hr>

# Define and use a dynamic template that satisfies a given set of requirements

:question: Create a new index with a dynamic mapping `accounts-fullname` where the `firstname` and `lastname` fields are combined to create the `fullname` field.

:anger: This does not work. #FIXME:

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html

> Dynamic templates allow you to define custom mappings that can be applied to dynamically added fields based on:
> - the datatype detected by Elasticsearch, with match_mapping_type.
> - the name of the field, with match and unmatch or match_pattern.
> - the full dotted path to the field, with path_match and path_unmatch.

```json
PUT accounts_fullname
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "*name",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}
```

Okay, so this doesn't work, nor does the example on the website (below)

```json
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "name.*",
          "path_unmatch": "*.middle",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}

PUT my_index/_doc/1
{
  "name": {
    "first":  "John",
    "middle": "Winston",
    "last":   "Lennon"
  }
}

GET my_index/_doc/1

POST my_index/_search
{
  "query": {
    "match_all" :{}
  
  }
}
```
you will find full_name field is not created?!?


#FIXME: find an example that actually works!

</details>
<hr>



# Define an Index Lifecycle Management policy for a time-series index

https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html


:warning: It is better to define the ILM via the Web UI, than do it within the Dev Console.  Rick Raposa does exactly that in one of his exam prep guides.

:warning: What you have to get right is the rollover to next step process.

> **Example from Rich Raposa**
> 
> * the corresponding index template is called **task3**
> * the data is hot for 3 minutes, then immediately rolls over to warm
> * the data is then warm for 5 minutes, then rolls over to cold
> * 10 minutes after rolling over, the data is deleted.
> ...
> "This is worth at least 5-6 points of the task"


```json
PUT _ilm/policy/task3
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "3m"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "0m",
        "actions": {
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "5m",
        "actions": {
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "10m",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      }
    }
  }
}
```

# Test an ILM policy

This builds a test policy and index to verify index rotation

## Create ILM template

As far as i can tell ILM cron runs every 5-10mins.  So doing anything less than this does not work

```json
PUT _ilm/policy/test-ilm
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "5m"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "10m",
        "actions": {
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "15m",
        "actions": {
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "30m",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      }
    }
  }
}
```

## Create an index template (not a data_stream)

```json
PUT _index_template/test-ilm-tmpl
{
  "template": {
    "settings": {
      "index": {
        "number_of_shards": 1,
        "number_of_replicas" : 0,
        "lifecycle": {
          "name": "test-ilm",
          "rollover_alias": "test-index"
        }
      }
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "field": {
          "type": "text"
        }
      }
    }
  },
  "index_patterns": [
    "test-index-*"
  ],
  "composed_of": []
}
```


## Bootstrap the initial index
:bulb: This is very important - do this before data ingest

See https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html#ilm-gs-alias-bootstrap

:bulb: Needs to have the aliases added or it won't work!

```json
PUT test-index-000000
{
  "aliases": {
    "test-index": {
      "is_write_index": true
    }
  }
}
```


## Add a doc or two

```json
PUT test-index/_doc/1
{
  "@timestamp" : "2021-10-04T16:26:00.000Z",
  "field":"test data"
}

PUT test-index/_doc/2
{
  "@timestamp" : "2021-10-04T16:58:00.000Z",
  "field":"test data"
}
```

## Check index

```json
GET test-index-000000
```

## Check alias

```json
GET test-index
```

## View ILM phase for each index (rinse and repeat here)

At this point you will have indicies being created and rotated
keep requerying this and see that they are.

```json
GET test-index/_ilm/explain?filter_path=*.*.age,*.*.phase
```

### Results

So, importantly what you can see here is that the index is rolled over at 5-ish minutes.

Then, each move to a new phase is from that initial rollover (at 5-ish minutes)

You can also see that no time is exact.  So days/hours are a better time frame than minutes in production. (at least ths is what i saw).

```json
// output 

{
  "indices" : {
    "test-index-000003" : {
      "age" : "39.65m",
      "phase" : "delete"
    },
    "test-index-000004" : {
      "age" : "29.64m",
      "phase" : "cold"
    },
    "test-index-000005" : {
      "age" : "19.65m",
      "phase" : "warm"
    },
    "test-index-000006" : {
      "age" : "9.65m",
      "phase" : "hot"
    },
  }
}
```

# Another example for timeseries data
## 1 .define an ILM policy to the following requirements
- ILM policy name `timeseries_policy`

Rollover at:
- Max size of 50GB
- 30 days

Delete at:
- 90 days



```json
PUT _ilm/policy/timeseries_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50GB", 
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

## index creation

2. Create a index template with the following requirements

- index pattern `timeseries-*`
- ILM policy `timeseries_policy`
- 1x shard, 0x replicas

```json
PUT _index_template/timeseries_template
{
  "index_patterns": ["timeseries-*"],                 
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "timeseries_policy",      
      "index.lifecycle.rollover_alias": "timeseries"    
    }
  }
}
```

### Bootstrap index
```json
PUT timeseries-000001
{
  "aliases": {
    "timeseries": {
      "is_write_index": true
    }
  }
}
```

### Add a document
write a document to the alias

```json
POST timeseries/_doc
{
  "message": "logged the request",
  "@timestamp": "1591890611"
}
```

## Force a rollover

```json
POST timeseries/_rollover
```

## Check index alias write status


```json
GET _alias/timeseries

// output

{
  "timeseries-000002" : {
    "aliases" : {
      "timeseries" : {
        "is_write_index" : true
      }
    }
  },
  "timeseries-000001" : {
    "aliases" : {
      "timeseries" : {
        "is_write_index" : false
      }
    }
  }
}

```

### View document count

```json
GET _cat/indices/time*?v

// output

health status index             uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   timeseries-000002 bmzwZVAfToqwdbB58JBgQg   1   0          0            0       208b           208b
green  open   timeseries-000001 0Lt9rjDxTAOi2DigXL7cXw   1   0          1            0      4.6kb          4.6kb

```



# Define an index template that creates a new data stream

## What is a datastream
https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html

> A data stream lets you store append-only time series data across multiple indices while giving you a single named resource for requests. _Data streams are well-suited for logs, events, metrics, and other continuously generated data._

:warning: __Use for data that doesn't need updating__ - So __not__ for alarms that open/close or support tickets, etc.  Use an index and alias for that.

> ### Datastream Write index
>
> The most recently created backing index is the data stream’s write index. The stream adds new documents to this index _only_.
>
> You cannot add new documents to other backing indices, even by sending requests directly to the index.
>
> You also cannot perform operations on a write index that may hinder indexing, such as:
>
> * Clone
> * Delete
> * Freeze
> * Shrink
> * Split

> ### Data-streams are Append-only (mainly)
> Data streams are designed for use cases where existing data is rarely, if ever, updated. You cannot send update or deletion requests for existing documents directly to a data stream. Instead, use the update by query and delete by query APIs.
> 
> If needed, you can update or delete documents by submitting requests directly to the document’s backing index.
> 
> If you frequently update or delete existing documents, use an index alias and index template instead of a data stream. You can still use ILM to manage indices for the alias.

https://www.elastic.co/guide/en/elasticsearch/reference/current/set-up-a-data-stream.html

> To set up a data stream, follow these steps:
> 
> Step 1. Create an index lifecycle policy
> 
> Step 2. Create component templates
> 
> Step 3. Create an index template
> 
> Step 4. Create the data stream
> 
> Step 5. Secure the data stream


## 1. Create an ILM policy

This is a normal ILM policy - just like those above

```json
PUT _ilm/policy/ds_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50GB", 
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

## 2. Create the component templates

Use a `wildcard` type to later extract other data from the message field. (not required for DS, but useful)

### Mappings

Creates a component template for mappings

```json

PUT _component_template/my-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "wildcard"
        }
      }
    }
  },
  "_meta": {
    "description": "Mappings for @timestamp and message fields",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

### The component template
Creates a component template for index settings

```json
PUT _component_template/my-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "ds-policy"
    }
  },
  "_meta": {
    "description": "Settings for ILM",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

## 3. Create the combined index template 
Based on the components already created above

:bulb: Note: the `"data_stream": { },` this defines the index as a datastream.

```json
PUT _index_template/my-component-index-template
{
  "index_patterns": ["my-data-stream*"],
  "data_stream": { },
  "composed_of": [ 
    "my-mappings", 
    "my-settings" 
  ],
  "priority": 500,
  "_meta": {
    "description": "Template for my time series data",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

## 4. Create the data stream
By adding a document to it.

:bulb: There is no need to bootstrap the DS index as was done in the index/alias version above.

```json
POST my-data-stream/_doc
{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "message": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
}

// output

{
  "_index" : ".ds-my-data-stream-2021.06.27-000001",
  "_type" : "_doc",
  "_id" : "fMjrTnoBw1F50El3ljBq",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### View document count

```json
GET _cat/indices/.ds-my-data-stream-2021.06.27-000001?v

// output

health status index                                uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .ds-my-data-stream-2021.06.27-000001 none_iX6QbasrQSlT7eTZg   1   1          1            0        4kb            4kb
```

## 5. Secure a datastream

Usual privileges apply.

Aliases, filtered aliases and all the user privileges can be used (eg. from Kibana admin)
