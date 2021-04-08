
# Install docker and docker-compose

As root run the following

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

chmod +x /usr/local/bin/docker-compose

docker-compose --version

ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

# Copy example server git repo
