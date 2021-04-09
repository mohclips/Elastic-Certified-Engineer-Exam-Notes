
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
$ cd

$ wget https://raw.githubusercontent.com/mohclips/Elastic-Certified-Engineer-Exam-Notes/main/DockerComposeExamples/1es-1kb-xpackSec.yml
```

> Username: elastic

> Password: Password01

# Boot up a server 

```bash
[elastic@centos8streams]$ docker-compose -f 1es-1kb-xpackSec.yml up
...

```

# Add data to query

> See: [Importing Sample data](Importing_sample_data.md)
