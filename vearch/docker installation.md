docker installation

remove old version

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

Install using repository

```
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io
```

If error container-selinux >= 2.9, download it from [here](http://mirror.centos.org/centos/7/extras/x86_64/Packages/)

```
rpm -ivh container-selinux-2.107-3.el7.noarch.rpm
```

then install docker again

```
sudo yum install docker-ce docker-ce-cli containerd.io
```

Or download them [here](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)

https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.12-3.el7.x86_64.rpm

https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.12-3.el7.x86_64.rpmhttps://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm





```
systemctl start docker
docker --version
```

modify default path of docker

```text
vi /etc/docker/daemon.json 
{
  "data-root": "/home/docker"
}

systemctl restart docker
```

