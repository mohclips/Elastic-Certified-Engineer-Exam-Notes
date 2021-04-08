# Installation and Configuration

# Deploy and start an Elasticsearch cluster that satisfies a given set of requirements

## Preparing the host

### Create a user
Create a user called `elastic`

<details>
  <summary>View Solution (click to reveal)</summary>

As root, create a user called elastic, set the password (I used something simple, you might not want to do this if your server is accessible from the internet or by other users).  Add that user to the `wheel` group to allow them access to use sudo.

```bash
[root@centos8streams ~]# useradd -m -c "elastic training" elastic
[root@centos8streams ~]# echo "Password1!" | passwd --stdin elastic
[root@centos8streams ~]# usermod -G wheel elastic
```
</details>
<hr>

### Set nofile limits
Set the `nofile` limits to the recommended size

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html
https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html

> Elasticsearch uses a lot of file descriptors or file handles. Running out of file descriptors can be disastrous and will most probably lead to data loss. Make sure to increase the limit on the number of open files descriptors for the user running Elasticsearch to `65,536 or higher`.

`vi /etc/security/limits.conf`

```
elasticsearch  -  nofile  65535
elastic        -  nofile  65535
```


This change will only take effect the next time the elasticsearch user opens a new session.
</details>
<hr>

### Set vm.max_map_count
Set the `vm.max_map_count` to the minimum required value

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/guide/master/_file_descriptors_and_mmap.html
https://www.elastic.co/guide/en/elasticsearch/reference/master/_maximum_map_count_check.html

> Continuing from the previous point, to use mmap effectively, Elasticsearch also requires the ability to create many memory-mapped areas. The maximum map count check checks that the kernel allows a process to have at least 262,144 memory-mapped areas and is enforced on Linux only. 

Temporary fix

`sysctl -w vm.max_map_count=262144`

Permenant fix

`vi /etc/sysctl.conf`

```
vm.max_map_count=262144`
```
</details>
<hr>

### Install ElasticSearch
Note: that this is just the ElasticSearch component, and not Kibana, Logstash or Beats.

### Simple installation
Here we are installing the tarball. 

Download the version that matches the exam (at time of writing it is still 7.2)

Go here : https://www.elastic.co/downloads/elasticsearch

Then go to Not the version you're looking for? `View past releases`.

Find the version that matches: eg. https://www.elastic.co/downloads/past-releases/elasticsearch-7-2-1

Grab the `LINUX` tarball - right click and copy link

#### Download to the server

```bash
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-linux-x86_64.tar.gz
```

Extract the files

```bash
$ tar xf elasticsearch-7.2.1-linux-x86_64.tar.gz
```

Rename the folder

```bash
$ mv elasticsearch-7.2.1 elasticsearch
```

## Start the application

Note that we aren't using a service here as we want to test the startup

```bash
$ cd
$ ./elasticsearch
$ ./bin/elasticsearch --pidfile mypidfile
```

## Test Application health

In another terminal session run the following curl command.

```bash
[elastic@centos8streams ~]$ curl localhost:9200/_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1617885668 12:41:08  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

This shows the cluster health

We are looking for status = `green`


To look at node health

```bash
[elastic@centos8streams ~]$ curl localhost:9200/_cat/nodes?v
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           11          95   1    0.00    0.00     0.00 mdi       *      centos8streams.preview.local
```


## Stop the application

In another terminal session

```bash
[elastic@centos8streams elasticsearch]$ ls
bin  config  data  jdk  lib  LICENSE.txt  logs  modules  mypidfile  NOTICE.txt  plugins  README.textile
[elastic@centos8streams elasticsearch]$ pkill -F mypidfile
```

<hr>


# Configure the nodes of a cluster to satisfy a given set of requirements

Using the previously installed application we will update the two main configuration files.

`elasticsearch.yml` and `jvm.options`

See: https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html

for v7.2 See: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html  There are differences between 7.12 (current) and 7.2 (exam).


```bash
[elastic@centos8streams config]$ pwd
/home/elastic/elasticsearch/config
[elastic@centos8streams config]$ ls
elasticsearch.keystore  elasticsearch.yml  jvm.options  log4j2.properties  role_mapping.yml  roles.yml  users  users_roles
[elastic@centos8streams config]$
```

## Configuration settings

Written in YAML.  Below are in dotted notation, but can be written in YAML dictionaries.

The following `elasticsearch.yml` are worth noting - but there are many mode.

- cluster.name: The name of the cluster (most likely the same for all your nodes)
- node.name: The name of this node - needs to be unique
- node.master: true/false - master-eligible role
- node.data: true/false - data role
- node.ingest: true/false - ingest role
- node.attr.some_attribute: node attributes to tag nodes (such as hot/warm architecture)
- path.repo: The filesystem path to be used as the snapshot repository
- network.host: Interfaces for Elasticsearch to bind to (typically _local_ and/or _site_)
-
- discovery.seed_hosts: List of nodes to 'seed' thus ping on startup (normally these are master-eligible nodes)
- cluster.initial_master_nodes: List of master-eligible node names for bootstrapping the cluster and preventing split-brain
-
- xpack.security.enabled: true/false
- xpack.security.transport.ssl.enabled: true/false
- xpack.security.transport.ssl.verification_mode: Set to full for node-level verification, or certificate
for just certificate-level verification
- xpack.security.transport.ssl.keystore.path: Path to keystore file for transport network encryption
- xpack.security.transport.ssl.truststore.path: Path to truststore file for transport network encryption
- xpack.security.http.ssl.keystore.path: Path to keystore file for http network encryption
- xpack.security.http.ssl.truststore.path: Path to truststore file for http network encryption

The most important `jvm.options` are the following:

- -Xms initial heap size
- -Xmx max heap size


### Set the JVM heap space

Set the JVM heap space to 2 gig


<details>
  <summary>View Solution (click to reveal)</summary>


Edit the jvm.options file

```bash
[elastic@centos8streams config]$ cd /home/elastic/elasticsearch/config
[elastic@centos8streams config]$ vi jvm.options
```

```ini
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms2g
-Xmx2g

```
This would be done on each affected node and then restart elasticsearch on that node.

</details>
<hr>

### Set the cluster and node config

Set the cluster to the following

- cluster name: testcluster01
- node name: node01
- node roles: master, data
- node attributes: datacenter = uksouth


<details>
  <summary>View Solution (click to reveal)</summary>


Edit the elasticsearch.yml file

```bash
[elastic@centos8streams config]$ cd /home/elastic/elasticsearch/config
[elastic@centos8streams config]$ vi elasticsearch.yml
```

Set the following
```yaml

cluster.name: testcluster01
node.name: node01
node.master: true
node.data: true
node.ingest: false
node.attr.datacenter: uksouth
```

This would be done on each affected node and then restart elasticsearch on that node.

### Check

Using the API pull the node settings back.

```bash
[elastic@centos8streams config]$ curl -s -X GET http://localhost:9200/_nodes/settings | jq .
```
```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "testcluster01",
  "nodes": {
    "1XCLS3R4ROG1GASp5xM4GQ": {
      "name": "node01",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1",
      "version": "7.2.1",
      "build_flavor": "default",
      "build_type": "tar",
      "build_hash": "fe6cb20",
      "roles": [
        "master",
        "data"
      ],
      "attributes": {
        "ml.machine_memory": "1905602560",
        "xpack.installed": "true",
        "ml.max_open_jobs": "20",
        "datacenter": "uksouth"
      },
      "settings": {
        "pidfile": "mypidfile",
        "cluster": {
          "name": "testcluster01"
        },
        "node": {
          "name": "node01",
          "attr": {
            "datacenter": "uksouth",
            "xpack": {
              "installed": "true"
            },
            "ml": {
              "machine_memory": "1905602560",
              "max_open_jobs": "20"
            }
          },
          "data": "true",
          "ingest": "false",
          "master": "true"
        },
        "path": {
          "logs": "/home/elastic/elasticsearch/logs",
          "home": "/home/elastic/elasticsearch"
        },
        "client": {
          "type": "node"
        },
        "http": {
          "type": "security4",
          "type.default": "netty4"
        },
        "transport": {
          "type": "security4",
          "features": {
            "x-pack": "true"
          },
          "type.default": "netty4"
        }
      }
    }
  }
}
```


</details>
<hr>





# Secure a cluster using Elasticsearch Security
# Define role-based access control using Elasticsearch Security
