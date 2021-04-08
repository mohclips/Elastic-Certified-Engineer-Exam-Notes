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
$ cd elasticsearch
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

Written in YAML.  Below are in dotted notation, but can be written as YAML dictionaries.

The following `elasticsearch.yml` are worth noting - but there are many mode.

- **cluster.name**: The name of the cluster (most likely the same for all your nodes)
- **node.name**: The name of this node - needs to be unique
- **node.master**: true/false - master-eligible role
- **node.data**: true/false - data role
- **node.ingest**: true/false - ingest role
- **node.attr.some_attribute**: node attributes to tag nodes (such as hot/warm architecture)
- **path.repo**: The filesystem path to be used as the snapshot repository
- **network.host**: Interfaces for Elasticsearch to bind to (typically _local_ and/or _site_)
-
- **discovery.seed_hosts**: List of nodes to 'seed' thus ping on startup (normally these are master-eligible nodes)
- **cluster.initial_master_nodes**: List of master-eligible node names for bootstrapping the cluster and preventing split-brain
-
- **xpack.security.enabled**: true/false
- **xpack.security.transport.ssl.enabled**: true/false
- **xpack.security.transport.ssl.verification_mode**: full/certificate - node-level verification (DNS), or certificate for just certificate-level verification (each end has certs)
- **xpack.security.transport.ssl.keystore.path**: Path to keystore file for transport network encryption
- **xpack.security.transport.ssl.truststore.path**: Path to truststore file for transport network encryption
- **xpack.security.http.ssl.keystore.path**: Path to keystore file for http network encryption
- **xpack.security.http.ssl.truststore.path**: Path to truststore file for http network encryption

The most important `jvm.options` are the following:

- **-Xms** initial heap size
- **-Xmx** max heap size


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

:warning: Note that the `node.ingest` is default `true`, so you need to specifically set this to `false`.

This would be done on each affected node and then restart elasticsearch on that node.

### Check

Using the API pull the node settings back.

Here i have used the awesome command `jq` to pretty print the results.  You may want to install this on your nodes. (probably not available in the exam setting)

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

To secure a cluster you need to do a few things:

- Create CA and node certificates
- Enable SSL on the Transport Network
- Enable SSL on the HTTP Network
- Change the built-in user default passwords

## Create a node certificate

Create the CA file and node certificate.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html

v7.2 https://www.elastic.co/guide/en/elasticsearch/reference/7.2/encrypting-communications-certificates.html

### Create the CA file

Use: `./bin/elasticsearch-certutil ca`

```bash
[elastic@centos8streams elasticsearch]$ pwd
/home/elastic/elasticsearch
[elastic@centos8streams elasticsearch]$ ./bin/elasticsearch-certutil ca
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/home/elastic/elasticsearch/lib/tools/security-cli/bcprov-jdk15on-1.61.jar) to constructor sun.security.provider.Sun()
WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
This tool assists you in the generation of X.509 certificates and certificate
signing requests for use with SSL/TLS in the Elastic stack.

The 'ca' mode generates a new 'certificate authority'
This will create a new X.509 certificate and private key that can be used
to sign certificate when running in 'cert' mode.

Use the 'ca-dn' option if you wish to configure the 'distinguished name'
of the certificate authority

By default the 'ca' mode produces a single PKCS#12 output file which holds:
    * The CA certificate
    * The CA's private key

If you elect to generate PEM format certificates (the -pem option), then the output will
be a zip file containing individual files for the CA certificate and private key

Please enter the desired output file [elastic-stack-ca.p12]:
Enter password for elastic-stack-ca.p12 :
```

Ignore the reflective WARNINGS.
Accept the defaults.

### Create the actual node certificate

Use: `./bin/elasticsearch-certutil cert --ca ./elastic-stack-ca.p12 --name node01 --dns centos8streams.preview.local --ip 172.30.5.202`


```bash
[elastic@centos8streams elasticsearch]$ ./bin/elasticsearch-certutil cert --ca ./elastic-stack-ca.p12 --name node01 --dns centos8streams.preview.local --ip 172.30.5.202
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/home/elastic/elasticsearch/lib/tools/security-cli/bcprov-jdk15on-1.61.jar) to constructor sun.security.provider.Sun()
WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
This tool assists you in the generation of X.509 certificates and certificate
signing requests for use with SSL/TLS in the Elastic stack.

The 'cert' mode generates X.509 certificate and private keys.
    * By default, this generates a single certificate and key for use
       on a single instance.
    * The '-multiple' option will prompt you to enter details for multiple
       instances and will generate a certificate and key for each one
    * The '-in' option allows for the certificate generation to be automated by describing
       the details of each instance in a YAML file

    * An instance is any piece of the Elastic Stack that requires a SSL certificate.
      Depending on your configuration, Elasticsearch, Logstash, Kibana, and Beats
      may all require a certificate and private key.
    * The minimum required value for each instance is a name. This can simply be the
      hostname, which will be used as the Common Name of the certificate. A full
      distinguished name may also be used.
    * A filename value may be required for each instance. This is necessary when the
      name would result in an invalid file or directory name. The name provided here
      is used as the directory name (within the zip) and the prefix for the key and
      certificate files. The filename is required if you are prompted and the name
      is not displayed in the prompt.
    * IP addresses and DNS names are optional. Multiple values can be specified as a
      comma separated string. If no IP addresses or DNS names are provided, you may
      disable hostname verification in your SSL configuration.

    * All certificates generated by this tool will be signed by a certificate authority (CA).
    * The tool can automatically generate a new CA for you, or you can provide your own with the
         -ca or -ca-cert command line options.

By default the 'cert' mode produces a single PKCS#12 output file which holds:
    * The instance certificate
    * The private key for the instance certificate
    * The CA certificate

If you specify any of the following options:
    * -pem (PEM formatted output)
    * -keep-ca-key (retain generated CA key)
    * -multiple (generate multiple certificates)
    * -in (generate certificates from an input file)
then the output will be be a zip file containing individual certificate/key files

Enter password for CA (elastic-stack-ca.p12) :
Please enter the desired output file [node01.p12]:
Enter password for node01.p12 :

Certificates written to /home/elastic/elasticsearch/node01.p12

This file should be properly secured as it contains the private key for
your instance.

This file is a self contained file and can be copied and used 'as is'
For each Elastic product that you wish to configure, you should copy
this '.p12' file to the relevant configuration directory
and then follow the SSL configuration instructions in the product guide.

For client applications, you may only need to copy the CA certificate and
configure the client to trust this certificate.
```

Ignore the reflective WARNINGS.
Accept the defaults.

This file can be your keystore and truststore.

</details>
<hr>

## Enable SSL on the Transport Network 

The Transport Network is the network between nodes within a cluster (or between clusters).  On a single-node cluster you do not need this.
On a production cluster you WILL need this.

<details>
  <summary>View Solution (click to reveal)</summary>
  
### Enable SSL on Transport Network

https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#encrypt-internode-communication

v7.2 https://www.elastic.co/guide/en/elasticsearch/reference/7.2/encrypting-internode.html

Edit the elasticsearch.yml file and set
```yaml
cluster.initial_master_nodes: ["node01"]
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: ${node.name}.p12
xpack.security.transport.ssl.truststore.path: ${node.name}.p12
```

`${node.name}` is an elasticsearch environment variable.  

If you set a password when creating the node certificate then you need to do the following, where it will ask you for the password each time.

```bash
$ ./bin/elasticsearch-keystore create 

$ ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password

$ ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

Restart Elasticsearch afterwards.

</details>
<hr>

## Change the built-in user default passwords

Serveral accounts needs their passwords changed

<details>
  <summary>View Solution (click to reveal)</summary>

Use: `./bin/elasticsearch-setup-passwords`

`auto` - Uses randomly generated passwords

`interactive` - Uses passwords entered by a user


```bash
[elastic@centos8streams elasticsearch]$ pwd
/home/elastic/elasticsearch
[elastic@centos8streams elasticsearch]$ ./bin/elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]

```

</details>

<hr>

### Enable SSL on the HTTP Network

This is the HTTPS used to access port 9200. eg. when using curl or python.
It is advised that default user passwords should be changed before doing this. (see previous section)

https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup-https.html

v7.2 https://www.elastic.co/guide/en/elasticsearch/reference/7.2/configuring-tls.html#tls-http

<details>
  <summary>View Solution (click to reveal)</summary>

You should have created the CA certs and keystores before (see previous sections)

Edit the elasticsearch.yml file and set
```yaml
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: ${node.name}.p12
xpack.security.http.ssl.truststore.path: ${node.name}.p12
```

Restart Elasticsearch afterwards.

</details>

<hr>

# Define role-based access control using Elasticsearch Security

>  :warning: :warning: :warning: IMPORTANT NOTE: from here on it is assumed you have a working kibana node to work from the "development console" 

> See: [Running test servers on docker](Test_servers_on_docker.md)

To do this you need to:

- create a role
- create a user

## Create a role
