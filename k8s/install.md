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
