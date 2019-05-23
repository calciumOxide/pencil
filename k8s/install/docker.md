#安装docker

## 1.1

## 1.2 二进制安装
 1,[下载地址](https://download.docker.com/linux/static/stable/x86_64/)
 
 2,解压, tar xzvf docker-18.03.1-ce.tgz
 
 3,复制二进制文件到/usr/bin目录下
 ```
 cp docker/* /usr/bin/
 
 ```
 
 5,配置 docker.service文件
 
 vi /usr/lib/systemd/system/docker.service
 ```text
[Unit]
 Description=Docker Application Container Engine
 Documentation=https://docs.docker.com
 After=network-online.target firewalld.service
 Wants=network-online.target
 
 [Service]
 Type=notify
 ExecStart=/usr/bin/dockerd
 ExecReload=/bin/kill -s HUP $MAINPID
 LimitNOFILE=infinity
 LimitNPROC=infinity
 TimeoutStartSec=0
 Delegate=yes
 KillMode=process
 Restart=on-failure
 StartLimitBurst=3
 StartLimitInterval=60s
 
 [Install]
 WantedBy=multi-user.target
 6,启动dockerd服务进程
 
 systemctl daemon-reload
 systemctl start docker.service
```
 4、如果想进行IP更改（可跳过）
```text
vim /etc/docker/daemon.json
 ```
```
{
    "bip": "172.17.0.1/24"
}
```

