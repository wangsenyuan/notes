* 1. prepare enviorment
  * 1.1 firewall settings
  ```
    firewall-cmd --add-port=6443/tcp --permanent
    firewall-cmd --add-port=2379-2380/tcp --permanent
    firewall-cmd --add-port=10250-10255/tcp --permanent
    firewall-cmd --add-port=30000-32767/tcp --permanent
    firewall-cmd --reload
  ```
  * stop swap space
  ```
    swapoff -a
  ```

* 2. docker
  * 2.1 install docker
  ```
    yum upgrade
    yum install -y docker
  ```
  * 2.2 config docker
  ```
  mkdir -p /etc/docker
  vi /etc/docker/daemon.json
  {
    "registry-mirrors":["https://s0j60603.mirror.aliyuncs.com"]
  }

  vi /etc/sysconfig/docker
  OPTIONS='--selinux-enabled --log-driver=json-file --signature-verification=false’

  systemctl daemon-reload & systemctl restart docker
  ```
  * 2.3 docker login aliyun registry to make docker record the auth
  ```
  docker login --username=160002@maycur registry.cn-hangzhou.aliyuncs.com
  ```

* 3 kubernetes
  * 3.1
  ```
  cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
  ```
  * 3.2 config yum to use aliyun repository
  ```
  vi /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
   name=Kubernetes
   baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=0
   [kubernetes]
   name=Kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
   enabled=1
   gpgcheck=0
  ```
  * 3.3 install
  ```
  yum install -y kubelet kubectl kubeadm
  ``` 
  * 3.4
  ```
  setenforce 0
  ```
  * 3.5
  ```
  cat > /etc/systemd/system/kubelet.service.d/20-pod-infra-image.conf <<EOF
  [Service]
   Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/maycur-k8s/pause-amd64:3.1"
   EOF
  ```
  * 3.6
  ```
  systemctl enable kubelet && sudo systemctl start kubelet
  ```

* 4 ETCD
  * three etcd nodes
```
  etcd0: 192.168.5.12
  etcd1: 192.168.5.13
  etcd2: 192.168.4.240
```

  * 4.1 install etcd on every nodes
```
  yum install -y etcd
```
  * 4.2 config etcd conf file (take etcd0 as example)
```
  vi /etc/etcd/etcd.conf
```
```
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://192.168.5.12:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.5.12:2379,http://127.0.0.1:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="etcd0"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.5.12:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.5.12:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="etcd0=http://192.168.5.12:2380,etcd1=http://192.168.5.13:2380,etcd2=http://192.168.4.240:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
```

  * 4.3 change etcd.service
```
vi /usr/lib/systemd/system/etcd.service
```
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\" --advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" --initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" --initial-cluster=\"${ETCD_INITIAL_CLUSTER}\" --initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\""
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
  * 4.4 start etcd
```
  systemctl deamon-reload & systemctl start etcd & systemctl status etcd
```
  * 4.5 verify
```
  etcdctl member list
```
