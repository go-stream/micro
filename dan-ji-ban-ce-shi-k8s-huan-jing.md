

```
yum install *rhsm*

yum install etcd kubernetes

vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'


vim /etc/kubernetes/apiserver 
把--admission_control 的ServiceAccount删除

systemctl start etcd 
systemctl restart docker 
systemctl start kube-apiserver
systemctl start kube-controller-manager  
systemctl start kube-scheduler  
systemctl start kubelet  
systemctl start kube-proxy  

docker pull nginx
kubectl run nginx --replicas=2 --labels="run=load-balancer-example" --image=nginx  --port=80
kubectl expose deployment nginx --type=NodePort --name=example-service
kubectl get svc
kubectl get node
kubectl get pod
kubectl describe pod xx


kubectl describe svc example-service

{
$ kubectl describe svc example-service
Name:                     example-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              <none>
Selector:                 run=load-balancer-example
Type:                     NodePort
IP:                       10.254.39.84
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30110/TCP
Endpoints:                10.1.69.3:80,10.1.99.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
}
{
error1:
StartContainer" for "POD" with ErrImagePull: "image pull failed for registry.access.redhat.com/rhel7/pod-infrastructure:latest, this may be because there are no credentials on this request.  details: (open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or directory

yum install *rhsm*


error2 ping 不同

iptables -I FORWARD -i flannel0  -j ACCEPT
}





































 
```









