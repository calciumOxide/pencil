# 安装ETCD

## 1.1 源码安装

  源码地址：```https://github.com/etcd-io/etcd```
  
  Golang编译 ......

## 1.1.5 二进制安装
tar xf /root/etcd-v3.1.7-linux-amd64.tar.gz

### etcd systemd配置文件
```text
vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target

[Service]
Type=simple
WorkingDirectory=/var/lib/etcd
EnvironmentFile=-/usr/local/kubernetes/config/etcd.conf
ExecStart=/usr/local/kubernetes/bin/etcd \
        --name=${ETCD_NAME} \
        --data-dir=${ETCD_DATA_DIR} \
        --listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
        --listen-client-urls=${ETCD_LISTEN_CLIENT_URLS} \
        --advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
        --initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
        --initial-cluster=${ETCD_INITIAL_CLUSTER} \
        --initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
        --initial-cluster-state=${ETCD_INITIAL_CLUSTER_STATE}
Type=notify

[Install]
WantedBy=multi-user.target

``` 
### etcd配置文件

```text
etcd配置文件
[root@server1 ~]# cat /usr/local/kubernetes/config/etcd.conf 
#[member]
ETCD_NAME="etcd01"                                          ###修改为本机对应的名字，etcd02，etcd03
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.10.10.1:2380"     ###修改为本机IP
ETCD_INITIAL_CLUSTER="etcd01=http://10.10.10.1:2380,etcd02=http://10.10.10.2:2380,etcd03=http://10.10.10.3:2380"    ###把IP更换成集群IP
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://10.10.10.1:2379"           ###修改为本机IP

```
  
## 1.2 安装包

```
ETCD_VER=v3.3.13

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
ETCDCTL_API=3 /tmp/etcd-download-test/etcdctl version
```

### 1.2.1 关闭防火墙
```$xslt
systemctl status firewalld.service
```

## 1.3 配置

  ### 1.3.1 版本选择
  ```$xslt
    /tmp/etcd-download-test/etcd --version
    ETCDCTL_API=3 
    ETCDCTL_API=2
    /tmp/etcd-download-test/etcdctl version
  ``` 
  ### 1.3.2 etcd 命令 
```/ # etcd --help
   usage: etcd [flags]
          start an etcd server
   
          etcd --version
          show the version of etcd
   
          etcd -h | --help
          show the help information about etcd
   
          etcd --config-file
          path to the server configuration file
   
          etcd gateway
          run the stateless pass-through etcd TCP connection forwarding proxy
   
          etcd grpc-proxy
          run the stateless etcd v3 gRPC L7 reverse proxy


member flags:

    --name 'default'
        human-readable name for this member.
    --data-dir '${name}.etcd'
        path to the data directory.
    --wal-dir ''
        path to the dedicated wal directory.
    --snapshot-count '100000'
        number of committed transactions to trigger a snapshot to disk.
    --heartbeat-interval '100'
        time (in milliseconds) of a heartbeat interval.
    --election-timeout '1000'
        time (in milliseconds) for an election to timeout. See tuning documentation for details.
    --initial-election-tick-advance 'true'
        whether to fast-forward initial election ticks on boot for faster election.
    --listen-peer-urls 'http://localhost:2380'
        list of URLs to listen on for peer traffic.
    --listen-client-urls 'http://localhost:2379'
        list of URLs to listen on for client traffic.
    --max-snapshots '5'
        maximum number of snapshot files to retain (0 is unlimited).
    --max-wals '5'
        maximum number of wal files to retain (0 is unlimited).
    --cors ''
        comma-separated whitelist of origins for CORS (cross-origin resource sharing).
    --quota-backend-bytes '0'
        raise alarms when backend size exceeds the given quota (0 defaults to low space quota).
    --max-txn-ops '128'
        maximum number of operations permitted in a transaction.
    --max-request-bytes '1572864'
        maximum client request size in bytes the server will accept.
    --grpc-keepalive-min-time '5s'
        minimum duration interval that a client should wait before pinging server.
    --grpc-keepalive-interval '2h'
        frequency duration of server-to-client ping to check if a connection is alive (0 to disable).
    --grpc-keepalive-timeout '20s'
        additional duration of wait before closing a non-responsive connection (0 to disable).



clustering flags:

    --initial-advertise-peer-urls 'http://localhost:2380'
        list of this member's peer URLs to advertise to the rest of the cluster.
    --initial-cluster 'default=http://localhost:2380'
        initial cluster configuration for bootstrapping.
    --initial-cluster-state 'new'
        initial cluster state ('new' or 'existing').
    --initial-cluster-token 'etcd-cluster'
        initial cluster token for the etcd cluster during bootstrap.
        Specifying this can protect you from unintended cross-cluster interaction when running multiple clusters.
    --advertise-client-urls 'http://localhost:2379'
        list of this member's client URLs to advertise to the public.
        The client URLs advertised should be accessible to machines that talk to etcd cluster. etcd client libraries parse these URLs to connect to the cluster.
    --discovery ''
        discovery URL used to bootstrap the cluster.
    --discovery-fallback 'proxy'
        expected behavior ('exit' or 'proxy') when discovery services fails.
        "proxy" supports v2 API only.
    --discovery-proxy ''
        HTTP proxy to use for traffic to discovery service.
    --discovery-srv ''
        dns srv domain used to bootstrap the cluster.
    --strict-reconfig-check 'true'
        reject reconfiguration requests that would cause quorum loss.
    --auto-compaction-retention '0'
        auto compaction retention length. 0 means disable auto compaction.
    --auto-compaction-mode 'periodic'
        interpret 'auto-compaction-retention' one of: periodic|revision. 'periodic' for duration based retention, defaulting to hours if no time unit is provided (e.g. '5m'). 'revision' for revision number based retention.
    --enable-v2 'true'
        Accept etcd V2 client requests.


proxy flags:
    "proxy" supports v2 API only.

    --proxy 'off'
        proxy mode setting ('off', 'readonly' or 'on').
    --proxy-failure-wait 5000
        time (in milliseconds) an endpoint will be held in a failed state.
    --proxy-refresh-interval 30000
        time (in milliseconds) of the endpoints refresh interval.
    --proxy-dial-timeout 1000
        time (in milliseconds) for a dial to timeout.
    --proxy-write-timeout 5000
        time (in milliseconds) for a write to timeout.
    --proxy-read-timeout 0
        time (in milliseconds) for a read to timeout.


    --ca-file '' [DEPRECATED]
        path to the client server TLS CA file. '-ca-file ca.crt' could be replaced by '-trusted-ca-file ca.crt -client-cert-auth' and etcd will perform the same.
    --cert-file ''
        path to the client server TLS cert file.
    --key-file ''
        path to the client server TLS key file.
    --client-cert-auth 'false'
        enable client cert authentication.
    --client-crl-file ''
        path to the client certificate revocation list file.
    --trusted-ca-file ''
        path to the client server TLS trusted CA cert file.
    --auto-tls 'false'
        client TLS using generated certificates.
    --peer-ca-file '' [DEPRECATED]
        path to the peer server TLS CA file. '-peer-ca-file ca.crt' could be replaced by '-peer-trusted-ca-file ca.crt -peer-client-cert-auth' and etcd will perform the same.
    --peer-cert-file ''
        path to the peer server TLS cert file.
    --peer-key-file ''
        path to the peer server TLS key file.
    --peer-client-cert-auth 'false'
        enable peer client cert authentication.
    --peer-trusted-ca-file ''
        path to the peer server TLS trusted CA file.
    --peer-auto-tls 'false'
        peer TLS using self-generated certificates if --peer-key-file and --peer-cert-file are not provided.
    --peer-crl-file ''
        path to the peer certificate revocation list file.
    
logging flags

    --debug 'false'
        enable debug-level logging for etcd.
    --log-package-levels ''
        specify a particular log level for each etcd package (eg: 'etcdmain=CRITICAL,etcdserver=DEBUG').
    --log-output 'default'
        specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.

unsafe flags:

Please be CAUTIOUS when using unsafe flags because it will break the guarantees
given by the consensus protocol.

    --force-new-cluster 'false'
        force to create a new one-member cluster.


profiling flags:
    --enable-pprof 'false'
        Enable runtime profiling data via HTTP server. Address is at client URL + "/debug/pprof/"
    --metrics 'basic'
        Set level of detail for exported metrics, specify 'extensive' to include histogram metrics.
    --listen-metrics-urls ''
        List of URLs to listen on for metrics.


auth flags:
    --auth-token 'simple'
        Specify a v3 authentication token type and its options ('simple' or 'jwt').
        
experimental flags:
    --experimental-initial-corrupt-check 'false'
        enable to check data corruption before serving any client/peer traffic.
    --experimental-corrupt-check-time '0s'
        duration of time between cluster corruption check passes.
    --experimental-enable-v2v3 ''
        serve v2 requests through the v3 backend under a given prefix.

```









