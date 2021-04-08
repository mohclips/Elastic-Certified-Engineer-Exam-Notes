# Installation and Configuration

# Deploy and start an Elasticsearch cluster that satisfies a given set of requirements

## Preparing the host

### Create a user
Create a user called `elastic`

<details>
  <summary>Solution</summary>

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
  <summary>Solution</summary>

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
  <summary>Solution</summary>

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

Download to the server

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

```bash
[elastic@centos8streams ~]$ curl localhost:9200/_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1617885668 12:41:08  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

We are looking for `green`




## Stop the application

In another terminal session

```bash
$ pkill -F mypidfile
```




# Configure the nodes of a cluster to satisfy a given set of requirements
# Secure a cluster using Elasticsearch Security
# Define role-based access control using Elasticsearch Security
