# flannel安装

## 源码
### git
url: ```https://github.com/coreos/flannel```

## 二进制
url: ```https://github.com/coreos/flannel/releases/download```

### 1、移动二进制到bin目录
    [root@server1 ~]# tar xf flannel-v0.7.1-linux-amd64.tar.gz 
    [root@server1 ~]# mv /root/{flanneld,mk-docker-opts.sh} /usr/local/kubernetes/bin

### 2、flannel配置文件
```
vim /usr/local/kubernetes/config/flanneld
```
```text
# Flanneld configuration options  
# etcd url location.  Point this to the server where etcd runs，自身IP
FLANNEL_ETCD="http://10.10.10.1:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment，etcd-key的目录
FLANNEL_ETCD_KEY="/atomic.io/network"

# Any additional options that you want to pass，根据自己的网卡名填写
FLANNEL_OPTIONS="--iface=eth0"

```
  
### 3、flannel systemd配置文件
```
vim /usr/lib/systemd/system/flanneld.service 
```
```text
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/usr/local/kubernetes/config/flanneld
EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=/usr/local/kubernetes/bin/flanneld -etcd-endpoints=${FLANNEL_ETCD} -etcd-prefix=${FLANNEL_ETCD_KEY} $FLANNEL_OPTIONS
ExecStartPost=/usr/local/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service
```
### 4、设置etcd-key

*注意： 在一台上设置即可，会同步过去！！！*

```
etcdctl mkdir /atomic.io/network
```
###下面的IP跟你docker本身的IP地址一个网段
```text
[root@server1 ~]# etcdctl mk /atomic.io/network/config  "{ \"Network\": \"172.17.0.0/16\", \"SubnetLen\": 24, \"Backend\": { \"Type\": \"vxlan\" } }" 
  -- { "Network": "172.17.0.0/16", "SubnetLen": 24, "Backend": { "Type": "vxlan" } }
```
```text
flanneld -etcd-endpoints=http://10.112.118.166:2379 -etcd-prefix=/atomic.io/network &
```

### 5、设置docker配置
因为docker要使用flanneld，所以在其配置中加入EnvironmentFile=-/etc/sysconfig/flanneld，EnvironmentFile=-/run/flannel/subnet.env，--bip=${FLANNEL_SUBNET}

[root@server1 ~]# vim /usr/lib/systemd/system/docker.service

```text
dockerd --bip=172.17.77.1/24 &
```

## 8、集群测试
   如果Master中没有装kubelet，kubectl get nodes就看不到Master！！！
   
```text
[root@server1 ~]# kubectl get nodes
   NAME         STATUS    ROLES     AGE       VERSION
   10.10.10.1   Ready     <none>    1d        v1.8.13
   10.10.10.2   Ready     <none>    1d        v1.8.13
   10.10.10.3   Ready     <none>    1d        v1.8.13
   
[root@server1 ~]# kubectl get cs
   NAME                 STATUS    MESSAGE              ERROR
   controller-manager   Healthy   ok                   
   scheduler            Healthy   ok                   
   etcd-0               Healthy   {"health": "true"} 
```   
























