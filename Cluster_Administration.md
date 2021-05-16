# Cluster Administration

The following topics will need a few nodes.

Use the following:
```bash
sudo sysctl -w vm.max_map_count=262144
```

## Node filters

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster.html

- _all, to add all nodes to the subset.
- _local, to add the local node to the subset.
- _master, to add the currently-elected master node to the subset.
- a node id or name, to add this node to the subset.
- an IP address or hostname, to add all matching nodes to the subset.
- a pattern, using * wildcards, which adds all nodes to the subset whose name, address or hostname matches the pattern.
- master:true, data:true, ingest:true or coordinating_only:true, which respectively add to the subset all master-eligible nodes, all data nodes, all ingest nodes, and all coordinating-only nodes.
- master:false, data:false, ingest:false or coordinating_only:false, which respectively remove from the subset all master-eligible nodes, all data nodes, all ingest nodes, and all coordinating-only nodes.
- a pair of patterns, using * wildcards, of the form attrname:attrvalue, which adds to the subset all nodes with a custom node attribute whose name and value match the respective patterns. Custom node attributes are configured by setting properties in the configuration file of the form node.attr.attrname: attrvalue.


# Allocate the shards of an index to specific nodes based on a given set of requirements

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html

The index allocation settings support the following built-in attributes:

- _name = Match nodes by node name
- _host_ip = Match nodes by host IP address (IP associated with hostname)
- _publish_ip = Match nodes by publish IP address
- _ip = Match either _host_ip or _publish_ip
- _host = Match nodes by hostname

You can use wildcards when specifying attribute values

Or use `custom` attributes.

<hr>

:question: 1. Display the current node attributes

<details>
  <summary>View Solution (click to reveal)</summary>

If you are using the docker-compose `3esnode-cluster.yml` you will find that the three nodes have a `location` attribute applied to them.

If you have used your own cluster, apply `location` attributes `north`, `east`, `south` to each of your nodes.

```json
GET _cat/nodes?v

// output

ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.18.0.3           11          78  10    0.27    0.24     0.44 mdi       *      es03
172.18.0.4           12          78  10    0.27    0.24     0.44 mdi       -      es01
172.18.0.2           10          78   9    0.27    0.24     0.44 mdi       -      es02

// get actual attributes

GET _cat/nodeattrs?v&s=node

// output

node host       ip         attr            value
es01 172.18.0.4 172.18.0.4 location        north
es01 172.18.0.4 172.18.0.4 xpack.installed true
es02 172.18.0.2 172.18.0.2 location        south
es02 172.18.0.2 172.18.0.2 xpack.installed true
es03 172.18.0.3 172.18.0.3 location        east
es03 172.18.0.3 172.18.0.3 xpack.installed true

```
</details>
<hr>

:question: 2. Allocate a new index `test_north` only the node that has attribute `location=north`

<details>
  <summary>View Solution (click to reveal)</summary>


```json
PUT /test_north
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0, 
    "index.routing.allocation.require.location": "north"
  }
}

// output 

{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test_north"
}
```

## View the allocation

Note: that the index can be accessed via a wild card '*'

```json
GET _cat/indices/test_*?v

//  output

health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test_north                      a74o_2ixRka2seBCZirSaQ   1   0          0            0       230b           230b


// view where the shards are stored - the actual allocation

GET _cat/shards?v&s=index

index                           shard prirep state   docs   store ip         node
.kibana_1                       0     p      STARTED    4  18.3kb 172.18.0.2 es02
.kibana_1                       0     r      STARTED    4  15.4kb 172.18.0.3 es03
.kibana_task_manager            0     r      STARTED    2  29.8kb 172.18.0.4 es01
.kibana_task_manager            0     p      STARTED    2  45.5kb 172.18.0.3 es03
.monitoring-es-7-2021.05.13     0     r      STARTED   41 662.5kb 172.18.0.4 es01
.monitoring-es-7-2021.05.13     0     p      STARTED   41   720kb 172.18.0.2 es02
.monitoring-kibana-7-2021.05.13 0     r      STARTED    5  70.2kb 172.18.0.2 es02
.monitoring-kibana-7-2021.05.13 0     p      STARTED  206 116.2kb 172.18.0.3 es03
test_north                      0     p      STARTED    0    230b 172.18.0.4 es01

```
You can see above that all the internal indices have multiple shards allocated across multiple nodes.

`test_north` is only allocated to `es01` node, which is where the attribute `location=north` is set.

</details/>
<hr>

:question: 3. Allocate a new index `test_southeast` to the cluster but make sure it is not allocated to the node in  `location=north`

<details>
  <summary>View Solution (click to reveal)</summary>


```json
PUT /test_southeast
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1, 
    "index.routing.allocation.exclude.location": "north"
  }
}

// output

{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test_southeast"
}
```

## Now check the shard allocation

```json 
GET _cat/shards/test_*?v&s=index

// output

index                           shard prirep state   docs   store ip         node
test_north                      0     p      STARTED    0    283b 172.18.0.4 es01
test_southeast                  0     p      STARTED    0    230b 172.18.0.2 es02
test_southeast                  0     r      STARTED    0    230b 172.18.0.3 es03
```
So, above you can see there is a primary `p` and replica `r` shards allocated to `es02` and `es03`

</details>
<hr>

:question: 4.  What happens if we increase the replica shard allocation to above `1` to `2` we have three nodes don't we?

One primary plus two replcias equals three.  Three nodes.

Create a new index called `test_northeast` make sure it cannot be allocated to the `south` location.  Also make sure it has `2` replicas.


<details>
  <summary>View Solution (click to reveal)</summary>

```json
PUT /test_northeast
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2, 
    "index.routing.allocation.exclude.location": "south"
  }
}

// output

{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test_northeast"
}
```

All looks fine.

Or is it?

Check the index health...

```json
GET _cat/indices/test_*?v

// output

health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   test_northeast                  4vpwoBaARVuRQ8Jfl-wXaw   1   2          0            0       460b           230b
green  open   test_north                      a74o_2ixRka2seBCZirSaQ   1   0          0            0       283b           283b
green  open   test_southeast                  X61MsKylRz6EUztckI7ILA   1   1          0            0       566b           283b

```

The index health is `yellow` !

Follow on to the diagnostics topic,to find out why :)

</details>
<hr>

# Configure shard allocation awareness and forced awareness for an index

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html
## Allocation awareness

> You can use custom node attributes as awareness attributes to enable Elasticsearch to take your physical hardware configuration into account when allocating shards. 
>
> If Elasticsearch knows which nodes are on the same physical server, in the same rack, or in the same zone, it can distribute the primary shard and its replica shards to minimise the risk of losing all shard copies in the event of a failure.

This allows you to setup the master eligible nodes so that they area aware of where they are locationwise.

Thus if a rack, zone or datacenter is down, the cluster will try to allocate the replicas to the same location as the primary shards.

:question: Enable shard allocation awareness

<details>
  <summary>View Solution (click to reveal)</summary>

This is done in the `elasticsearch.yml`

```json
cluster.routing.allocation.awareness.attributes: <location_attribute>
```
</details>
<hr>

## Forced awareness

This works the same as Allocation Awareness, but it will never allocate the missing replicas to the last remaining zone.


> By default, if one location fails, Elasticsearch assigns all of the missing replica shards to the remaining locations. While you might have sufficient resources across all locations to host your primary and replica shards, a single location might be unable to host ALL of the shards.
> 
> To prevent a single location from being overloaded in the event of a failure, you can set cluster.routing.allocation.awareness.force so no replicas are allocated until nodes are available in another location.
> 
> For example, if you have an awareness attribute called zone and configure nodes in zone1 and zone2, you can use forced awareness to prevent Elasticsearch from allocating replicas if only one zone is available

:question: Enable shard forced allocation awareness

<details>
  <summary>View Solution (click to reveal)</summary>

This is done in the `elasticsearch.yml`

```json
cluster.routing.allocation.awareness.attributes: <location_attribute>
cluster.routing.allocation.awareness.force.zone.values: <location_1>,  <location_3>,  <location_3>,...
```
</details>
<hr>

# Diagnose shard issues and repair a cluster's health

In the above question `4` the index `test_northeast` is showing it's health as `yellow`, which means it has a fault.


:question: Diagnose the fault in index `test_northeast`

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cat-shards.html

```json
GET _cat/shards/test_northeast?v&s=index

// output

index          shard prirep state      docs store ip         node
test_northeast 0     p      STARTED       0  283b 172.18.0.4 es01
test_northeast 0     r      STARTED       0  283b 172.18.0.3 es03
test_northeast 0     r      UNASSIGNED                       
```
Here we can see that a shard is `UNASSIGNED`.

Why is that?

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html

A lot of information is presented here, if you have a larger underlying issue you will see many explainations across many indices.  Try to keep that in mind.

For each index and then each node, read the explanations

```json
GET _cluster/allocation/explain

// output

{
  "index" : "test_northeast",
  "shard" : 0,
  "primary" : false,
  "current_state" : "unassigned",
  "unassigned_info" : {
    "reason" : "INDEX_CREATED",
    "at" : "2021-05-13T10:47:56.065Z",
    "last_allocation_status" : "no_attempt"
  },
  "can_allocate" : "no",
  "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions" : [
    {
      "node_id" : "801RTxZORfifFhY58TnwTQ",
      "node_name" : "es02",
      "transport_address" : "172.18.0.2:9300",
      "node_attributes" : {
        "location" : "south",
        "xpack.installed" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 1,
      "deciders" : [
        {
          "decider" : "filter",
          "decision" : "NO",
          "explanation" : """node matches index setting [index.routing.allocation.exclude.] filters [location:"south"]"""
        }
      ]
    },
    {
      "node_id" : "pwbM6eDoS9urwE5btAnn3w",
      "node_name" : "es01",
      "transport_address" : "172.18.0.4:9300",
      "node_attributes" : {
        "location" : "north",
        "xpack.installed" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 2,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[test_northeast][0], node[pwbM6eDoS9urwE5btAnn3w], [P], s[STARTED], a[id=2Pp4LDJZSfqQQrzuie4Gxg]]"
        }
      ]
    },
    {
      "node_id" : "ZJnCw4XtSQO_nBpawz_KTA",
      "node_name" : "es03",
      "transport_address" : "172.18.0.3:9300",
      "node_attributes" : {
        "location" : "east",
        "xpack.installed" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 3,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[test_northeast][0], node[ZJnCw4XtSQO_nBpawz_KTA], [R], s[STARTED], a[id=MZmL_-91T3in8gld7gndqg]]"
        }
      ]
    }
  ]
}

```

You can remove some of the verbosity with this...

```json
GET _cluster/allocation/explain?filter_path=index,allocate_explanation,node_allocation_decisions.node_name,node_allocation_decisions.deciders.explanation

// output

{
  "index" : "test_northeast",
  "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions" : [
    {
      "node_name" : "es02",
      "deciders" : [
        {
          "explanation" : """node matches index setting [index.routing.allocation.exclude.] filters [location:"south"]"""
        }
      ]
    },
    {
      "node_name" : "es01",
      "deciders" : [
        {
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[test_northeast][0], node[pwbM6eDoS9urwE5btAnn3w], [P], s[STARTED], a[id=2Pp4LDJZSfqQQrzuie4Gxg]]"
        }
      ]
    },
    {
      "node_name" : "es03",
      "deciders" : [
        {
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[test_northeast][0], node[ZJnCw4XtSQO_nBpawz_KTA], [R], s[STARTED], a[id=MZmL_-91T3in8gld7gndqg]]"
        }
      ]
    }
  ]
}

```

## Repair

:question: Repair by reducing the number of replicas required.  This matches the number of replia nodes available.  In this case 1.

```json
PUT /test_northeast/_settings
{
    "number_of_replicas": 1
}

// output

{
  "acknowledged" : true
}
```

Check the index health again


```json
GET _cat/indices/test_northeast?v

// output

health status index          uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test_northeast 4vpwoBaARVuRQ8Jfl-wXaw   1   1          0            0       566b           283b
```

</details>
<hr>

# Backup and restore a cluster and/or specific indices

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/backup-cluster.html

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html

1. :question: Backup the `shakespeare` index to a snapshot called `shakespeare_snapshot_<current_date>`

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html#_snapshot

You will need to make sure that the `path.repo` setting has been applied to each ElasticSearch node before doing this.

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html#_shared_file_system_repository

The docker images in this Git Repository have this set in the `1es-1kb-xpackSec.yml` single node cluster.  Which was used predominately through out the other sections.

Normally you would save the snapshots to share storage like NFS, AWS S3 etc.   In this demo we use the local filesystem `/tmp` this is not recommended in production.

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

```json
PUT /_snapshot/my_test_backup/%3Cshakespeare-snapshot-%7Bnow%2Fd%7D%3E
{
  "indices": "shakespeare",
  "ignore_unavailable": true,
  "include_global_state": false
}
```
Date math requires the snapshot name to be enclosed in angled brackets '<>'  '%3C' '%3E'


## Check the snapshot 

List all snapshots

```json
GET /_snapshot/my_test_backup/_all

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/backup-cluster-configuration.html

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

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-backup.html

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

See https://www.elastic.co/guide/en/elasticsearch/reference/7.2/restore-security-configuration.html


# Configure a cluster for use with a hot/warm architecture

Following on from the above shard allocation topics, this becomes quite easy to do.

:question: Create a node attribute called `temp`, assign `hot`, `warm` values to the nodes that you need to re-allocate the shards to.

:question: Assign `index.routing.allocation.require.temp` and `hot`, `warm` values to each index you wish to apply the hot/warm archtecture to.

# Configure a cluster for cross cluster search

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html

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
