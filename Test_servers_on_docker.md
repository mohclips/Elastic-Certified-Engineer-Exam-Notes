
# Install docker and docker-compose

As root run the following:

```bash
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

dnf list docker-ce

dnf install docker-ce --nobest -y

systemctl start docker

systemctl enable docker

docker --version

docker run hello-world

dnf install curl -y

curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod 755 /usr/local/bin/docker-compose

docker-compose --version

ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

usermod -aG docker elastic

```

# Copy example server git repo

```bash
$ sudo dnf install git -y

$ cd

$ git clone https://github.com/glenacota/elastic-training-repo.git
```

# Boot up a server 


```bash
[elastic@centos8streams ~]$ cd elastic-training-repo/clusters-to-go/7.2.0/

[elastic@centos8streams 7.2.0]$ docker-compose -f 1es-1kibana.yml up
Creating network "720_elastic-net" with the default driver
Creating volume "720_esnode-data" with local driver
Pulling esnode (docker.elastic.co/elasticsearch/elasticsearch:7.2.0)...
7.2.0: Pulling from elasticsearch/elasticsearch
...

```

# Add data to query
