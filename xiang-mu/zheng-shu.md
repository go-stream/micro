CN: kube-apiserver 从证书中提取该字段作为请求的用户名 \(User Name\)；浏览器使用该字段验证网站是否合法

O: kube-apiserver 从证书中提取该字段作为请求用户所属的组 \(Group\)

### \(1\). 创建 CA 配置文件 ca-config.json

```bash
 cat <<EOF >> ca-config.json
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
```



