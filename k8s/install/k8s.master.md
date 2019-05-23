# 安装Master节点组件

## 1、移动二进制到bin目录下
```text
[root@server1 ~]# tar xf /root/kubernetes-server-linux-amd64.tar.gz
[root@server1 kubernetes]# mv /root/kubernetes/server/bin/{kube-apiserver,kube-scheduler,kube-controller-manager,kubectl,kubelet} /usr/local/kubernetes/bin

```
## 2、apiserver安装
###（1）apiserver配置文件
```text
vim /usr/local/kubernetes/config/kube-apiserver 
```
```text
#启用日志标准错误
KUBE_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE_LOG_LEVEL="--v=4"
#Etcd服务地址
KUBE_ETCD_SERVERS="--etcd-servers=http://10.10.10.1:2379"
#API服务监听地址
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
#API服务监听端口
KUBE_API_PORT="--insecure-port=8080"
#对集群中成员提供API服务地址
KUBE_ADVERTISE_ADDR="--advertise-address=10.10.10.1"
#允许容器请求特权模式，默认false
KUBE_ALLOW_PRIV="--allow-privileged=false"
#集群分配的IP范围，自定义但是要跟后面的kubelet（服务节点）的配置DNS在一个区间
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.0.0.0/24" 
```
### （2）apiserver systemd配置文件
```text
vim /usr/lib/systemd/system/kube-apiserver.service
```
```text
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/usr/local/kubernetes/config/kube-apiserver
ExecStart=/usr/local/kubernetes/bin/kube-apiserver \
${KUBE_LOGTOSTDERR} \
${KUBE_LOG_LEVEL} \
${KUBE_ETCD_SERVERS} \
${KUBE_API_ADDRESS} \
${KUBE_API_PORT} \
${KUBE_ADVERTISE_ADDR} \
${KUBE_ALLOW_PRIV} \
${KUBE_SERVICE_ADDRESSES}
Restart=on-failure
[Install]
WantedBy=multi-user.target

```
```text
kube-apiserver --logtostderr=true --v=4 --etcd-servers=http://10.112.118.166:2379 --insecure-bind-address=0.0.0.0 --insecure-port=8080 --advertise-address=10.112.118.166 --service-cluster-ip-range=10.0.0.0/24 &
```

## 3、安装Scheduler
### 1）scheduler配置文件
    [root@server1 ~]# vim /usr/local/kubernetes/config/kube-scheduler 
    KUBE_LOGTOSTDERR="--logtostderr=true"
    KUBE_LOG_LEVEL="--v=4"
    KUBE_MASTER="--master=10.10.10.1:8080"
    KUBE_LEADER_ELECT="--leader-elect"
    
### 2）scheduler systemd配置文件
```
vim /usr/lib/systemd/system/kube-scheduler.service 
```

```text
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/usr/local/kubernetes/config/kube-scheduler
ExecStart=/usr/local/kubernetes/bin/kube-scheduler \
${KUBE_LOGTOSTDERR} \
${KUBE_LOG_LEVEL} \
${KUBE_MASTER} \
${KUBE_LEADER_ELECT}
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

```text
[root@server1 ~]# systemctl enable kube-scheduler 
[root@server1 ~]# systemctl start kube-scheduler 

```
```text
kube-scheduler --logtostderr=true --v=4 --master=10.112.118.166:8080 --leader-elect &
```

## 4、controller-manager安装
### 1）controller-manage配置文件
```
vim /usr/local/kubernetes/config/kube-controller-manager 
```
```
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=4"
KUBE_MASTER="--master=10.10.10.1:8080"
```

### 2）controller-manage systemd配置文件
```text

vim /usr/lib/systemd/system/kube-controller-manager.service
```

```text
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/usr/local/kubernetes/config/kube-controller-manager
ExecStart=/usr/local/kubernetes/bin/kube-controller-manager \
${KUBE_LOGTOSTDERR} \
${KUBE_LOG_LEVEL} \
${KUBE_MASTER} \
${KUBE_LEADER_ELECT}
Restart=on-failure
[Install]
WantedBy=multi-user.target

```
```text
[root@server1 ~]# systemctl enable kube-controller-manager
[root@server1 ~]# systemctl start kube-controller-manager
```

```text
kube-controller-manager --logtostderr=true --v=4 --master=10.112.118.166:8080 &
```

## 5、安装kubelet
安装完此服务，kubectl get nodes就可以看到Master，也可以选择不安装！！！
### 1）kubeconfig配置文件
```
vim /usr/local/kubernetes/config/kubelet.kubeconfig
```
```
apiVersion: v1
kind: Config
clusters:
  - cluster:
      server: http://10.112.118.166:8080                ###Master的IP，即自身IP
    name: local
contexts:
  - context:
      cluster: local
    name: local
current-context: local
``` 

### 2）kubelet配置文件
```text
vim /usr/local/kubernetes/config/kubelet
``` 
```text
# 启用日志标准错误
KUBE_LOGTOSTDERR="--logtostderr=true"
# 日志级别
KUBE_LOG_LEVEL="--v=4"
# Kubelet服务IP地址
NODE_ADDRESS="--address=10.10.10.1"
# Kubelet服务端口
NODE_PORT="--port=10250"
# 自定义节点名称
NODE_HOSTNAME="--hostname-override=10.10.10.1"
# kubeconfig路径，指定连接API服务器
KUBELET_KUBECONFIG="--kubeconfig=/usr/local/kubernetes/config/kubelet.kubeconfig"
# 允许容器请求特权模式，默认false
KUBE_ALLOW_PRIV="--allow-privileged=false"
# DNS信息，DNS的IP
KUBELET_DNS_IP="--cluster-dns=10.0.0.2"
KUBELET_DNS_DOMAIN="--cluster-domain=cluster.local"
# 禁用使用Swap
KUBELET_SWAP="--fail-swap-on=false"
```
### 2）kubelet配置文件
```
vim /usr/local/kubernetes/config/kubelet
```
```text
# 启用日志标准错误
KUBE_LOGTOSTDERR="--logtostderr=true"
# 日志级别
KUBE_LOG_LEVEL="--v=4"
# Kubelet服务IP地址
NODE_ADDRESS="--address=10.112.118.166"
# Kubelet服务端口
NODE_PORT="--port=10250"
# 自定义节点名称
NODE_HOSTNAME="--hostname-override=10.112.118.166"
# kubeconfig路径，指定连接API服务器
KUBELET_KUBECONFIG="--kubeconfig=/usr/local/kubernetes/config/kubelet.kubeconfig"
# 允许容器请求特权模式，默认false
KUBE_ALLOW_PRIV="--allow-privileged=false"
# DNS信息，DNS的IP
KUBELET_DNS_IP="--cluster-dns=10.0.0.2"
KUBELET_DNS_DOMAIN="--cluster-domain=cluster.local"
# 禁用使用Swap
KUBELET_SWAP="--fail-swap-on=false"
```

## 3）kubelet systemd配置文件
```
vim /usr/lib/systemd/system/kubelet.service
```
```text
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service
[Service]
EnvironmentFile=-/usr/local/kubernetes/config/kubelet
ExecStart=/usr/local/kubernetes/bin/kubelet \
${KUBE_LOGTOSTDERR} \
${KUBE_LOG_LEVEL} \
${NODE_ADDRESS} \
${NODE_PORT} \
${NODE_HOSTNAME} \
${KUBELET_KUBECONFIG} \
${KUBE_ALLOW_PRIV} \
${KUBELET_DNS_IP} \
${KUBELET_DNS_DOMAIN} \
${KUBELET_SWAP}
Restart=on-failure
KillMode=process
[Install]
WantedBy=multi-user.target
```
### 4）启动服务，并设置开机启动：
```text
[root@server1 ~]# swapoff -a                 ###启动之前要先关闭swap
[root@server1 ~]# systemctl enable kubelet
[root@server1 ~]# systemctl start kubelet  
```
```text
kubelet --logtostderr=true --v=4 --address=10.112.118.166 --port=10250 --hostname-override=10.112.118.166 --kubeconfig=/usr/k8s.config/kubelet.cfg --cluster-dns=10.0.0.2 --cluster-domain=cluster.local --fail-swap-on=false &

```

# 六、安装node(server2和server3)
 1、移动二进制到bin目录
 [root@server2 ~]# tar xf /root/kubernetes-node-linux-amd64.tar.gz 
 [root@server2 ~]# mv /root/kubernetes/node/bin/{kubelet,kube-proxy} /usr/local/kubernetes/bin/
 
## 2、安装kubelet

## 3、安装proxy
### 1）proxy配置文件
```text
cat /usr/local/kubernetes/config/kube-proxy
``` 
```text
#启用日志标准错误
KUBE_LOGTOSTDERR="--logtostderr=true"
#日志级别
KUBE_LOG_LEVEL="--v=4"
#自定义节点名称（自身IP）
NODE_HOSTNAME="--hostname-override=10.10.10.2"
#API服务地址（MasterIP）
KUBE_MASTER="--master=http://10.10.10.1:8080"
```
### 2）proxy systemd配置文件
```
cat /usr/lib/systemd/system/kube-proxy.service
```
```text
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=-/usr/local/kubernetes/config/kube-proxy
ExecStart=/usr/local/kubernetes/bin/kube-proxy \
${KUBE_LOGTOSTDERR} \
${KUBE_LOG_LEVEL} \
${NODE_HOSTNAME} \
${KUBE_MASTER}
Restart=on-failure
[Install]
WantedBy=multi-user.target

```
```text
[root@server2 ~]# systemctl enable kube-proxy
[root@server2 ~]# systemctl restart kube-proxy
```
 ```text
kube-proxy --logtostderr=true --v=4 --hostname-override=10.112.118.166 --master=http://10.112.118.166:8080 &
```