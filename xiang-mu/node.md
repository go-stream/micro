docker:

```bash
 yum remove docker docker-common docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-edge
yum-config-manager --enable docker-ce-test
yum install docker-ce


yum install -y flannel

systemctl restart docker
```

# https://github.com/asinglestep/k8s-learn/blob/master/v1.9.0%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.md

# 零. 配置

```
Centos7.4
Kubernetes version: 1.9.0
Etcd version: 3.2.0+git
Docker version 17.11.0-ce, build 1caf76c
```

| ip | 说明 |
| :--- | :--- |
| 192.168.74.47 | etcd、master |
| 192.168.74.48 | node |
| 192.168.74.49 | node |
| 192.168.74.50 | node |

# 一. 安装

## 1. 安装etcd

```
$ go get github.com/coreos/etcd
$ cd $GOPATH/src/github.com/coreos/etcd
$ ./build
```

## 2. 安装k8s

```
$ go get -d k8s.io/kubernetes
$ cd $GOPATH/src/k8s.io/kubernetes
$ make
```

## 3. 安装CFSSL

```
$ wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

## 4. 安装docker-ce

```
(1). 移除旧的docker、docker-engine
$ sudo yum remove docker docker-common docker-selinux docker-engine

(2). 安装依赖
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

(3). 添加源
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

(4). 安装docker-ce
$ sudo yum install docker-ce
```

## 5. 安装flanneld

```
$ sudo yum install -y flannel
```

# 二、生成证书和kubeconfig文件

## 1. 创建 CA

### \(0\). 说明

```
CN: kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法
O: kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)
```

### \(1\). 创建 CA 配置文件 ca-config.json

```
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
```

### \(2\). 创建 CA 证书签名请求 ca-csr.json

```
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

### \(3\). 生成 CA 证书和私钥

```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

## 2. 创建 kubernetes 证书

### \(1\). 创建 kubernetes 证书签名请求文件 kubernetes-csr.json

```
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "172.16.221.129",
      "172.16.221.130",
      "172.16.221.131",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```

注： 如果 hosts 字段不为空则需要指定授权使用该证书的 IP 或域名列表

### \(2\). 生成 kubernetes 证书和私钥

```
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
```

## 3. 创建 admin 证书

### \(1\). 创建 admin 证书签名请求文件 admin-csr.json

```
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
```

### \(2\). 生成 admin 证书和私钥

```
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
```

## 4. 创建 kube-proxy 证书

### \(1\). 创建 kube-proxy 证书签名请求文件 kube-proxy-csr.json

```
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

### \(2\). 生成 kube-proxy 客户端证书和私钥

```
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

## 5. 生成token

```
$ export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
$ echo ${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap" 
>
 token.csv
```

## 6. 生成kubelet kubeconfig文件

```
$ kubectl config set-cluster kubernetes --certificate-authority=ca.pem  --embed-certs=true --server=https://172.16.221.129:6443 --kubeconfig=kubelet-bootstrap.kubeconfig

$ kubectl config set-credentials kubelet-bootstrap --token=${BOOTSTRAP_TOKEN} --kubeconfig=kubelet-bootstrap.kubeconfig

$ kubectl config set-context default  --cluster=kubernetes --user=kubelet-bootstrap  --kubeconfig=kubelet-bootstrap.kubeconfig

$ kubectl config use-context default --kubeconfig=kubelet-bootstrap.kubeconfig
```

## 7. 生成kube-proxy kubeconfig文件

```
$ kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://172.16.221.129:6443 --kubeconfig=kube-proxy.kubeconfig

$ kubectl config set-credentials kube-proxy --client-certificate=kube-proxy.pem  --client-key=kube-proxy-key.pem --embed-certs=true --kubeconfig=kube-proxy.kubeconfig

$ kubectl config set-context default --cluster=kubernetes  --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig

$ kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

## 8. 拷贝证书、kubeconfig和token文件

```
(1). 进入到生成证书的目录下
for ip in $@
do
    scp k8s.tar root@$ip:/etc/k8s/ssl
    ssh root@$ip 'cd /etc/k8s/ssl 
&
&
 tar xf k8s.tar 
&
&
 mv *.kubeconfig token.csv ../ 
&
&
 rm k8s.tar'
done
```

# 三、启动服务

## 1. 启动master

### \(1\). 关闭防火墙

```
$ sudo systemctl stop firewalld.service
$ sudo systemctl disable firewalld.service // 禁止开机启动
```

### \(2\). 启动etcd

```
$ $GOPATH/src/github.com/coreos/etcd/bin/etcd --advertise-client-urls=http://0.0.0.0:2379 --listen-client-urls=http://0.0.0.0:2379
```

### \(3\). 启动docker

```
$ systemctl start docker
```

### \(4\). 启动kube-apiserver

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-apiserver --advertise-address=0.0.0.0 --bind-address=0.0.0.0 --insecure-bind-address=0.0.0.0 --etcd-servers=http://127.0.0.1:2379 --service-cluster-ip-range=10.254.0.0/16 --admission-control=ServiceAccount,NamespaceLifecycle,NamespaceExists,LimitRanger,ResourceQuota --authorization-mode=RBAC --runtime-config=rbac.authorization.k8s.io/v1beta1 --kubelet-https=true --token-auth-file=/etc/k8s/token.csv --service-node-port-range=30000-32767 --tls-cert-file=/etc/k8s/ssl/kubernetes.pem --tls-private-key-file=/etc/k8s/ssl/kubernetes-key.pem --client-ca-file=/etc/k8s/ssl/ca.pem --service-account-key-file=/etc/k8s/ssl/ca-key.pem
```

#### 注:

```
--authorization-mode=RBAC 指定在安全端口使用 RBAC 授权模式，拒绝未通过授权的请求
```

### \(5\). 启动kube-controller-manager

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-controller-manager --address=127.0.0.1 --service-cluster-ip-range=10.254.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/k8s/ssl/ca.pem --cluster-signing-key-file=/etc/k8s/ssl/ca-key.pem --service-account-private-key-file=/etc/k8s/ssl/ca-key.pem --root-ca-file=/etc/k8s/ssl/ca.pem --leader-elect=true --master=http://127.0.0.1:8080
```

### \(6\). 启动kube-scheduler

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-scheduler --leader-elect=true --address=127.0.0.1 --master=http://127.0.0.1:8080
```

### \(7\). 将token.csv文件中的kubelet-bootstrap 用户赋予 system:node-bootstrapper cluster 角色\(role\)，kubelet 才能有权限创建认证请求

```
$ kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
```

### \(8\). 启动kubelet

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kubelet --address=172.16.221.131 --hostname-override=172.16.221.131 --pod-infra-container-image=singlestep/pod-infrastructure --cgroup-driver=cgroupfs --cluster-dns=10.254.0.2 --experimental-bootstrap-kubeconfig=/etc/k8s/kubelet-bootstrap.kubeconfig --kubeconfig=/etc/k8s/kubelet.kubeconfig --require-kubeconfig --cert-dir=/etc/k8s/ssl --cluster-domain=cluster.local. --hairpin-mode promiscuous-bridge --serialize-image-pulls=false --fail-swap-on=false --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

#### 注:

```
报csr for this node already exists, reusing错时
(1). 查看未授权的 CSR 请求
$ kubectl get csr
(2). 通过 CSR 请求
$ kubectl certificate approve XXX(未授权的csr名字)
(3). 报 Unable to register node "172.16.221.131" with API server: nodes is forbidden: User "system:node:172.16.221.131" cannot create nodes at the cluster scope错时，给node的用户或组添加system:node权限
kubectl create clusterrolebinding node-172.16.221.131 --clusterrole=system:node  --user=system:node:172.16.221.131
(4). 报Failed to get system container stats for "/user.slice/user-1000.slice/session-40.scope": failed to get cgroup stats for "/user.slice/user-1000.slice/session-40.scope": failed to get container info for "/user.slice/user-1000.slice/session-40.scope": unknown container "/user.slice/user-1000.slice/session-40.scope"错时，启动kubelet加 --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice 参数

出现Successfully registered node XXX表示注册成功
```

### \(9\). 启动kube-proxy

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-proxy --bind-address=172.16.221.131 --hostname-override=172.16.221.131 --kubeconfig=/etc/k8s/kube-proxy.kubeconfig --cluster-cidr=10.254.0.0/16
```

### \(10\). 启动flanneld

```
(1). 修改配置
$ sudo vim /etc/sysconfig/flanneld
修改
FLANNEL_ETCD_ENDPOINTS="http://172.16.221.131:2379"
FLANNEL_ETCD_PREFIX="/kube-centos/network"

(2). 在etcd中创建网络配置
$ etcdctl set /kube-centos/network/config '{ "Network" : "10.1.0.0/16" }'

(3). 启动flanneld
$ sudo systemctl start flanneld.service

(4). 重启docker
防止主机重启后 docker 自动重启时加载不到该上述环境变量，修改docker.service文件

$ sudo vim /usr/lib/systemd/system/docker.service

在ExecStart之前添加
EnvironmentFile=-/run/flannel/docker
EnvironmentFile=-/run/flannel/subnet.env

修改ExecStart=/usr/bin/dockerd \
          $DOCKER_OPT_BIP \
          $DOCKER_OPT_IPMASQ \
          $DOCKER_OPT_MTU

重启docker
$ systemctl daemon-reload 
$ sudo systemctl restart docker.service

查看docker0和flannel网桥是否在同一个子网中
$ ifconfig
docker0: flags=4099
<
UP,BROADCAST,MULTICAST
>
  mtu 1500
        inet 10.1.10.1  netmask 255.255.255.0  broadcast 0.0.0.0
flannel0: flags=4305
<
UP,POINTOPOINT,RUNNING,NOARP,MULTICAST
>
  mtu 1472
        inet 10.1.10.0  netmask 255.255.0.0  destination 10.1.10.0

(5). 重启kubelet
```

## 2. 启动node\(172.16.221.130\)

### \(1\). 关闭防火墙

```
$ sudo systemctl stop firewalld.service
$ sudo systemctl disable firewalld.service // 禁止开机启动
```

### \(2\). 启动docker

```
$ systemctl start docker
```

### \(3\). 启动kubelet

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kubelet --address=172.16.221.130 --hostname-override=172.16.221.130 --pod-infra-container-image=singlestep/pod-infrastructure --cgroup-driver=cgroupfs --cluster-dns=10.254.0.2 --experimental-bootstrap-kubeconfig=/etc/k8s/kubelet-bootstrap.kubeconfig --kubeconfig=/etc/k8s/kubelet.kubeconfig --require-kubeconfig --cert-dir=/etc/k8s/ssl --cluster-domain=cluster.local. --hairpin-mode promiscuous-bridge --serialize-image-pulls=false --fail-swap-on=false --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

### \(4\). 启动kube-proxy

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-proxy --bind-address=172.16.221.130 --hostname-override=172.16.221.130 --kubeconfig=/etc/k8s/kube-proxy.kubeconfig --cluster-cidr=10.254.0.0/16
```

### \(5\). 启动flanneld

```
(1). 修改配置
$ sudo vim /etc/sysconfig/flanneld
修改
FLANNEL_ETCD_ENDPOINTS="http://172.16.221.131:2379"
FLANNEL_ETCD_PREFIX="/kube-centos/network"

(2). 启动flanneld
$ sudo systemctl start flanneld.service

(3). 重启docker
防止主机重启后 docker 自动重启时加载不到该上述环境变量，修改docker.service文件
$ sudo vim /usr/lib/systemd/system/docker.service

在ExecStart之前添加
EnvironmentFile=-/run/flannel/docker
EnvironmentFile=-/run/flannel/subnet.env

修改ExecStart=/usr/bin/dockerd \
          $DOCKER_OPT_BIP \
          $DOCKER_OPT_IPMASQ \
          $DOCKER_OPT_MTU

重启docker
$ systemctl daemon-reload 
$ sudo systemctl restart docker.service

查看docker0和flannel网桥是否在同一个子网中
$ ifconfig
docker0: flags=4099
<
UP,BROADCAST,MULTICAST
>
  mtu 1500
        inet 10.1.1.1  netmask 255.255.255.0  broadcast 0.0.0.0
flannel0: flags=4305
<
UP,POINTOPOINT,RUNNING,NOARP,MULTICAST
>
  mtu 1472
        inet 10.1.1.0  netmask 255.255.0.0  destination 10.1.1.0

(4). 重启kubelet
```

## 3. 启动node\(172.16.221.129\)

### \(1\). 关闭防火墙

```
$ sudo systemctl stop firewalld.service
$ sudo systemctl disable firewalld.service // 禁止开机启动
```

### \(2\). 启动docker

```
$ systemctl start docker
```

### \(3\). 启动kubelet

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kubelet --address=172.16.221.129 --hostname-override=172.16.221.129 --pod-infra-container-image=singlestep/pod-infrastructure --cgroup-driver=cgroupfs --cluster-dns=10.254.0.2 --experimental-bootstrap-kubeconfig=/etc/k8s/kubelet-bootstrap.kubeconfig --kubeconfig=/etc/k8s/kubelet.kubeconfig --require-kubeconfig --cert-dir=/etc/k8s/ssl --cluster-domain=cluster.local. --hairpin-mode promiscuous-bridge --serialize-image-pulls=false --fail-swap-on=false --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

### \(4\). 启动kube-proxy

```
$ sudo $GOPATH/src/k8s.io/kubernetes/_output/bin/kube-proxy --bind-address=172.16.221.129 --hostname-override=172.16.221.129 --kubeconfig=/etc/k8s/kube-proxy.kubeconfig --cluster-cidr=10.254.0.0/16
```

### \(5\). 启动flanneld

```
(1). 修改配置
$ sudo vim /etc/sysconfig/flanneld
修改
FLANNEL_ETCD_ENDPOINTS="http://172.16.221.131:2379"
FLANNEL_ETCD_PREFIX="/kube-centos/network"

(2). 启动flanneld
$ sudo systemctl start flanneld.service

(3). 重启docker
防止主机重启后 docker 自动重启时加载不到该上述环境变量，修改docker.service文件

$ sudo vim /usr/lib/systemd/system/docker.service

在ExecStart之前添加
EnvironmentFile=-/run/flannel/docker
EnvironmentFile=-/run/flannel/subnet.env

修改ExecStart=/usr/bin/dockerd \
          $DOCKER_OPT_BIP \
          $DOCKER_OPT_IPMASQ \
          $DOCKER_OPT_MTU

重启docker
$ systemctl daemon-reload 
$ sudo systemctl restart docker.service

查看docker0和flannel网桥是否在同一个子网中
$ ifconfig
docker0: flags=4099
<
UP,BROADCAST,MULTICAST
>
  mtu 1500
        inet 10.1.59.1  netmask 255.255.255.0  broadcast 0.0.0.0
flannel0: flags=4305
<
UP,POINTOPOINT,RUNNING,NOARP,MULTICAST
>
  mtu 1472
        inet 10.1.59.0  netmask 255.255.0.0  destination 10.1.59.0

(4). 重启kubelet
```

## 4. 验证

```
(1). $ etcdctl ls /kube-centos/network/subnets
/kube-centos/network/subnets/10.1.59.0-24
/kube-centos/network/subnets/10.1.1.0-24
/kube-centos/network/subnets/10.1.10.0-24

(2). $ etcdctl get /kube-centos/network/subnets/10.1.59.0-24
{"PublicIP":"172.16.221.129"}

(3). $ etcdctl get /kube-centos/network/subnets/10.1.1.0-24
{"PublicIP":"172.16.221.130"}

(4). $ etcdctl get /kube-centos/network/subnets/10.1.10.0-24
{"PublicIP":"172.16.221.131"}

(5). 下载nginx镜像
$ docker pull nginx

(6). $ kubectl run nginx --replicas=2 --labels="run=load-balancer-example" --image=nginx  --port=80

(7). $ kubectl expose deployment nginx --type=NodePort --name=example-service

(8). $ kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
example-service   NodePort    10.254.109.88   
<
none
>
        80:31183/TCP   3h
kubernetes        ClusterIP   10.254.0.1      
<
none
>
        443/TCP        1d

(9). 访问172.16.221.129:31183、172.16.221.130:31183、172.16.221.131:31183
```

## 5. 问题总结

### \(1\). flannel网络ping不通

```
$ kubectl describe svc example-service
Name:                     example-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              
<
none
>

Selector:                 run=load-balancer-example
Type:                     NodePort
IP:                       10.254.39.84
Port:                     
<
unset
>
  80/TCP
TargetPort:               80/TCP
NodePort:                 
<
unset
>
  30110/TCP
Endpoints:                10.1.69.3:80,10.1.99.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   
<
none
>


$ ping 10.1.99.2
PING 10.1.99.2 (10.1.99.2) 56(84) bytes of data.
^C
--- 10.1.99.2 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1000ms

解决方法:
sudo iptables -I FORWARD -i flannel0  -j ACCEPT
```

* © 2017
  GitHub
  , Inc.
* [Terms](https://github.com/site/terms)
* [Privacy](https://github.com/site/privacy)
* [Security](https://github.com/security)
* [Status](https://status.github.com/)
* [Help](https://help.github.com/)

* [Contact GitHub](https://github.com/contact)

* [API](https://developer.github.com/)

* [Training](https://training.github.com/)

* [Shop](https://shop.github.com/)

* [Blog](https://github.com/blog)

* [About](https://github.com/about)



