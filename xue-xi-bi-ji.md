1708的centos7

yum update

vim /etc/yum.repos.d/docker.repo

```bash

[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/

enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
```

yum install docker-engine -y

systemctl enable docker.service

systemctl start docker

docker info

