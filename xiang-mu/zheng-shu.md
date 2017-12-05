CN: kube-apiserver 从证书中提取该字段作为请求的用户名 \(User Name\)；浏览器使用该字段验证网站是否合法

O: kube-apiserver 从证书中提取该字段作为请求用户所属的组 \(Group\)

### 

```bash
#!/bin/bash
source /etc/profile


#CN: kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法
#O: kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)

#创建 CA 配置文件

# 创建 CA 证书签名请求
#生成 CA 证书和私钥
#  创建 kubernetes 证书签名请求文件
#生成 kubernetes 证书和私钥
#创建 admin 证书签名请求文件 admin-csr.json
#生成 admin 证书和私钥
#创建 kube-proxy 证书签名请求文件 kube-proxy-csr.json
#生成 kube-proxy 客户端证书和私钥
#生成token
#生成kubelet kubeconfig文件
#7. 生成kube-proxy kubeconfig文件

mkdir -p /etc/k8s/ssl
cd /etc/k8s/ssl

cat <<EOF > ca-config.json
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
EOF

cat <<EOF > ca-csr.json
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
EOF

cat <<EOF > kubernetes-csr.json
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "192.168.74.47",
      "192.168.74.48",
      "192.168.74.49",
      "192.168.74.50",
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
EOF

cat <<EOF > admin-csr.json
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
EOF

cat <<EOF > kube-proxy-csr.json
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
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy

export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
echo ${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap" > token.csv

kubectl config set-cluster kubernetes --certificate-authority=ca.pem  --embed-certs=true --server=https://192.168.74.47:6443 --kubeconfig=kubelet-bootstrap.kubeconfig

kubectl config set-credentials kubelet-bootstrap --token=${BOOTSTRAP_TOKEN} --kubeconfig=kubelet-bootstrap.kubeconfig

kubectl config set-context default  --cluster=kubernetes --user=kubelet-bootstrap  --kubeconfig=kubelet-bootstrap.kubeconfig

kubectl config use-context default --kubeconfig=kubelet-bootstrap.kubeconfig


kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.74.47:6443 --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials kube-proxy --client-certificate=kube-proxy.pem  --client-key=kube-proxy-key.pem --embed-certs=true --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default --cluster=kubernetes  --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig

mv *.kubeconfig token.csv ../

#8. 拷贝证书、kubeconfig和token文件
mkdir -p /etc/k8s/

scp /etc/k8s/* root@192.168.74.48:/etc/k8s/
scp /etc/k8s/* root@192.168.74.49:/etc/k8s/
scp /etc/k8s/* root@192.168.74.50:/etc/k8s/









echo "ssl created"
echo "--------------------------------------------------------------------"

```



