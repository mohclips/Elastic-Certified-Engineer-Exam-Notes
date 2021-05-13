# Cluster Administration

The following will need a couple of clusters.

Use the following:
```bash
xxxxxxxxxxxxx

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


https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html

The index allocation settings support the following built-in attributes:

- _name = Match nodes by node name
- _host_ip = Match nodes by host IP address (IP associated with hostname)
- _publish_ip = Match nodes by publish IP address
- _ip = Match either _host_ip or _publish_ip
- _host = Match nodes by hostname

You can use wildcards when specifying attribute values


# Configure shard allocation awareness and forced awareness for an index


# Diagnose shard issues and repair a cluster's health


# Backup and restore a cluster and/or specific indices


# Configure a cluster for use with a hot/warm architecture


# Configure a cluster for cross cluster search