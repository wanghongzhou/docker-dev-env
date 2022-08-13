# docker-dev-env

Docker script for building the development environment.

1. [Solr installation](./solr)
2. [Redis installation](./redis)
3. [MySQL installation](./mysql)
4. [Nacos installation](./nacos)
5. [Canal installation](./canal)
6. [Seata installation](./seata)
7. [Minio installation](./minio)
8. [Nginx installation](./nginx)
9. [RocketMQ installation](./rocketmq)
10. [RabbitMQ installation](./rabbitmq)
11. [Zookeeper installation](./zookeeper)
12. [Kibana installation](./kibana)
13. [Logstash installation](./logstash)
14. [Elasticsearch installation](./elasticsearch)
15. [Nexus installation](./nexus)
16. [Jenkins installation](./jenkins)
17. [Gitlab installation](./gitlab)
18. [Gitlab Runner installation](./gitlab-runner)
19. [Portainer installation](./portainer)
20. [Docker Registry installation](./docker-registry)
20. [Ceph, Harbor, K8s, Kubesphere installation](./cloud)
---

## Install docker in CentOS8

- Since March 2017, Docker has been divided into two branch versions: Docker CE and Docker EE.
- This article introduces the installation and use of Docker CE.

1. Remove it if an older version is already installed

   ```shell
   $ sudo yum remove docker \
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

2. First, to facilitate the addition of software sources and support for the Device Apper storage type, install the following software packages
   
   ```shell
   $ sudo yum update
   $ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ```
   
3. Add Docker stable version of yum software source

   ```shell
   $ sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. Then update the yum software source cache and install Docker

   ```shell
   $ sudo yum update
   $ sudo yum -y install docker-ce
   ```

5. Finally, confirm the normal startup of Docker service and set it to automatic startup

   ```shell
   $ sudo systemctl start docker
   $ sudo systemctl enable docker
   ```

6. Check the installed Docker version

   ```shell
   $ docker version
   ```

---

## Configuring docker registry mirror

1. Add the following configuration in the docker configuration file `/etc/docker/daemon.json`, If the file does not exist, create a new one.
   
   ```json
   {
       "registry-mirrors": [
           "https://ccr.ccs.tencentyun.com",
           "https://ustc-edu-cn.mirror.aliyuncs.com"
       ]
   }
   ```
   
2. Restart the docker

   ```shell
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart docker
   ```

3. Check whether the mirror takes effect

   ```shell
   $ docker info
   ...
   Registry: 
       https://ccr.ccs.tencentyun.com
       https://ustc-edu-cn.mirror.aliyuncs.com
   ```

---

## Install the docker compose

1. Execute the following command, you can customize the version you need by changing the version in the URL.

   ```shell
   $ curl -L https://get.daocloud.io/docker/compose/releases/download/v2.7.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   $ chmod +x /usr/local/bin/docker-compose
   ```

2. Check whether the installation is successful

   ```shell
   $ docker-compose version
   ```