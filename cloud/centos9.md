# Disabling the SELinux

``` shell
# Disable SELinux and modify config
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

# View selinux status
sestatus
```

# Disabling the swap partition

``` shell
# Disable swap and modify config
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

# View swap partition
swapon --show
```

# Disabling the unneeded services

``` shell
# View all started services
systemctl list-unit-files --type=service --state=enabled

# Disable the firewalld
systemctl disable --now firewalld
systemctl status firewalld

# Disable the bluetooth
systemctl disable --now bluetooth
systemctl status bluetooth
```

# Configure system restrictions

```shell
cat << EOF | sudo tee -a /etc/security/limits.conf
* soft nofile 655350
* hard nofile 655350
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
EOF
```

# Configure static ip

``` shell
# View available connections:
nmcli connection show

# Setting static ip
nmcli connection modify eno8303 ipv4.method manual ipv4.address 192.168.8.200/24 ipv4.gateway 192.168.8.1 ipv4.dns "127.0.0.1 114.114.114.114" ipv4.dns-search cloud.com

# Enabled connection
nmcli connection up eno8303
```

# Configure hostname

```shell
# Set its own host name
hostnamectl set-hostname k8s-master-200
```

# Configure DNS Server

```shell
# Add virtual domain name cloud.com
vi /etc/named.conf
listen-on port 53 { any; };
#listen-on-v6 port 53 { ::1; };
allow-query     { any; };
forwarders { 211.140.13.188; 223.5.5.5; 223.6.6.6; };
dnssec-validation no;

zone "cloud.com" {
    type master;
    file "/var/named/cloud.com.zone";
	allow-update { none; };
};

# Add cloud.com zone file 
cat << EOF | tee /var/named/cloud.com.zone
\$TTL 86400
@       IN      SOA     ns1.cloud.com. admin.cloud.com. (
                        2023090901      ; Serial
                        3600            ; Refresh
                        1800            ; Retry
                        604800          ; Expire
                        86400 )         ; Minimum TTL

; Name Servers
@       IN      NS      ns1.cloud.com.

; Define A records
ns1     IN      A       192.168.8.200
www     IN      A       192.168.8.200
k8s-vip IN      A       192.168.8.254

; Define CNAME record
mail    IN      CNAME   www.cloud.com.
EOF

# Restart named
systemctl enable named
systemctl restart named

# Test dns
nslookup www.cloud.com
```

# Configure ssh login 

```shell
# Add ssh public key 
cat << EOF | tee -a ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDNCfT0FIH1oBzwTCYrBbe7bfHEIn/aubfld/qjVQS7a89D84bLkMJjBwF4pKvbRr/g5IIzYMNO2Njb3rEQILmkFzTPYd8zH9hQhy4c6sjTsAic86zxwgCHxerlkz5KZIlcskfZOUmyGZXxna4M2CM1PMeXqn/ehIZaCSQrxjVNw+yueNhl3EPYYDwVTHXZKkrdPGLjmrjTkWYVUTDebqx7bp6QnKITcEoDeylKxZL9iClHKGhQwWn1Sk= whz-work@163.com
EOF

# Modify permission
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

# Configure chronyd synchronization time

```shell
# Edit config file, add ntp server and allow ntp client connect
vi /etc/chrony.conf
# pool 2.centos.pool.ntp.org iburst
server ntp1.aliyun.com iburst
server ntp2.aliyun.com iburst
server ntp3.aliyun.com iburst

allow 192.168.0.0/16

# Setting timezone
timedatectl set-timezone Asia/Shanghai

# Restart chronyd service
systemctl restart chronyd
```

# Configure the Alibaba Cloud Yum mirror

```shell
# Backup repo configuration
mv /etc/yum.repos.d/centos.repo /etc/yum.repos.d/centos.repo.bak
mv /etc/yum.repos.d/centos-addons.repo /etc/yum.repos.d/centos-addons.repo.bak

# Add centos.repo
cat << EOF | tee /etc/yum.repos.d/centos.repo
[baseos]
name=CentOS Stream $releasever - BaseOS
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/BaseOS/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

[baseos-debuginfo]
name=CentOS Stream $releasever - BaseOS - Debug
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/BaseOS/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[baseos-source]
name=CentOS Stream $releasever - BaseOS - Source
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/BaseOS/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[appstream]
name=CentOS Stream $releasever - AppStream
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/AppStream/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

[appstream-debuginfo]
name=CentOS Stream $releasever - AppStream - Debug
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/AppStream/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[appstream-source]
name=CentOS Stream $releasever - AppStream - Source
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/AppStream/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[crb]
name=CentOS Stream $releasever - CRB
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/CRB/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[crb-debuginfo]
name=CentOS Stream $releasever - CRB - Debug
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/CRB/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[crb-source]
name=CentOS Stream $releasever - CRB - Source
baseurl=https://mirrors.aliyun.com/centos-stream/$stream/CRB/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0
EOF

# Add centos-addons.repo
cat << EOF | tee /etc/yum.repos.d/centos-addons.repo
[highavailability]
name=CentOS Stream $releasever - HighAvailability
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/HighAvailability/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[highavailability-debuginfo]
name=CentOS Stream $releasever - HighAvailability - Debug
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/HighAvailability/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[highavailability-source]
name=CentOS Stream $releasever - HighAvailability - Source
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/HighAvailability/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[nfv]
name=CentOS Stream $releasever - NFV
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/NFV/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[nfv-debuginfo]
name=CentOS Stream $releasever - NFV - Debug
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/NFV/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[nfv-source]
name=CentOS Stream $releasever - NFV - Source
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/NFV/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[rt]
name=CentOS Stream $releasever - RT
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/RT/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[rt-debuginfo]
name=CentOS Stream $releasever - RT - Debug
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/RT/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[rt-source]
name=CentOS Stream $releasever - RT - Source
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/RT/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[resilientstorage]
name=CentOS Stream $releasever - ResilientStorage
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/ResilientStorage/$basearch/os/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[resilientstorage-debuginfo]
name=CentOS Stream $releasever - ResilientStorage - Debug
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/ResilientStorage/$basearch/debug/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[resilientstorage-source]
name=CentOS Stream $releasever - ResilientStorage - Source
baseurl=http://mirrors.aliyun.com/centos-stream/$stream/ResilientStorage/source/tree/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[extras-common]
name=CentOS Stream $releasever - Extras packages
baseurl=http://mirrors.aliyun.com/centos-stream/SIGs/$stream/extras/$basearch/extras-common/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Extras-SHA512
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

[extras-common-source]
name=CentOS Stream $releasever - Extras packages - Source
baseurl=http://mirrors.aliyun.com/centos-stream/SIGs/$stream/extras/source/extras-common/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Extras-SHA512
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0
EOF

# Add epel mirror
yum install https://mirrors.aliyun.com/epel/epel-release-latest-9.noarch.rpm
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*
sed -i 's|^#baseurl=https://download.example/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*

# Execute update
yum makecache && yum update
```

# Install Ceph

```shell
# Install cephadm
dnf search release-ceph
dnf install --assumeyes centos-release-ceph-reef
dnf install --assumeyes cephadm

# Running the bootstrap command
cephadm bootstrap --mon-ip 192.168.8.200

# Tnstall the ceph-common package, which contains all of the ceph commands
cephadm add-repo --release reef
cephadm install ceph-common
ceph status

# Add storage to the cluster
wipefs -af /dev/sdb
wipefs -af /dev/sdc
ceph orch apply osd --all-available-devices
            
# Modify config
ceph config set global osd_pool_default_size 1
ceph config set global mon_warn_on_pool_no_redundancy false
ceph config set mon auth_allow_insecure_global_id_reclaim false
ceph config set mgr mgr/cephadm/autotune_memory_target_ratio 0.2
ceph config set osd osd_memory_target_autotune true
```

# Install Docker&Containerd

```shell
# Add Docker stable version of yum software source
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Add docker and containerd
yum -y install docker-ce
systemctl enable --now docker
systemctl enable --now containerd

# Configure docker
cat << EOF | sudo tee /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "5"
    },
    "registry-mirrors": [
		"https://docker.nju.edu.cn"
    ],
    "insecure-registries": ["harbor.cloud.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# Configure containerd
containerd config default > /etc/containerd/config.toml
vi /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri"]  
  sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9"

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
  
[plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.cloud.com".tls]
          insecure_skip_verify = true
        [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.cloud.com".auth]
          username = "admin"
          password = "aw3edqwerT"
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://docker.nju.edu.cn"]

systemctl daemon-reload
systemctl restart containerd

# Add CNI plugin
mkdir -p /opt/cni/bin
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.3.0.tgz
```

# Install HAProxy and Keepalived

```shell
yum -y install keepalived haproxy
systemctl enable --now haproxy 
systemctl enable --now keepalived 

# Configuration HAProxy
cat << EOF | sudo tee /etc/haproxy/haproxy.cfg
global
  log 127.0.0.1 local0 err
  chroot      /var/lib/haproxy
  pidfile     /var/run/haproxy.pid
  maxconn     4096
  user        haproxy
  group       haproxy
  daemon
  stats socket /var/lib/haproxy/stats

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 10s
  timeout http-keep-alive 10s

listen stats
  bind    *:8888
  mode    http
  stats   enable
  stats   hide-version
  stats   uri       /stats
  stats   refresh   30s
  stats   realm     Haproxy\ Statistics
  stats   auth      admin:password
  
frontend k8s-apiserver
  bind *:16443
  mode tcp
  option tcplog
  default_backend k8s-apiserver

backend k8s-apiserver
  mode tcp
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master-200  192.168.8.200:6443  check
  server k8s-master-201  192.168.8.201:6443  check
EOF

# Configuration Keepalived
cat << EOF | sudo tee /etc/keepalived/keepalived.conf
global_defs {
  notification_email {
  }
  router_id LVS_200
  vrrp_skip_check_adv_addr
  vrrp_garp_interval 0
  vrrp_gna_interval 0
}
vrrp_script haproxy-check {
    script "killall -0 haproxy"
    interval 2
    weight 20
}
vrrp_instance haproxy-virtual-ip {
    state MASTER
    priority 101
    interface eno8303
    virtual_router_id 51
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    unicast_src_ip 192.168.8.200
    unicast_peer {
        192.168.8.201
    }
    virtual_ipaddress {
        192.168.8.254/24 dev eno8303
    }
    track_script {
        haproxy-check weight 20
    }
}
EOF

systemctl restart haproxy 
systemctl restart keepalived
```

# Install Kubernetes

```shell
yum -y install ipset ipvsadm socat ebtables conntrack sysstat libseccomp

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
lsmod | grep -e overlay  -e br_netfilter

cat << EOF | sudo tee /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF

modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack
lsmod | grep -e ip_vs -e nf_conntrack

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables   = 1
net.bridge.bridge-nf-call-ip6tables  = 1
net.bridge.bridge-nf-call-arptables  = 1

# improve memory management
vm.swappiness                        = 0
vm.panic_on_oom                      = 0
vm.overcommit_memory                 = 1
vm.dirty_ratio                       = 20
vm.dirty_background_ratio            = 10
vm.max_map_count					 = 262144

# improve network performance
net.core.somaxconn                   = 40960
net.core.netdev_max_backlog          = 10000
net.core.rmem_default                = 262144
net.core.rmem_max                    = 16777216
net.core.wmem_default                = 262144
net.core.wmem_max                    = 16777216

# improve TCP performance
net.ipv4.tcp_keepalive_time          = 600
net.ipv4.tcp_keepalive_intvl         = 30
net.ipv4.tcp_keepalive_probes        = 10
net.ipv4.tcp_max_tw_buckets          = 6000
net.ipv4.tcp_max_syn_backlog         = 8096
net.ipv4.tcp_syncookies              = 1
net.ipv4.tcp_synack_retries          = 2
net.ipv4.neigh.default.gc_stale_time = 120
net.ipv4.conf.all.rp_filter          = 1
net.ipv4.conf.default.rp_filter      = 0
net.ipv4.conf.default.arp_announce   = 2
net.ipv4.conf.lo.arp_announce        = 2
net.ipv4.conf.all.arp_announce       = 2
net.ipv4.ip_local_port_range         = 45001 65000
net.ipv4.ip_local_reserved_ports     = 30000-32767
net.ipv4.ip_forward                  = 1
net.ipv4.ip_nonlocal_bind            = 1
net.ipv4.tcp_wmem                    = 4096 87380 16777216
net.ipv4.tcp_rmem                    = 4096 87380 16777216

net.ipv6.conf.all.disable_ipv6       = 1
net.ipv6.conf.default.disable_ipv6   = 1
net.ipv6.conf.lo.disable_ipv6        = 1
net.ipv6.neigh.default.gc_thresh1    = 8192
net.ipv6.neigh.default.gc_thresh2    = 32768
net.ipv6.neigh.default.gc_thresh3    = 65536

net.netfilter.nf_conntrack_max       = 2310720

fs.nr_open                           = 52706963
fs.file-max                          = 52706963
fs.inotify.max_user_instances        = 8192
fs.inotify.max_user_watches          = 524288

kernel.pid_max                       = 4194303
kernel.softlockup_panic              = 1
kernel.softlockup_all_cpu_backtrace  = 1
EOF

sudo sysctl --system

# Add kubernetes repo
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# Install cri-tools
yum install -y cri-tools
cat << EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
EOF

crictl version

# Install kubenertes
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet

# Add the kubectl command is automatically completed
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
alias k=kubectl
source <(kubectl completion bash | sed s/kubectl/k/g)
echo "alias k=kubectl" >> ~/.bashrc
echo "source <(kubectl completion bash | sed s/kubectl/k/g)" >> ~/.bashrc

# Generate default configuration
cat << EOF | sudo tee kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.8.200
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: k8s-master-200
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: k8s-vip:16443
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.28.2
networking:
  dnsDomain: cluster.local
  podSubnet: 172.90.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
EOF

# list images
kubeadm config images list --config kubeadm-config.yaml

# pull images in all machine
kubeadm config images pull --config kubeadm-config.yaml

# init master in k8s-node-1
kubeadm init --config kubeadm-config.yaml --upload-certs

# To start using your cluster, you need to run the following as a regular user:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Alternatively, if you are the root user, you can run:
export KUBECONFIG=/etc/kubernetes/admin.conf

# Configure NetworkManager
cat << EOF | sudo tee /etc/NetworkManager/conf.d/calico.conf
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
systemctl restart NetworkManager
EOF

# Install calico 
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
curl -o calico-resources.yaml https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
vi calico-resources.yaml 
    - blockSize: 26
      cidr: 172.90.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    nodeAddressAutodetectionV4:
      interface: eno8303

kubectl create -f tigera-operator.yaml
kubectl create -f calico-resources.yaml 

# eanble 
kubectl taint nodes k8s-master-200 node-role.kubernetes.io/control-plane-
```

# Install Metrics-server

- Add --enable-aggregator-routing=true to Kube-Apiserver

  ```shell
  # modify apiserver
  vi /etc/kubernetes/manifests/kube-apiserver.yaml
  --enable-aggregator-routing=true
  ```

- Deploy metrics-server

  ```shell
  curl -o metrics-server.yaml https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.4/components.yaml
  
  vi metrics-server.yaml
  # add tls ignore and modify image
  - --kubelet-insecure-tls
  image: registry.aliyuncs.com/google_containers/metrics-server:v0.6.4
  
  kubectl apply -f metrics-server.yaml
  ```

# Install Helm

```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

# Install Ceph-CSI

```shell
cat << EOF | sudo tee csi-ceph-config.yaml
apiVersion: v1
kind: ConfigMap
data:
  ceph.conf: |
    [global]
    auth_cluster_required = cephx
    auth_service_required = cephx
    auth_client_required = cephx
  # keyring is a required key and its value should be empty
  keyring: |
metadata:
  name: ceph-config
  namespace: ceph-csi
EOF

ceph mon dump
epoch 1
fsid 875542b2-52dc-11ee-81c0-208810bc0b1a
last_changed 2023-09-14T08:56:23.466261+0000
created 2023-09-14T08:56:23.466261+0000
min_mon_release 18 (reef)
election_strategy: 1
0: [v2:192.168.8.200:3300/0,v1:192.168.8.200:6789/0] mon.k8s-master-200
dumped monmap epoch 1

cat << EOF | sudo tee csi-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ceph-csi-config
  namespace: ceph-csi
data:
  config.json: |-
    [
      {
        "clusterID": "875542b2-52dc-11ee-81c0-208810bc0b1a",
        "monitors": [
          "192.168.8.200:6789"
        ]
      }
    ]
EOF

cat << EOF | sudo tee csi-kms-configmap.yaml
apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    {}
metadata:
  name: ceph-csi-encryption-kms-config
  namespace: ceph-csi
EOF

ceph auth get client.admin
[client.admin]
        key = AQC3ygJlAxfKDxAAcFfHqMjs21LCMgJugDGMZA==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
        
cat << EOF | sudo tee csi-rbd-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: ceph-csi
stringData:
  userID: admin
  userKey: AQC3ygJlAxfKDxAAcFfHqMjs21LCMgJugDGMZA==
EOF

cat << EOF | sudo tee csi-cephfs-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-cephfs-secret
  namespace: ceph-csi
stringData:
  userID: admin
  userKey: AQC3ygJlAxfKDxAAcFfHqMjs21LCMgJugDGMZA==
  adminID: admin
  adminKey: AQC3ygJlAxfKDxAAcFfHqMjs21LCMgJugDGMZA==
EOF

kubectl create ns ceph-csi
kubectl apply -f csi-ceph-config.yaml
kubectl apply -f csi-configmap.yaml
kubectl apply -f csi-kms-configmap.yaml
kubectl apply -f csi-rbd-secret.yaml
kubectl apply -f csi-cephfs-secret.yaml

curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/rbd/kubernetes/csi-rbdplugin.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/rbd/kubernetes/csidriver.yaml

# update default namespace
grep -rl "namespace: default" ./
sed -i "s/namespace: default/namespace: ceph-csi/g" $(grep -rl "namespace: default" ./)

# modify image in csi-rbdplugin.yaml and csi-rbdplugin-provisioner.yaml
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.5.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v6.2.2
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v4.3.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.8.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.8.0

kubectl -n ceph-csi apply -f csi-nodeplugin-rbac.yaml
kubectl -n ceph-csi apply -f csi-provisioner-rbac.yaml
kubectl -n ceph-csi apply -f csi-rbdplugin.yaml
kubectl -n ceph-csi apply -f csi-rbdplugin-provisioner.yaml
kubectl apply -f csidriver.yaml

curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/cephfs/kubernetes/csi-nodeplugin-rbac.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/cephfs/kubernetes/csi-provisioner-rbac.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/cephfs/kubernetes/csi-cephfsplugin-provisioner.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/cephfs/kubernetes/csi-cephfsplugin.yaml
curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.9/deploy/cephfs/kubernetes/csidriver.yaml

# modify image in csi-cephfsplugin.yaml and csi-cephfsplugin-provisioner.yaml
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.5.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v6.2.2
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v4.3.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.8.0
registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.8.0

kubectl -n ceph-csi apply -f csi-nodeplugin-rbac.yaml
kubectl -n ceph-csi apply -f csi-provisioner-rbac.yaml
kubectl -n ceph-csi apply -f csi-cephfsplugin.yaml
kubectl -n ceph-csi apply -f csi-cephfsplugin-provisioner.yaml
kubectl apply -f csidriver.yaml

cat << EOF | sudo tee csi-rbd-ssd-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-ssd-sc
   namespace: ceph-csi
   annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: 875542b2-52dc-11ee-81c0-208810bc0b1a
   pool: rbd.k8s.ssd
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
   csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
   csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
   csi.storage.k8s.io/fstype: xfs
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
 - discard
EOF

cat << EOF | sudo tee csi-rbd-hdd-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-hdd-sc
   namespace: ceph-csi
   annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: 875542b2-52dc-11ee-81c0-208810bc0b1a
   pool: rbd.k8s.hdd
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
   csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
   csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
   csi.storage.k8s.io/fstype: xfs
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
 - discard
EOF

cat << EOF | sudo tee csi-cephfs-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-cephfs-sc
   namespace: ceph-csi
provisioner: cephfs.csi.ceph.com
parameters:
   clusterID: 875542b2-52dc-11ee-81c0-208810bc0b1a
   fsName: k8s
   pool: cephfs.k8s.data
   mounter: kernel
   csi.storage.k8s.io/provisioner-secret-name: csi-cephfs-secret
   csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
   csi.storage.k8s.io/controller-expand-secret-name: csi-cephfs-secret
   csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
   csi.storage.k8s.io/node-stage-secret-name: csi-cephfs-secret
   csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
reclaimPolicy: Delete
allowVolumeExpansion: true
EOF

kubectl -n ceph-csi apply -f csi-rbd-ssd-sc.yaml
kubectl -n ceph-csi apply -f csi-rbd-hdd-sc.yaml
kubectl -n ceph-csi apply -f csi-cephfs-sc.yaml
kubectl get sc
```

# Install Kubesphere

```shell
curl -o kubesphere-installer.yaml https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/kubesphere-installer.yaml
curl -o kubesphere-configuration.yaml https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/cluster-configuration.yaml

vim kubesphere-configuration.yaml
  etcd:
    monitoring: true  
    endpointIps: "192.168.60.11,192.168.60.12,192.168.60.13"
  common:
    redis:
      enabled: true
      enableHA: true
    openldap:
      enabled: true
  alerting:       
    enabled: true
  auditing:
    enabled: true 
  devops:
    enabled: true 
  events:
    enabled: true  
  logging: 
    enabled: true
  network:
    networkpolicy:
      enabled: true 
    ippool:
      type: calico
    topology:
      type: weave-scope 
  servicemesh: 
    enabled: true

kubectl -n kubesphere-monitoring-system create secret generic kube-etcd-client-certs  \
--from-file=etcd-client-ca.crt=/etc/kubernetes/pki/etcd/ca.crt  \
--from-file=etcd-client.crt=/etc/kubernetes/pki/etcd/healthcheck-client.crt  \
--from-file=etcd-client.key=/etc/kubernetes/pki/etcd/healthcheck-client.key

kubectl apply -f kubesphere-installer.yaml
kubectl apply -f kubesphere-configuration.yaml
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f

helm install ingress-nginx https://github.com/kubernetes/ingress-nginx/releases/download/helm-chart-4.7.2/ingress-nginx-4.7.2.tgz --set controller.admissionWebhooks.patch.image.registry="k8s.mirror.nju.edu.cn",controller.image.registry="k8s.mirror.nju.edu.cn",controller.opentelemetry.image="k8s.mirror.nju.edu.cn/ingress-nginx/opentelemetry:v20230721-3e2062ee5",defaultBackend.image.registry="k8s.mirror.nju.edu.cn" -n ingress-nginx --create-namespace

```

