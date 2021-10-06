# Cluster Management


# Diagnose shard issues and repair a cluster's health

:bulb: Pre-requistes:  Add a broken index (this is more about the diagnosis commands than the actual broken index)

```json
# delete if already in place
DELETE broken_index

# add a new "broken" index
PUT broken_index
{
  "settings": {
    "number_of_replicas": 2,
    "number_of_shards": 2
  }
}

# add a document
PUT broken_index/_doc/1
{
  "field1" : "data"
}
      
```

## Check health

:question: Check the Cluster health

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cluster-health.html

```json
GET _cluster/health

// output 

{
  "cluster_name" : "elastic-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 48,
  "active_shards" : 48,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 16,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 75.0
}
```

## Or

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cat-health.html

```json
GET _cat/health?v

// output

epoch      timestamp cluster         status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1633541301 17:28:21  elastic-cluster yellow          1         1     48  48    0    0       16             0                  -                 75.0%

```
</details>
<hr>

## Find broken indices

index health usually matches the cluster health

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cat-indices.html

> **Query Parameters**
> **health**
> (Optional, string) Health status used to limit returned indices. Valid values are:
>
> - green
> - yellow
> - red

```json
GET _cat/indices?health=yellow

// output

yellow open broken_index               xHfY0EcGRq-RRZv_cQONCw 2 2      1 0     4kb     4kb

```
</details>
<hr>

:question: Diagnose the fault in index `broken_index`

<details>
  <summary>View Solution (click to reveal)</summary>

Explain the index health

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cluster-allocation-explain.html

```json
GET _cluster/allocation/explain
{
  "index": "broken_index"
}

// output

{
  "error" : {
    "root_cause" : [
      {
        "type" : "action_request_validation_exception",
        "reason" : "Validation Failed: 1: shard must be specified;2: primary must be specified;"
      }
    ],
    "type" : "action_request_validation_exception",
    "reason" : "Validation Failed: 1: shard must be specified;2: primary must be specified;"
  },
  "status" : 400
}

```
</details>
<hr>

:question:  View the shard allocation for the index

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cat-shards.html

```json

GET _cat/shards/broken_index?v&s=index

// output

index        shard prirep state      docs store ip         node
broken_index 1     p      STARTED       0  208b 172.20.0.2 esnode
broken_index 1     r      UNASSIGNED                       
broken_index 1     r      UNASSIGNED                       
broken_index 0     p      STARTED       1 3.8kb 172.20.0.2 esnode
broken_index 0     r      UNASSIGNED                       
broken_index 0     r      UNASSIGNED    

```

Here we can see that a shard is `UNASSIGNED`.

Why is that?

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/cluster-allocation-explain.html

A lot of information is presented here, if you have a larger underlying issue you will see many explainations across many indices.  Try to keep that in mind.

For each index and then each node, read the explanations

</details>
<hr>



## Repair

:question: Repair by reducing the number of replicas required.  This matches the number of replia nodes available.  In this case 0 as we are running on a single node cluster.

```json
PUT /broken_index/_settings
{
    "number_of_replicas": 0
    
}

// output

{
  "acknowledged" : true
}
```

Check the index health again


```json
GET _cat/indices/broken_index?v

// output

health status index        uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   broken_index xHfY0EcGRq-RRZv_cQONCw   2   0          1            0        4kb            4kb

```

</details>
<hr>

# Backup and restore a cluster and/or specific indices

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/backup-cluster.html

> You cannot back up an Elasticsearch cluster by simply taking a copy of the data directories of all of its nodes. 
> 
> Elasticsearch may be making changes to the contents of its data directories while it is running; copying its data directories cannot be expected to capture a consistent picture of their contents. 
>
> If you try to restore a cluster from such a backup, it may fail and report corruption and/or missing files. Alternatively, it may appear to have succeeded though it silently lost some of its data. 
> 
> The only reliable way to back up a cluster is by using the snapshot and restore functionality.

## To have a complete backup for your cluster:

- Back up the data
- Back up the cluster configuration
- Back up the security configuration

## To restore your cluster from a backup:

- Restore the data
- Restore the security configuration

### Version compatibility
> Version compatibility refers to the underlying Lucene index compatibility. Follow the Upgrade documentation when migrating between versions.
>
> A snapshot contains a copy of the on-disk data structures that make up an index. This means that snapshots can only be restored to versions of Elasticsearch that can read the indices:

- A snapshot of an index created in 6.x can be restored to 7.x.
- A snapshot of an index created in 5.x can be restored to 6.x.
- A snapshot of an index created in 2.x can be restored to 5.x.
- A snapshot of an index created in 1.x can be restored to 2.x.


### Repositories
> You must register a snapshot repository before you can perform snapshot and restore operations. We recommend creating a new snapshot repository for each major version. The valid repository settings depend on the repository type.


:question: Backup and restore an index using `snapshots`

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/modules-snapshots.html

1. :question: Backup the `shakespeare` index to a snapshot called `shakespeare_snapshot_<current_date>`

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/snapshot-restore.html

You will need to make sure that the `path.repo` setting has been applied to each ElasticSearch node before doing this.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/snapshots-register-repository.html

The docker images in this Git Repository have this set in the `1es-1kb-xpackSec.yml` single node cluster.  Which was used predominately through out the other sections.

Normally you would save the snapshots to share storage like NFS, AWS S3 etc.   In this demo we use the local filesystem `/tmp` this is not recommended in production.


This can all be done in the kibana GUI https://www.elastic.co/guide/en/kibana/7.13/snapshot-repositories.html

## Check that path.repo is set

```json
GET /_nodes?pretty&filter_path=nodes.*.settings.path

// output

{
  "nodes" : {
    "HKbyLT8xRMC08bJO32XNFg" : {
      "settings" : {
        "path" : {
          "logs" : "/usr/share/elasticsearch/logs",
          "home" : "/usr/share/elasticsearch",
          "repo" : "/tmp"
        }
      }
    }
  }
}
```
As you can see the `path.repo` is set to `/tmp`.

## Regster the backup location
```json
PUT /_snapshot/my_test_backup
{
    "type": "fs",
    "settings": {
        "location": "/tmp/test1",
        "compress": true
    }
}

// output

{
  "acknowledged" : true
}
```
Notice that `/tmp` needed to be available and that you can then append a path to that.  eg. `/tmp/test1`


## Make a snapshot, to that registered location

Date math requires the snapshot name to be enclosed in angled brackets '<>' that is: '%3C' '%3E'

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/date-math-index-names.html#date-math-index-names
> You must enclose date math names in angle brackets. If you use the name in a request path, `special characters must be URI encoded.`

```json
PUT /_snapshot/my_test_backup/%3Cshakespeare-snapshot-%7Bnow%2Fd%7D%3E
{
  "indices": "shakespeare",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

## Check the snapshot 

List all snapshots

```json
GET /_snapshot/my_test_backup/_all

// or

GET /_snapshot/my_test_backup/*

// output 

{
  "snapshots" : [
    {
      "snapshot" : "shakespeare-snapshot-2021.05.13",
      "uuid" : "t0Qk_gDtT1uqvmrbNg7yIQ",
      "version_id" : 7020099,
      "version" : "7.2.0",
      "indices" : [
        "shakespeare"
      ],
      "include_global_state" : false,
      "state" : "SUCCESS",
      "start_time" : "2021-05-13T18:59:27.117Z",
      "start_time_in_millis" : 1620932367117,
      "end_time" : "2021-05-13T18:59:28.050Z",
      "end_time_in_millis" : 1620932368050,
      "duration_in_millis" : 933,
      "failures" : [ ],
      "shards" : {
        "total" : 1,
        "failed" : 0,
        "successful" : 1
      }
    }
  ]
}
```

</details>
<hr>

2. :question: Restore the `shakespeare_snapshot_<current_date>` index snapshot to the name `restored_index_shakespeare`

<details>
  <summary>View Solution (click to reveal)</summary>

```json
POST /_snapshot/my_test_backup/shakespeare-snapshot-2021.05.13/_restore
{
  "indices": "shakespeare",
  "ignore_unavailable": true,
  "include_global_state": true,
  "rename_pattern": "(.+)",
  "rename_replacement": "restored_index_$1"
}

// output

{
  "accepted" : true
}
```

Check the restored index

```json
GET _cat/indices/*shakespeare

// output

health status index                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   shakespeare                h21mMC7ZRWGB1OPjgW0VuQ   1   1     111396            0     20.5mb         20.5mb
yellow open   restored_index_shakespeare gKIdyU4jSnqmuBLRqPpZLw   1   1     111396            0     20.5mb         20.5mb
```

:warning: Health is yellow due to the number of replicas is above 0 for a single node deployment.

</details>
<hr>

3. :closed_book: Backup/Restore the cluster configuration

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/backup-cluster-configuration.html

> We recommend that you take regular (ideally, daily) backups of your Elasticsearch config ($ES_PATH_CONF) directory using the file backup software of your choice.

Normally `/etc/elasticsearch`

> Some settings in configuration files might be overridden by cluster settings. You can capture these settings in a data backup snapshot by specifying the include_global_state: true (default) parameter for the snapshot API. 
>
>Alternatively, you can extract these configuration values in text format by using the get settings API:

```json
GET _cluster/settings?pretty&flat_settings&filter_path=persistent
```

So, it's a probably good idea to backup your `/etc/elasticsearch/` folder and run an external API call to download the persistant settings to a text file.

4. :closed_book: Backup/Restore the Security configuration

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-backup.html

## Back up file-based security configuration
> Elasticsearch security features are configured using the xpack.security namespace inside the elasticsearch.yml and elasticsearch.keystore files. In addition there are several other extra configuration files inside the same ES_PATH_CONF directory. These files define roles and role mappings and configure the file realm. 

## Back up index-based security configuration

> Elasticsearch security features store system configuration data inside a dedicated index. This index is named .security-6 in the Elasticsearch 6.x versions and .security-7 in the 7.x releases. The .security alias always points to the appropriate index. This index contains the data which is not available in configuration files and cannot be reliably backed up using standard filesystem tools. This data describes:
>
> - the definition of users in the native realm (including hashed passwords)
> - role definitions (defined via the create roles API)
> - role mappings (defined via the create role mappings API)
> - application privileges
> - API keys
>
> The .security index thus contains resources and definitions in addition to configuration information. All of that information is required in a complete security features backup.

> Use the standard Elasticsearch snapshot functionality to backup .security, as you would for any other data index.
>
> Snapshot the .security index in a dedicated repository, where read and write access is strictly restricted and audited.

So, backup the `/etc/elasticsearch` folder,

and

snapshot the `.security` index alias.


## Restore the Security index

See https://www.elastic.co/guide/en/elasticsearch/reference/7.13/restore-security-configuration.html





# Configure a snapshot to be searchable

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/searchable-snapshots.html

> Searchable snapshots let you use snapshots to search infrequently accessed and read-only data in a very cost-effective fashion. The cold and frozen data tiers use searchable snapshots to reduce your storage and operating costs.
> 
> Searchable snapshots eliminate the need for replica shards, potentially halving the local storage needed to search your data. Searchable snapshots rely on the same snapshot mechanism you already use for backups and have minimal impact on your snapshot repository storage costs.

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/ilm-searchable-snapshot.html

#TODO:


# Configure a cluster for cross cluster search

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/modules-cross-cluster-search.html

## Update the cluster settings with the seed node of each remote cluster

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_one": {
          "seeds": [
            "10.0.0.1:9300"
          ]
        },
        "cluster_two": {
          "seeds": [
            "2.0.0.1:9300"
          ]
        },
        "cluster_three": {
          "seeds": [
            "3.0.0.1:9300"
          ]
        }
      }
    }
  }
}
```
Here we have three clusters, one on the local network somewhere and two others out on the internet.

## Perform as remote cluster search

```json
GET /cluster_one:twitter/_search
{
  "query": {
    "match": {
      "user": "kimchy"
    }
  }
}
```
Here we search one of the remote clusters from inside out local cluster.

Remote clusters are accessed as such <remote_name>:<index_name>

## Perform a multiple cross cluster search

```json
GET /twitter,cluster_one:twitter,cluster_two:twitter/_search
{
  "query": {
    "match": {
      "user": "kimchy"
    }
  }
}
```
Here we search the local cluster and two remote clusters.

# Implement cross-cluster replication

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/xpack-ccr.html

> With cross-cluster replication, you can replicate indices across clusters to:
>
> - Continue handling search requests in the event of a datacenter outage
> - Prevent search volume from impacting indexing throughput
> - Reduce search latency by processing search requests in geo-proximity to the user
>
> Cross-cluster replication uses an active-passive model. You index to a leader index, and the data is replicated to one or more read-only follower indices. Before you can add a follower index to a cluster, you must configure the remote cluster that contains the leader index.

This is a heavily involved process - follow this link
https://www.elastic.co/guide/en/elasticsearch/reference/7.13/ccr-getting-started.html

#TODO:  build two clusters, add sample data and replicate between them
!! needs new docker compose



# Define role-based access control using Elasticsearch Security

>  :warning: :warning: :warning: IMPORTANT NOTE: from here on it is assumed you have a working kibana node to work from the "development console" and that you have imported the sample data.

> See: [Running test servers on docker](Test_servers_on_docker.md)

> See: [Importing Sample data](Importing_sample_data.md)

:question: To do this section you need to:

- create roles and users

:warning: Not all of the security features can be used due to Basic Licence limitations.

You will see an error like the below if you have a Basic licence.

The following role parameters are not available under Basic licence.

```yaml
  "field_security" : { ... }, # field level security
  "query": "..." # document level security
```

Error message:
```yaml
{
  "error": {
    "root_cause": [
      {
        "type": "security_exception",
        "reason": "current license is non-compliant for [field and document level security]",
        "license.expired.feature": "field and document level security"
      }
    ],
    "type": "security_exception",
    "reason": "current license is non-compliant for [field and document level security]",
    "license.expired.feature": "field and document level security"
  },
  "status": 403
}
```

## Create a role

:warning: This assumes you have ingested the Elastic sample data.
> See: [Importing Sample data](Importing_sample_data.md)

### :question:  Create role and user for the following:

- a role called `flights_all` for read only access on the Flight sample data
- the role should have cluster monitor access
- a user called `flight_reader_all` that has the role applied
- the user password should be `flight123`

<details>
  <summary>View Solution (click to reveal)</summary>


https://www.elastic.co/guide/en/elasticsearch/reference/7.13/built-in-roles.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-privileges.html

```json
PUT _security/role/flights_all
{
  "cluster": [ "monitor" ],
  "indices": [
    {
      "names": ["kibana_sample_data_flights"],
      "privileges": ["read","view_index_metadata", "monitor"]
    }
  ]
}

PUT _security/user/flight_reader_all
{
  "password": "flight123",
  "roles": [ "kibana_user", "flights_all" ],
  "full_name": "flights all",
  "email": "fa@abc.com"
}
```

Test the user access

- Logout as elastic 
- Login to Kibana as `flight_reader_all` password `flight123`
- Go to dev console and see what indices you have access to.

Check that we can access the index stats (monitor)
and only the index we have allowed access to.

```json
GET _cat/indices

green open kibana_sample_data_flights R1AptZYHTrivEUEmtftubg 1 0 13059 0 6.6mb 6.6mb
```

Check that we can query the index.  Get the document count

```json
GET kibana_sample_data_flights/_count

{
  "count" : 13059,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```


</details>




### :question:  Create a role with field and document level security, plus a user for the following:

:warning: can only be applied if you have purchased an elastic licence. This will not work on the Basic/Free licence.

- a role called `flights_australia` for read only access on the Flght data that only allows access to data that has a Destination Country of Australia.
- the following fields are allowed to be displayed: Flight Number, Country of Origin and City of Origin
- a user called `flight_reader_au` should have the role applied to it

<details>
  <summary>View Solution (click to reveal)</summary>

```json

# test your query first
POST kibana_sample_data_flights/_search
{
  "query": {
        "match": {
          "DestCountry": "AU"
        }
      }
}

PUT _security/role/flights_australia
{
  "indices": [
    {
      "names": [
        "kibana_sample_data_flights"
      ],
      "privileges": [
        "read"
      ],
      "field_security": {
        "grant": ["FlightNum", "OriginCountry", "OriginCityName"]
      }, 
      "query": {
        "match": {
          "DestCountry": "AU"
        }
      }
    }
  ]
}
```

Create a user for that role

```yaml
PUT _security/user/flight_reader_au
{
  "password": "flight123",
  "roles": "flights_australia",
  "full_name": "flights australia",
  "email": "fau@abc.com"
}
```

Test

- Logout as elastic 
- Login to Kibana as `flight_reader_au`
- Go to dev console and see what indices you have access to.

#TODO:  write a query to count the number of documents accessible

</details>
