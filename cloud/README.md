# Ceph, Harbor, K8s, Kubesphere Installation

---

## Prerequisite

|  Hostname  |    IP Address    |    Virtual IP     | Roles                                                        |
| :--------: | :--------------: | :---------------: | ------------------------------------------------------------ |
| k8s-node-1 | 192.168.60.11/32 | 192.168.60.254/32 | k8s-master, ceph-osd, ceph-mon, ceph-mgr, ceph-rgw, HAProxy, Keepalived |
| k8s-node-2 | 192.168.60.12/32 | 192.168.60.254/32 | k8s-master, ceph-osd, ceph-mon, ceph-mgr, ceph-rgw, HAProxy, Keepalived |
| k8s-node-3 | 192.168.60.13/32 | 192.168.60.254/32 | k8s-master, ceph-osd, ceph-mon, ceph-mgr, ceph-rgw, HAProxy, Keepalived |
| k8s-node-4 | 192.168.60.14/32 |                   | k8s-worker, ceph-osd, ceph-mon, ceph-mds                     |
| k8s-node-5 | 192.168.60.15/32 |                   | k8s-worker, ceph-osd, ceph-mon, ceph-mds                     |
| k8s-node-6 | 192.168.60.16/32 |                   | k8s-worker, ceph-osd, ceph-mon, harbor                       |

### Minimum hardware requirements

* 8GB or more RAM per machine
* 4 Core or more CPUs per machine
* A separate data disk per machine
* All the machine networks can communicate with each other

### Prepare the Environment

* Make sure each machine has a unique hostname, MAC address  and product_uuid 

  ```shell
  # Verify the MAC address and product_uuid are unique for every node
  sudo cat /sys/class/net/eth0/address 
  sudo cat /sys/class/dmi/id/product_uuid
  
  # Set its own host name for every node
  hostnamectl set-hostname xxxx
  
  # Configure IP mapping for every node
  cat << EOF | sudo tee -a /etc/hosts
  192.168.60.254   k8s-vip
  192.168.60.11    k8s-node-1
  192.168.60.12    k8s-node-2
  192.168.60.13    k8s-node-3
  192.168.60.14    k8s-node-4
  192.168.60.15    k8s-node-5
  192.168.60.16    k8s-node-6
  192.168.60.16    harbor.cloud.com
  EOF
  ```

* Distribute the public key

  ```shell
  # Update yum repository for every node
  curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
  curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
  yum install -y expect
  
  # Distribute the public key on ks8-node-1 
  ssh-keygen -t rsa -P "" -f /root/.ssh/id_rsa  # If there is this, skip this step
  for i in {1..6};do
  expect -c "
  spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.60.1$i
      expect {
              "yes/no" {send "yes\\r"; exp_continue}
              "password" {send "123456\\r"; exp_continue}
              "Password" {send "123456\\r";}
      } "
  done 
  ```

* Kernel upgrade for every node

  ```shell
  # Install yum repository for 	every node
  rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
  rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-6.el7.elrepo.noarch.rpm
  
  # Check version
  yum --enablerepo="elrepo-kernel" list --showduplicates | sort -r | grep kernel-lt.x86_64
  
  # Install kernel
  yum --enablerepo="elrepo-kernel" install -y kernel-lt-devel kernel-lt 
  
  # Check the existing kernel boot order
  awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
  
  # Setting the Boot Number
  grub2-set-default 0
  
  # Reboot
  reboot
  
  # Check kernel version
  uname -r
  ```

* Close the SELinux and Frewalld for every node

  ```shell
  # Close the SELinux
  setenforce 0
  sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
  
  # Close the firewalld
  systemctl disable --now firewalld
  ```

* Disabling the swap partition for every node

  ```shell
  # Disabling the swap partition
  swapoff -a
  sed -ri 's/.*swap.*/#&/' /etc/fstab
  ```

* Modifying system restrictions for every node

  ```shell
  # Modify system limits
  cat << EOF | sudo tee -a /etc/security/limits.conf
  * soft nofile 655350
  * hard nofile 655350
  * soft nproc 655350
  * hard nproc 655350
  * soft memlock unlimited
  * hard memlock unlimited
  EOF
  ```

* Allow iptables to check bridge traffic

  ```shell
  cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
  overlay
  br_netfilter
  EOF
  
  sudo modprobe overlay
  sudo modprobe br_netfilter
  
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
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
  
  net.bridge.bridge-nf-call-ip6tables  = 1
  net.bridge.bridge-nf-call-iptables   = 1
  net.bridge.bridge-nf-call-arptables  = 1
  
  net.netfilter.nf_conntrack_max       = 2310720
  
  net.core.netdev_max_backlog          = 16384
  net.core.rmem_max                    = 16777216
  net.core.wmem_max                    = 16777216
  net.core.somaxconn                   = 32768
  
  fs.nr_open                           = 52706963
  fs.file-max                          = 52706963
  fs.inotify.max_user_instances        = 8192
  fs.inotify.max_user_watches          = 524288
  
  vm.swappiness                        = 0
  vm.panic_on_oom                      = 0
  vm.overcommit_memory                 = 1
  vm.max_map_count                     = 262144
  
  kernel.pid_max                       = 4194303
  kernel.softlockup_panic              = 1
  kernel.softlockup_all_cpu_backtrace  = 1
  EOF
  
  sudo sysctl --system
  ```

* Setting Time Synchronization

  ```shell
  # Setting timezone
  timedatectl set-timezone Asia/Shanghai
  
  # Apply crontab
  crontab -e
  */5 * * * * /usr/sbin/ntpdate time2.aliyun.com &> /dev/null
  ```

## Install Ceph 

* If the installation fails, you can remove it with the following command:

  ```shell
  # Execute in the deployed directory
  ceph-deploy purge k8s-node-1 k8s-node-2 k8s-node-3 k8s-node-4 k8s-node-5 k8s-node-6
  ceph-deploy purgedata k8s-node-1 k8s-node-2 k8s-node-3 k8s-node-4 k8s-node-5 k8s-node-6
  ceph-deploy forgetkeys
  rm -rf /etc/ceph/*
  rm -rf /var/lib/ceph/*
  rm -rf /var/log/ceph/*
  rm -rf /var/run/ceph/*
  
  # remove osd
  lsblk
  dmsetup remove --force ceph--fed0a738--e67f--4f1d--8a9d--055c2832bd88-osd--block--e18635b3--b703--4278--9ce5--b09585c812cc
  ```

* Add ceph yum repository for every node

  ```shell
  cat << EOF | sudo tee /etc/yum.repos.d/ceph.repo
  [ceph]
  name=ceph
  baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/x86_64/
  enabled=1
  gpgcheck=0
  priority=1
  
  [ceph-noarch]
  name=cephnoarch
  baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/
  enabled=1
  gpgcheck=0
  priority=1
  
  [ceph-source]
  name=Ceph source packages
  baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/SRPMS
  enabled=1
  gpgcheck=0
  priority=1
  EOF
  ```

* Install depends for every node

  ```shell
  # Install depends on all host
  yum install -y python-setuptools python-pecan
  ```

* Install ceph-deploy and deploy

  ```shell
  # Install ceph-deploy on ks8-node-1
  yum install -y ceph-deploy
  ceph-deploy --version
  
  # Create deploy and monitor
  mkdir ~/ceph-deploy && cd ~/ceph-deploy
  ceph-deploy new k8s-node-1 k8s-node-2 k8s-node-3 k8s-node-4 k8s-node-5 k8s-node-6
  
  # Configuration ceph
  cat << EOF | sudo tee -a ceph.conf
  osd pool default size = 2
  osd pool default min size = 1
  osd pool default pg num = 64
  osd pool default pgp num = 64
  mon max pg per osd = 500
  
  [mon]
  mon clock drift allowed = 2
  mon clock drift warn backoff = 30
  mon osd min down reporters = 10
  mon osd down out interval = 600
  
  [osd]
  osd mkfs type = xfs
  osd mkfs options xfs = -f -i size=2048
  filestore xattr use omap = true
  filestore min sync interval = 10
  filestore max sync interval = 15
  filestore queue max ops = 25000
  filestore queue max bytes = 1048576000
  filestore queue committing max ops = 50000
  filestore queue committing max bytes = 10485760000
  filestore split multiple = 8
  filestore merge threshold = 40
  filestore fd cache size = 1024
  journal max write bytes = 1073714824
  journal max write entries = 10000
  journal queue max ops = 50000
  journal queue max bytes = 10485760000
  osd max write size = 512
  osd client message size cap = 2147483648
  osd deep scrub stride = 131072
  osd op threads = 16
  osd disk threads = 4
  osd map cache size = 1024
  osd map cache bl size = 128
  osd mount options xfs = "rw,noexec,nodev,noatime,nodiratime,nobarrier"
  osd recovery op priority = 2
  osd recovery max active = 10
  osd max backfills = 4
  osd min pg log entries = 30000
  osd max pg log entries = 100000
  osd mon heartbeat interval = 40
  ms dispatch throttle bytes = 1048576000
  objecter inflight ops = 819200
  osd op log threshold = 50
  osd crush chooseleaf type = 0
  
  [client]
  rbd cache = true
  rbd cache size = 335544320
  rbd cache max dirty = 134217728
  rbd cache max dirty age = 30
  rbd cache writethrough until flush = false 
  rbd cache max dirty object = 2   
  rbd cache target dirty = 235544320
  rbd default features = 1
  
  EOF
  
  # Install ceph
  ceph-deploy install --no-adjust-repos k8s-node-1 k8s-node-2 k8s-node-3 k8s-node-4 k8s-node-5 k8s-node-6
  ceph --version
  
  # Create moniter initial
  ceph-deploy mon create-initial
  
  # Apply config to every node
  ceph-deploy --overwrite-conf admin k8s-node-1 k8s-node-2 k8s-node-3 k8s-node-4 k8s-node-5 k8s-node-6
  
  # Create manager instance 
  ceph-deploy mgr create k8s-node-1 k8s-node-2 k8s-node-3
  ceph config set global mon_warn_on_pool_no_redundancy false
  ceph config set mon auth_allow_insecure_global_id_reclaim false
  ceph -s
  ```

* Install Dashboard

  ```shell
  # Install dashboard on k8s-node-1，k8s-node-2，k8s-node-3
  yum install -y ceph-mgr-dashboard 
  ceph mgr module enable dashboard
  
  # Configuration mgr
  ceph config set mgr mgr/dashboard/server_addr 192.168.60.254
  ceph config set mgr mgr/dashboard/server_port 84
  ceph config set mgr mgr/dashboard/ssl_server_port 8484
  ceph config set mgr mgr/dashboard/ssl true
  
  # Restart mgr
  ceph mgr module disable dashboard
  ceph mgr module enable dashboard
  ceph mgr services
  
  # Add dashboard account
  ceph dashboard create-self-signed-cert
  echo "Password" > ceph_password
  ceph dashboard ac-user-create Admin -i ceph_password administrator
  ```

* Create OSD

  ```shell
  # Checking Disk Identifiers for every node (lsblk)
  ceph-deploy disk list k8s-node-1
  ceph-deploy disk list k8s-node-2
  ceph-deploy disk list k8s-node-3
  ceph-deploy disk list k8s-node-4
  ceph-deploy disk list k8s-node-5
  ceph-deploy disk list k8s-node-6
  
  # Erase hard drive data. Skip this step if there is no data
  ceph-deploy disk zap k8s-node-1 /dev/vdb
  ceph-deploy disk zap k8s-node-2 /dev/vdb
  ceph-deploy disk zap k8s-node-3 /dev/vdb
  ceph-deploy disk zap k8s-node-4 /dev/vdb
  ceph-deploy disk zap k8s-node-5 /dev/vdb
  ceph-deploy disk zap k8s-node-6 /dev/vdb
  
  # Create osd instance
  ceph-deploy osd create k8s-node-1 --data /dev/vdb
  ceph-deploy osd create k8s-node-2 --data /dev/vdb
  ceph-deploy osd create k8s-node-3 --data /dev/vdb
  ceph-deploy osd create k8s-node-4 --data /dev/vdb
  ceph-deploy osd create k8s-node-5 --data /dev/vdb
  ceph-deploy osd create k8s-node-6 --data /dev/vdb
  
  # Check
  ceph osd tree
  ```

### RBD

* Create kubernetes pool

  ```shell
  ceph osd pool create kubernetes 64 64
  ceph osd pool application enable kubernetes rbd
  ceph osd pool set kubernetes size 2    # Set the number of replices as required
  ceph osd pool ls
  rbd pool init kubernetes
  ```

* Create kubernetes account

  ```shell
  ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kubernetes' mgr 'profile rbd pool=kubernetes'
  ceph auth get client.kubernetes
  ```

* Test RBD

  ```shell
  rbd create --size 1024 kubernetes/docker_image
  rbd ls kubernetes
  rbd info kubernetes/docker_image
  rbd resize --size 2048 kubernetes/docker_image
  rbd info kubernetes/docker_image
  rbd resize --size 1024 kubernetes/docker_image --allow-shrink
  rbd info kubernetes/docker_image
  rbd rm kubernetes/docker_image
  ```

### CephFS 

* Create mds instance

  ```shell
  ceph-deploy mds create k8s-node-4 k8s-node-5
  ceph mds stat
  ```

* Create cephfs pool

  ```shell
  ceph osd pool create cehpfs-metadata 64 64
  ceph osd pool set cehpfs-metadata size 2   # Set the number of replices as required
  ceph osd pool create cehpfs-data 64 64	   
  ceph osd pool set cehpfs-data size 2       # Set the number of replices as required
  ceph fs new cephfs cehpfs-metadata cehpfs-data
  ceph fs status cephfs
  ```

* Create account

  ```shell
  # Create cephfs account
  ceph auth get-or-create client.cephfs mon 'allow r' mds 'allow rw' osd 'allow rw pool=cephfs-data, allow rw pool=cephfs-metadata'
  ceph auth get client.cephfs
  ```

* Test CephFS

  ```shell
  # Mount to disk on k8s-node-6
  mkdir /mnt/cephfs
  mount -t ceph k8s-node-1:6789:/ /mnt/cephfs -o name=cephfs,secret=AQCV0fNiIXRVFBAAiKcC9zkmpcMoib5OyTVZeg==
  stat -f /mnt/cephfs/
  ```

### RadosGW

* Install rgw

  ```shell
  ceph-deploy install --no-adjust-repos --rgw k8s-node-1 k8s-node-2 k8s-node-3
  ```


* Create rgw instance

  ```shell
  ceph-deploy rgw create k8s-node-1 k8s-node-2 k8s-node-3
  ceph osd pool set .rgw.root size 2    # Set the number of replices as required
  ceph osd pool set default.rgw.meta size 2    # Set the number of replices as required
  ceph osd pool set default.rgw.control size 2     # Set the number of replices as required
  rados lspools
  ```

* Create account 

  ```shell
  # Record the access_key and secret_key
  radosgw-admin user create --uid="radosgw" --display-name="radosgw" --system
  radosgw-admin subuser create --uid=radosgw --subuser=radosgw:swift --access=full
  radosgw-admin user info --uid=radosgw
  ```

* Test RadosGW

  ```shell
  echo KT3MJZ3M4MBM43AVBRLC > ceph_rgw_key
  echo vjdT3pj1YaVizOu1VAkOLeMDh7E8zW657DYq96k0 > ceph_rgw_secret
  
  ceph dashboard set-rgw-api-access-key -i ceph_rgw_key
  ceph dashboard set-rgw-api-secret-key -i ceph_rgw_secret
  ceph dashboard set-rgw-api-ssl-verify False
  ceph dashboard set-rgw-api-host k8s-node-1
  ceph dashboard set-rgw-api-port 7480
  ceph dashboard set-rgw-api-scheme http
  ceph dashboard set-rgw-api-user-id radosgw
  ```

## Install Docker

* Remove it if an older version is already installed

  ```shell
  sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-selinux \
                    docker-engine-selinux \
                    docker-engine
  ```

* First, to facilitate the addition of software sources and support for the Device Apper storage type, install the
  following software packages

  ```shell
  sudo yum update
  sudo yum install -y yum-utils device-mapper-persistent-data lvm2
  ```

* Add Docker stable version of yum software source

  ```shell
  sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```

* Then update the yum software source cache and install Docker

  ```shell
  sudo yum update
  sudo yum -y install docker-ce
  ```

* Finally, confirm the normal startup of Docker service and set it to automatic startup

  ```shell
  sudo systemctl start docker
  sudo systemctl enable docker
  ```

* Check the installed Docker version

  ```shell
  docker version
  ```

### Configuring docker registry mirror

* Add the following configuration in the docker configuration file `/etc/docker/daemon.json`, If the file does not
  exist, create a new one.

  ```shell
  cat << EOF | sudo tee /etc/docker/daemon.json
  {
      "storage-driver": "overlay2",
      "storage-opts": ["overlay2.override_kernel_check=true"],
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
          "max-size": "100m",
          "max-file": "5"
      },
      "registry-mirrors": [
          "https://ccr.ccs.tencentyun.com",
          "https://ustc-edu-cn.mirror.aliyuncs.com",
          "https://e9yneuy4.mirror.aliyuncs.com"
      ],
      "insecure-registries": ["harbor.cloud.com"]
  }
  EOF
  ```
  
* Restart the docker

  ```shell
  sudo systemctl daemon-reload
  sudo systemctl restart docker
  ```

* Check whether the mirror takes effect

  ```shell
  docker info
  ...
  Registry: 
      https://ccr.ccs.tencentyun.com
      https://ustc-edu-cn.mirror.aliyuncs.com
  ```

### Install the docker compose

* Execute the following command, you can customize the version you need by changing the version in the URL.

  ```shell
  curl -L https://get.daocloud.io/docker/compose/releases/download/v2.7.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose
  ```

* Check whether the installation is successful

  ```shell
  docker-compose version
  ```

### Install CRI-Dockerd

 * Install  cri-dockerd

   ```shell
   rpm -ivh https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.5/cri-dockerd-0.2.5-3.el7.x86_64.rpm
   sed -i 's#fd://#& --network-plugin=cni --pod-infra-container-image=harbor.cloud.com/k8s.gcr.io/pause:3.7#g' /usr/lib/systemd/system/cri-docker.service
   
   cat << EOF | sudo tee /etc/crictl.yaml
   runtime-endpoint: unix:///var/run/cri-dockerd.sock
   image-endpoint: unix:///var/run/cri-dockerd.sock
   timeout: 10
   EOF
   
   systemctl daemon-reload
   systemctl enable --now cri-docker
   systemctl status cri-docker
   ```

## Install Harbor

* Download Harbor on k8s-node-6

  ```shell
  # The file is relatively large, you can also upload yourself
  curl -O https://github.com/goharbor/harbor/releases/download/v2.4.3/harbor-offline-installer-v2.4.3.tgz
  tar -zxvf harbor-offline-installer-v2.4.3.tgz -C /usr/local/
  ```

* Install Harbor

  ```shell
  # Copy config
  cd /usr/local/harbor/
  cp harbor.yml.tmpl harbor.yml
  
  # Modify config
  vi harbor.yml
  hostname: harbor.cloud.com
  #https:
  #  # https port for harbor, default is 443
  #  port: 443
  #  # The path of cert and key files for nginx
  #  certificate: /your/certificate/path
  #  private_key: /your/private/key/path
  data_volume: /mnt/cephfs
  
  # Install
  ./install.sh
  ```

* Add Registries in Harbor

  | Provider   | Name       | Endpoint URL          | Access ID |
  | ---------- | ---------- | --------------------- | --------- |
  | Docker Hub | docker.io  | default               | empty     |
  | Quay       | quay.io    | https://quay.io       | empty     |
  | Quay       | gcr.io     | https://gcr.lank8s.cn | empty     |
  | Quay       | k8s.gcr.io | https://lank8s.cn     | empty     |

* Add Project in Harbor

  | Project Name | Access Level | Proxy Cache | Registry   |
  | ------------ | ------------ | ----------- | ---------- |
  | docker.io    | Public       | Yes         | docker.io  |
  | quay.io      | Public       | Yes         | quay.io    |
  | gcr.io       | Public       | Yes         | gcr.io     |
  | k8s.gcr.io   | Public       | Yes         | k8s.gcr.io |

* Test docker pull from proxy

  ```shell
  docker pull harbor.cloud.com/docker.io/library/nginx
  docker pull harbor.cloud.com/k8s.gcr.io/pause:3.6
  docker pull harbor.cloud.com/gcr.io/k8s-staging-sig-storage/csi-provisioner:canary
  docker pull harbor.cloud.com/quay.io/cephcsi/cephcsi:v3.6-canary
  ```

## Install HAProxy and Keepalived

* Install HAProxy and Keepalived on Master nodes

  ```shell
  yum -y install keepalived haproxy
  
  # Configuration HAProxy for k8s-node-1,  k8s-node-2, k8s-node-3
  cat << EOF | sudo tee /etc/haproxy/haproxy.cfg
  global
    log 127.0.0.1 local0 err
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
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
    timeout http-request 15s
    timeout http-keep-alive 15s
  
  listen stats
    bind    *:10000
    mode    http
    stats   enable
    stats   hide-version
    stats   uri       /stats
    stats   refresh   30s
    stats   realm     Haproxy\ Statistics
    stats   auth      Admin:Password
    
  frontend k8s-master
    bind *:16443
    mode tcp
    option tcplog
    tcp-request inspect-delay 5s
    default_backend k8s-master
  
  backend k8s-master
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server k8s-node-1  192.168.60.11:6443  check
    server k8s-node-2  192.168.60.12:6443  check
    server k8s-node-3  192.168.60.13:6443  check
  EOF
  
  # Configuration HAProxy for k8s-node-1
  cat << EOF | sudo tee /etc/keepalived/keepalived.conf
  global_defs {
    notification_email {
    }
    router_id LVS_DEVEL
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
      interface eth0
      virtual_router_id 51
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass K8SHA_KA_AUTH
      }
      unicast_src_ip 192.168.60.11
      unicast_peer {
          192.168.60.12
          192.168.60.13
      }
      virtual_ipaddress {
          192.168.60.254
      }
      track_script {
          haproxy-check weight 20
      }
  }
  EOF
  
  # Configuration KeepAlived for k8s-node-2
  cat << EOF | sudo tee /etc/keepalived/keepalived.conf
  global_defs {
    notification_email {
    }
    router_id LVS_DEVEL
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
      state BACKUP
      priority 100
      interface eth0
      virtual_router_id 51
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass K8SHA_KA_AUTH
      }
      unicast_src_ip 192.168.60.12
      unicast_peer {
          192.168.60.11
          192.168.60.13
      }
      virtual_ipaddress {
          192.168.60.254
      }
      track_script {
          haproxy-check weight 20
      }
  }
  EOF
  
  # Configuration KeepAlived for k8s-node-3
  cat << EOF | sudo tee /etc/keepalived/keepalived.conf
  global_defs {
    notification_email {
    }
    router_id LVS_DEVEL
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
      state BACKUP
      priority 100
      interface ens33
      virtual_router_id 51
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass K8SHA_KA_AUTH
      }
      unicast_src_ip 192.168.60.13
      unicast_peer {
          192.168.60.11
          192.168.60.12
      }
      virtual_ipaddress {
          192.168.60.254
      }
      track_script {
          haproxy-check weight 20
      }
  }
  EOF
  
  systemctl daemon-reload
  systemctl enable --now haproxy
  systemctl enable --now keepalived
  ```

## Install Kubernetes

* If the installation fails, you can remove it with the following command:

  ```shell
  kubeadm reset -f --cri-socket=unix:///var/run/cri-dockerd.sock
  rm -rf ~/.kube/
  rm -rf /etc/kubernetes/
  rm -rf /etc/systemd/system/kubelet.service.d
  rm -rf /etc/systemd/system/kubelet.service
  rm -rf /usr/bin/kube*
  rm -rf /etc/cni
  rm -rf /opt/cni
  rm -rf /var/lib/etcd
  rm -rf /var/etcd
  yum -y remove kubeadm* kubectl* kubelet* docker*
  ```

### Install kubernetes dependencies

* Install ipset，ipvs and socat etc

  ```shell
  yum -y install ipset ipvsadm socat ebtables conntrack sysstat libseccomp
  
  cat << EOF | sudo tee /etc/sysconfig/modules/ipvs.modules
  #!/bin/bash
  modprobe -- ip_vs
  modprobe -- ip_vs_rr
  modprobe -- ip_vs_wrr
  modprobe -- ip_vs_sh
  modprobe -- nf_conntrack
  modprobe -- ip_tables
  modprobe -- ip_set
  modprobe -- xt_set
  modprobe -- ipt_set
  modprobe -- ipt_rpfilter
  modprobe -- ipt_REJECT
  modprobe -- ipip
  EOF
  
  chmod +x /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack
  
  cat << EOF | sudo tee -a /etc/modules-load.d/k8s.conf
  ip6_udp_tunnel
  ip_set
  ip_set_hash_ip
  ip_set_hash_net
  iptable_filter
  iptable_nat
  iptable_mangle
  iptable_raw
  nf_conntrack_netlink
  nf_conntrack
  nf_conntrack_ipv4
  nf_defrag_ipv4
  nf_nat
  nf_nat_ipv4
  nf_nat_masquerade_ipv4
  nfnetlink
  udp_tunnel
  veth
  vxlan
  x_tables
  xt_addrtype
  xt_conntrack
  xt_comment
  xt_mark
  xt_multiport
  xt_nat
  xt_recent
  xt_set
  xt_statistic
  xt_tcpudp
  EOF
  
  cat << EOF | sudo tee /etc/modules-load.d/ipvs.conf
  ip_vs
  ip_vs_rr
  ip_vs_wrr
  ip_vs_sh
  nf_conntrack_ipv4
  EOF
  
  sudo sysctl --system
  ```

### Install Kubelet, Kubeadm and Kubectl

* Add kubernetes YUM source 

  ```shell
  cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
  enabled=1
  gpgcheck=0
  repo_gpgcheck=0
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
  EOF
  ```

* Install kubelet、kubeadm、kubectl

  ```shell
  # Check version
  yum list kubeadm.x86_64 --showduplicates | sort -r
  
  yum install -y kubelet-1.24.3-0 kubeadm-1.24.3-0  kubectl-1.24.3-0
  
  systemctl enable --now kubelet
  ```

* Configuration kubelet 

  ```shell
  cat << EOF | sudo tee /etc/sysconfig/kubelet
  KUBELET_EXTRA_ARGS="--cgroup-driver=systemd --container-runtime=remote --container-runtime-endpoint=unix:///var/run/cri-dockerd.sock"
  EOF
  
  systemctl restart kubelet
  ```

* Add the kubectl command is automatically completed 

  ```shell
  yum install -y bash-completion
  source /usr/share/bash-completion/bash_completion
  alias k=kubectl
  source <(kubectl completion bash | sed s/kubectl/k/g)
  echo "alias k=kubectl" >> ~/.bashrc
  echo "source <(kubectl completion bash | sed s/kubectl/k/g)" >> ~/.bashrc
  ```

### Initialize the master

* Generate the initial configuration

  ```shell
  # Generate default configuration
  kubeadm config print init-defaults > kubeadm-config.yaml
  
  # Add it to the k8s-node-1 machine
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
    advertiseAddress: 192.168.60.11
    bindPort: 6443
  nodeRegistration:
    criSocket: unix:///var/run/cri-dockerd.sock
    imagePullPolicy: IfNotPresent
    name: k8s-node-1
    taints: null
  ---
  apiServer:
    certSANs:
    - 192.168.60.11
    - 192.168.60.12
    - 192.168.60.13
    - 192.168.60.254
    - k8s-node-1
    - k8s-node-2
    - k8s-node-3
    - k8s-vip
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
  imageRepository: registry.aliyuncs.com/google_containers
  kind: ClusterConfiguration
  kubernetesVersion: 1.24.3
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
  ```

* Download the image and initialize the master

  ```shell
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
    
  # install calico and modify CALICO_IPV4POOL_CIDR
  curl -O https://docs.projectcalico.org/manifests/calico.yaml
  vi calico
  image: harbor.cloud.com/docker.io/  # modify
  - name: CALICO_IPV4POOL_CIDR
    value: "172.90.0.0/16"
  - name: IP_AUTODETECTION_METHOD
    value: "interface=ens33,eth0"
    
  kubectl apply -f calico.yaml 
  ```
  

### Adding Master to a cluster 

* The master joins the cluster 

  ```shell
  # Execute the following statement, on 46 and 47
  kubeadm join k8s-vip:16443 --token abcdef.0123456789abcdef \
          --discovery-token-ca-cert-hash sha256:9fd4de8b3fb6a03700c1f35db82b8381d430ad3036fb1096094fda0432ce6bee \
          --control-plane --certificate-key 1982044fa551c301ddc33d772b3481d3b100717a8e6fb43d0f1518247a800c11  --cri-socket unix:///var/run/cri-dockerd.sock
  
  # Execute the following statement, on 46 and 47
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  # Whether to make the master schedulable 
  kubectl taint nodes k8s-node-1 node-role.kubernetes.io/control-plane-
  kubectl taint nodes k8s-node-1 node-role.kubernetes.io/master:NoSchedule-
  kubectl taint nodes k8s-node-2 node-role.kubernetes.io/control-plane-
  kubectl taint nodes k8s-node-2 node-role.kubernetes.io/master:NoSchedule-
  kubectl taint nodes k8s-node-3 node-role.kubernetes.io/control-plane-
  kubectl taint nodes k8s-node-3 node-role.kubernetes.io/master:NoSchedule-
  ```

### Adding Worker to a cluster 

* The worker joins the cluster  

  ```shell
  # Execute the following statement, on 48 and 49
  kubeadm join k8s-vip:16443 --token abcdef.0123456789abcdef \
          --discovery-token-ca-cert-hash sha256:9fd4de8b3fb6a03700c1f35db82b8381d430ad3036fb1096094fda0432ce6bee  --cri-socket unix:///var/run/cri-dockerd.sock
  ```

## Install kubernetes Dashboard

* The Dashboard UI is not deployed by default. To deploy it, run the following command:

  ```shell
  curl -o kubernetes-dashboard.yaml https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
  
  vi kubernetes-dashboard.yaml 
  # modify
  image: harbor.cloud.com/docker.io/
  
  kubectl apply -f kubernetes-dashboard.yaml
  ```

## Install metrics-server

* Deploy metrics-server

  ```shell
  curl -o metrics-server.yaml https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml
  
  vi metrics-server.yaml
  # add tls ignore and modify image
  - --kubelet-insecure-tls
  image: registry.aliyuncs.com/google_containers/metrics-server:v0.6.1
  
  kubectl apply -f metrics-server.yaml
  ```

## Install Ceph-CSI

* Add secret to kubernetes

  ```shell
  ceph auth get client.kubernetes
      [client.kubernetes]
              key = AQDU7PFiYVhBDxAA3vx3VdOTcwzyS1j4qBmxFQ==  # userKey
              caps mgr = "profile rbd pool=kubernetes"
              caps mon = "profile rbd"
              caps osd = "profile rbd pool=kubernetes"
      exported keyring for client.kubernetes
  
  cat << EOF | sudo tee csi-rbd-secret.yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: csi-rbd-secret
    namespace: ceph-csi
  stringData:
    userID: kubernetes
    userKey: AQDU7PFiYVhBDxAA3vx3VdOTcwzyS1j4qBmxFQ==
  EOF
  
  kubectl create ns ceph-csi
  kubectl apply -f csi-rbd-secret.yaml
  ```

* Add ConfigMap to kubernates

  ```shell
  ceph mon dump
      epoch 2
      fsid c00c9345-1cfe-4fc9-82ea-c0e33ec4e361  # clusterID
      last_changed 2022-08-08 22:49:34.797878
      created 2022-08-08 22:49:08.329248
      min_mon_release 14 (nautilus)
      0: [v2:192.168.60.11:3300/0,v1:192.168.60.11:6789/0] mon.k8s-node-1
      1: [v2:192.168.60.12:3300/0,v1:192.168.60.12:6789/0] mon.k8s-node-2
      2: [v2:192.168.60.13:3300/0,v1:192.168.60.13:6789/0] mon.k8s-node-3
      3: [v2:192.168.60.14:3300/0,v1:192.168.60.14:6789/0] mon.k8s-node-4
      4: [v2:192.168.60.15:3300/0,v1:192.168.60.15:6789/0] mon.k8s-node-5
      5: [v2:192.168.60.16:3300/0,v1:192.168.60.16:6789/0] mon.k8s-node-6
      dumped monmap epoch 2'
  
  cat << EOF | sudo tee csi-configmap.yaml
  ---
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: ceph-csi-config
    namespace: ceph-csi
  data:
    config.json: |-
      [
        {
          "clusterID": "c00c9345-1cfe-4fc9-82ea-c0e33ec4e361",
          "monitors": [
            "192.168.60.11:6789",
            "192.168.60.12:6789",
            "192.168.60.13:6789",
            "192.168.60.14:6789",
            "192.168.60.15:6789",
            "192.168.60.16:6789"
          ]
        }
      ]
  EOF
  
  cat << EOF | sudo tee ceph-config.yaml
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
  
  # apply 
  kubectl apply -f ceph-config.yaml
  kubectl apply -f csi-configmap.yaml
  kubectl apply -f csi-kms-configmap.yaml
  ```

* Add RBAC and rbdplugin to kubernetes

  ```shell
  curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.6/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml
  curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.6/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml
  curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.6/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml
  curl -O https://raw.githubusercontent.com/ceph/ceph-csi/release-v3.6/deploy/rbd/kubernetes/csi-rbdplugin.yaml
  
  # update default namespace
  grep -rl "namespace: default" ./
  sed -i "s/namespace: default/namespace: ceph-csi/g" $(grep -rl "namespace: default" ./)
  
  # apply
  kubectl -n ceph-csi apply -f csi-nodeplugin-rbac.yaml
  kubectl -n ceph-csi apply -f csi-provisioner-rbac.yaml
  
  # modify image in csi-rbdplugin.yaml and csi-rbdplugin-provisioner.yaml
  harbor.cloud.com/gcr.io/k8s-staging-sig-storage/csi-provisioner:canary
  harbor.cloud.com/k8s.gcr.io/sig-storage/csi-snapshotter:v5.0.1
  harbor.cloud.com/k8s.gcr.io/sig-storage/csi-attacher:v3.4.0
  harbor.cloud.com/k8s.gcr.io/sig-storage/csi-resizer:v1.4.0
  harbor.cloud.com/k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.4.0
  harbor.cloud.com/quay.io/cephcsi/cephcsi:v3.6-canary
  
  # apply 
  kubectl -n ceph-csi apply -f csi-rbdplugin.yaml
  kubectl -n ceph-csi apply -f csi-rbdplugin-provisioner.yaml
  kubectl get pod -A
  ```

* Add ceph storageclass  to kubernetes

  ```shell
  cat << EOF | sudo tee csi-rbd-sc.yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
     name: csi-rbd-sc
     namespace: ceph-csi
     annotations:
      storageclass.kubernetes.io/is-default-class: "true"
  provisioner: rbd.csi.ceph.com
  parameters:
     clusterID: c00c9345-1cfe-4fc9-82ea-c0e33ec4e361
     pool: kubernetes
     imageFeatures: layering
     csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
     csi.storage.k8s.io/provisioner-secret-namespace: ceph-csi
     csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
     csi.storage.k8s.io/controller-expand-secret-namespace: ceph-csi
     csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
     csi.storage.k8s.io/node-stage-secret-namespace: ceph-csi
     csi.storage.k8s.io/fstype: ext4
  reclaimPolicy: Delete
  allowVolumeExpansion: true
  mountOptions:
   - discard
  EOF
  
  kubectl -n ceph-csi apply -f csi-rbd-sc.yaml
  kubectl get sc
  ```

* Test ceph-csi

  ```shell
  cat << EOF | sudo tee csi-rbd-test.yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: raw-block-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    volumeMode: Block
    resources:
      requests:
        storage: 1Gi
    storageClassName: csi-rbd-sc
  EOF
  
  kubectl apply -f csi-rbd-test.yaml
  kubectl get pvc
  kubectl get pv
  rbd ls kubernetes
  rbd info kubernetes/csi-vol-8f99227f-17c2-11ed-ba7b-e62cced8c67d
  ceph df
  kubectl delete -f csi-rbd-test.yaml
  ```

## Install KubeSphere

* Install kubeshpere

  ```shell
  curl -o kubesphere-installer.yaml https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/kubesphere-installer.yaml
  curl -o kubesphere-configuration.yaml https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/cluster-configuration.yaml
  
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
  
  # apply
  kubectl apply -f kubesphere-installer.yaml
  kubectl apply -f kubesphere-configuration.yaml
  kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
  ```

## Install Ingress-nginx

* Download the Ingress-nginx deployment file and install it

  ```shell
  # download
  curl -o ingress-nginx.yaml https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
  
  # add label to k8s-node-1,k8s-node-2,k8s-node-3
  kubectl label node k8s-node-1 ingress-nginx-node=true
  kubectl label node k8s-node-2 ingress-nginx-node=true
  kubectl label node k8s-node-3 ingress-nginx-node=true
  
  # modify ingress-nginx-controller deployment
  vi ingress-nginx.yaml
  
  # modify image
  harbor.cloud.com/k8s.gcr.io/ingress-nginx/controller:v1.3.0
  harbor.cloud.com/k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
  
  # add configuration
  kind: Deployment
  .....
  spec:
    replicas: 3  # add
    template:
  .....
      spec: 
        hostNetwork: true   # add
  	  containers:
        - args:
          - /nginx-ingress-controller
  .....
          name: controller
          ports:
          - containerPort: 80
            name: http
            protocol: TCP
            hostPort: 80   # add
          - containerPort: 443
            name: https
            protocol: TCP
            hostPort: 443  # add
          - containerPort: 8443
            name: webhook
            protocol: TCP
  .....
        nodeSelector:
          kubernetes.io/os: linux
          ingress-nginx-node: "true"  # add
  
  # install 
  kubectl apply -f ingress-nginx.yaml
  ```


### Config Kubernetes-dashboard for Ingress

* Add kubernetes-dashboard rule 

  ```shell
  cat << EOF | sudo tee ingress-dashboard.yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    namespace: kubernetes-dashboard
    name: ingress-dashboard
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/use-regex: "true"
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
      nginx.org/ssl-services: "kubernetes-dashboard"
  spec:
    tls:
      - hosts:
        - "k8s.cloud.com"
        secretName: kubernetes-dashboard-certs
    rules:
    - host: k8s.cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                 name: kubernetes-dashboard
                 port:
                   number: 443
  EOF
  
  # apply 
  kubectl apply -f ingress-dashboard.yaml
  ```

* Create a user for the Dashboard

  ```shell
  cat << EOF | sudo tee kubernetes-dashboard-admin.yml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: kubernetes-dashboard-admin
    namespace: kubernetes-dashboard
  ---
  kind: ClusterRoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: kubernetes-dashboard-admin
  subjects:
    - kind: ServiceAccount
      name: kubernetes-dashboard-admin
      namespace: kubernetes-dashboard
  roleRef:
    kind: ClusterRole
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  EOF
  
  kubectl apply -f kubernetes-dashboard-admin.yml 
  kubectl create token kubernetes-dashboard-admin -n kubernetes-dashboard 
  ```

### Config Kubesphere fro ingress

* add kubesphere rule

  ```shell
  cat << EOF | sudo tee ingress-kubesphere.yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    namespace: kubesphere-system
    name: ingress-kubesphere
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/use-regex: "true"
      nginx.ingress.kubernetes.io/rewrite-target: /
  spec:
    rules:
    - host: ks.cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                 name: ks-console
                 port:
                   number: 80
  EOF
  
  # apply 
  kubectl apply -f ingress-kubesphere.yaml
  ```

### Config Harbor for ingress

* Add harbor rule

  ```shell
  cat << EOF | sudo tee ingress-harbor.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: harbor
  spec:
    ports:
      - port: 80
        protocol: TCP
        targetPort: 80
  ---
  apiVersion: v1
  kind: Endpoints
  metadata:
    name: harbor
  subsets:
    - addresses:
        - ip: 192.168.60.16
      ports:
        - port: 80
          protocol: TCP
  ---
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: ingress-harbor
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/use-regex: "true"
      nginx.ingress.kubernetes.io/rewrite-target: /
  spec:
    rules:
    - host: harbor.cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                 name: harbor
                 port:
                   number: 80
  EOF
  
  # apply
  kubectl apply -f ingress-harbor.yaml 
  ```

### Config Ceph for ingress

* Add ceph rule

  ```shell
  cat << EOF | sudo tee ingress-ceph.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: ceph
    namespace: ceph-csi
  spec:
    ports:
      - port: 443
        protocol: TCP
        targetPort: 8484
  ---
  apiVersion: v1
  kind: Endpoints
  metadata:
    name: ceph
    namespace: ceph-csi
  subsets:
    - addresses:
        - ip: 192.168.60.254
      ports:
        - port: 8484
          protocol: TCP
  ---
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: ingress-ceph
    namespace: ceph-csi
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/use-regex: "true"
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
  spec:
    tls:
      - hosts:
        - "ceph.cloud.com"
    rules:
    - host: ceph.cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                 name: ceph
                 port:
                   number: 443
  EOF
  
  # apply
  kubectl apply -f ingress-ceph.yaml 
  ```

### Config HAProxy for Ingress

* Add haproxy rule

  ```shell
  cat << EOF | sudo tee ingress-haproxy.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: haproxy
  spec:
    ports:
      - port: 80
        protocol: TCP
        targetPort: 10000
  ---
  apiVersion: v1
  kind: Endpoints
  metadata:
    name: haproxy
  subsets:
    - addresses:
        - ip: 192.168.60.254
      ports:
        - port: 10000
          protocol: TCP
  ---
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: ingress-haproxy
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/use-regex: "true"
      nginx.ingress.kubernetes.io/rewrite-target: /stats
  spec:
    rules:
    - host: haproxy.cloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                 name: haproxy
                 port:
                   number: 80
  EOF
  
  # apply
  kubectl apply -f ingress-haproxy.yaml 
  ```

## 
