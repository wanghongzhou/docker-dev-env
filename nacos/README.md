# Nacos

Nacos is committed to help you discover, configure, and manage your microservices. It provides a set of simple and useful features enabling you to realize dynamic service discovery, service configuration, service metadata and traffic management.

---

## Prerequisite

- **Before installing Nacos, you must [install the MySQL](../mysql).**
- **You must create a database and import the [Nacos SQL script](https://github.com/alibaba/nacos/releases).**

## Install Nacos Standalone Mode Using Docker Compose

1. Before setting everything else, create a directory for Nacos home mount. Ensure that the directory exists and appropriate permission have been granted.
   
    ```shell
    $ mkdir -vp ~/docker/nacos/{logs,init.d}
    $ touch ~/docker/nacos/init.d/custom.properties
    ```
   
2. Modify the `~/docker/nacos/init.d/custom.properties` file according to your requirements.

    ```properties
    nacos.core.auth.enabled=false
    ```

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

    ```properties
    # common
    NACOS_IMAGE=nacos/nacos-server:v2.1.0
    NACOS_JVM_XMS=512m
    NACOS_JVM_XMX=512m
    NACOS_AUTH_ENABLE=false
    NACOS_PREFER_HOST_MODE=hostname
    NACOS_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
    NACOS_MYSQL_SERVICE_HOST=mysql          # In MySQL replica mode, mysql-replica-master
    NACOS_MYSQL_SERVICE_PORT=3306
    NACOS_MYSQL_SERVICE_USER=root
    NACOS_MYSQL_SERVICE_PASSWORD=123456
    NACOS_MYSQL_SERVICE_DB_NAME=nacos
    
    # Nacos standalone mode
    NACOS_HOME=~/docker/nacos
    NACOS_PORT_8848=8848
    NACOS_PORT_9848=9848
    NACOS_PORT_9555=9555
    ```

4. Make sure you are in the same directory as nacos.yml and start Nacos, Visit the Nacos URL `http://host:8848/nacos`, and log in with username `nacos` and the password `nacos`:
   
    ```shell
    $ docker-compose -f nacos.yml up -d
    ```
   
5. If something else goes wrong, for more detailed tutorial can be found on the [Nacos](https://hub.docker.com/r/nacos/nacos-server)

## Install Nacos Cluster Mode Using Docker Compose

1. Before setting everything else, create a directory for Nacos home mount. Ensure that the directory exists and appropriate permission have been granted.
   
    ```shell
    $ mkdir -vp ~/docker/nacos-cluster/{node1,node2,node3}/{logs,init.d}
    $ touch ~/docker/nacos-cluster/{node1,node2,node3}/init.d/custom.properties
    ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

    ```properties
    # common
    NACOS_IMAGE=nacos/nacos-server:v2.1.0
    NACOS_JVM_XMS=512m
    NACOS_JVM_XMX=512m
    NACOS_AUTH_ENABLE=false
    NACOS_PREFER_HOST_MODE=hostname
    NACOS_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
    NACOS_MYSQL_SERVICE_HOST=mysql          # In MySQL replica mode, mysql-replica-master
    NACOS_MYSQL_SERVICE_PORT=3306
    NACOS_MYSQL_SERVICE_USER=root
    NACOS_MYSQL_SERVICE_PASSWORD=123456
    NACOS_MYSQL_SERVICE_DB_NAME=nacos
    
    # Nacos cluster mode
    NACOS_CLUSTER_HOME=~/docker/nacos-cluster
    NACOS_CLUSTER_NODE1_PORT_8848=18848
    NACOS_CLUSTER_NODE1_PORT_9848=19848
    NACOS_CLUSTER_NODE1_PORT_9555=19555
    NACOS_CLUSTER_NODE2_PORT_8848=28848
    NACOS_CLUSTER_NODE2_PORT_9848=29848
    NACOS_CLUSTER_NODE2_PORT_9555=29555
    NACOS_CLUSTER_NODE3_PORT_8848=38848
    NACOS_CLUSTER_NODE3_PORT_9848=39848
    NACOS_CLUSTER_NODE3_PORT_9555=39555
    ```

3. Make sure you are in the same directory as nacos-cluster.yml and start Nacos, Visit the Nacos URL `http://host:8848/nacos`, and log in with username `nacos` and the password `nacos`:
   
    ```shell
    $ docker-compose -f nacos-cluster.yml up -d
    ```
   
4. If something else goes wrong, for more detailed tutorial can be found on the [Nacos](https://hub.docker.com/r/nacos/nacos-server)
