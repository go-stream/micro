```bash
systemctl stop firewalld.service
systemctl disable firewalld.service
firewall-cmd --state
vi /etc/selinux/config
    #将SELINUX=enforcing改为SELINUX=disabled 
reboot

yum install -y  git wget gcc gcc-c++ kernel-devel unzip vim  ntp ntpdate
ntpdate cn.pool.ntp.org
hwclock --systohc
echo "请确定/opt/中已经包含了go1.9.2.linux-amd64.tar.gz文件?"
read
tar zxvf go1.9.2.linux-amd64.tar.gz
mv go /usr/local/ 
mkdir -p /Golang/src /Golang/pkg /Golang/bin
cp /etc/profile /etc/profile_bk

 cat <<EOF >> /etc/profile
export GOROOT=/usr/local/go 
export GOBIN=\$GOROOT/bin
export GOPKG=\$GOROOT/pkg/tool/linux_amd64 
export GOARCH=amd64
export GOOS=linux
export GOPATH=/Golang
export PATH=\$PATH:\$GOBIN:\$GOPKG:\$GOPATH/bin
EOF

source /etc/profile
go version


 mkdir -p /Golang/src/golang.org/x/
cd /Golang/src/golang.org/x/
git clone https://github.com/golang/text.git
git clone https://github.com/golang/net.git
git clone https://github.com/golang/crypto.git
echo "golang over"
echo "--------------------------------------------------------------------"
```



