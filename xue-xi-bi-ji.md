1、环境

1708的centos7

yum update

vim /etc/yum.repos.d/docker.repo

```js
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



2、下载

docker pull microhq/micro

