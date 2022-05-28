# Canal

Canal is an open source product provided by Alibaba Group. Canal can parse incremental log data in Canal and allows you
to subscribe to and consume incremental data. You can use Canal to synchronize incremental data from a Canal database to
an Alibaba Cloud Elasticsearch cluster.

---

<a id = "jump1"></a>

## Install Canal Admin using Docker Compose

- ### Prerequisite
    - **Before installing Canal Admin, you must [install the MySQL](../mysql).**
    - **You must import the Canal
      Admin [SQL script](https://github.com/alibaba/canal/blob/master/admin/admin-web/src/main/resources/canal_manager.sql)
      .**

1. Before setting everything else, create a directory for Canal Admin home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/canal-admin/logs
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # canal admin
   CANAL_ADMIN_IMAGE=canal/canal-admin:v1.1.6
   CANAL_ADMIN_HOME=~/docker/canal-admin
   CANAL_ADMIN_PORT_8089=8089
   CANAL_ADMIN_MYSQL_SERVICE_NETWORK=dev_mysql
   CANAL_ADMIN_DATASOURCE_ADDRESS=mysql:3306
   CANAL_ADMIN_DATASOURCE_DATABASE=canal_manager
   CANAL_ADMIN_DATASOURCE_USERNAME=root
   CANAL_ADMIN_DATASOURCE_PASSWORD=123456
   CANAL_ADMIN_CANAL_ADMINUSER=admin
   CANAL_ADMIN_CANAL_ADMINPASSWD=admin
   ```

3. Make sure you are in the same directory as canal-admin.yml and start Canal Admin, log in with username `admin` and
   the password `123456`:

   ```shell
   $ docker-compose -f canal-admin.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Alibaba Canal](https://github.com/alibaba/canal)

## Install Canal standalone mode using Docker Compose

1. Before setting everything else, create a directory for Canal home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/canal/logs
   ```

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # canal common
   CANAL_IMAGE=canal/canal-server:v1.1.6
   CANAL_ADMIN_MANAGER=example.canal-admin.com:8089
   CANAL_ADMIN_USER=admin
   CANAL_ADMIN_PASSWD=4ACFE3202A5FF5CF467898FC58AAB1D615029441
   
   
   # canal standalone mode
   CANAL_HOME=~/docker/canal
   CANAL_PORT_11110=11110
   CANAL_PORT_11111=11111
   CANAL_PORT_11112=11112
   CANAL_REGISTER_IP=example.canal.com
   ```

4. Make sure you are in the same directory as canal.yml and start Canal:

   ```shell
   $ docker-compose -f canal.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Alibaba Canal](https://github.com/alibaba/canal)

## Install Canal cluster mode using Docker Compose

- ### Prerequisite
    - **Before installing Canal cluster mode, you must [install the Zookeeper](../zookeeper).**
    - **You must create a cluster named `canal-cluster` in the [Canal Admin](#jump1).**

1. Before setting everything else, create a directory for Canal home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/canal-cluster/{node1/logs,node2/logs,node3/logs}
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # canal common
   CANAL_IMAGE=canal/canal-server:v1.1.6
   CANAL_ADMIN_MANAGER=example.canal-admin.com:8089
   CANAL_ADMIN_USER=admin
   CANAL_ADMIN_PASSWD=4ACFE3202A5FF5CF467898FC58AAB1D615029441
   
   # canal cluster mode
   CANAL_CLUSTER_HOME=~/docker/canal-cluster
   CANAL_CLUSTER_NAME=canal-cluster
   CANAL_CLUSTER_ZKSERVERS=example1.zookeeper.com:2181,example2.zookeeper.com:2181,example3.zookeeper.com:2181
   CANAL_CLUSTER_NODE1_PORT_11110=11120
   CANAL_CLUSTER_NODE1_PORT_11111=11121
   CANAL_CLUSTER_NODE1_PORT_11112=11122
   CANAL_CLUSTER_NODE1_REGISTER_IP=example.canal.com
   CANAL_CLUSTER_NODE2_PORT_11110=11130
   CANAL_CLUSTER_NODE2_PORT_11111=11131
   CANAL_CLUSTER_NODE2_PORT_11112=11132
   CANAL_CLUSTER_NODE2_REGISTER_IP=example.canal.com
   CANAL_CLUSTER_NODE3_PORT_11110=11140
   CANAL_CLUSTER_NODE3_PORT_11111=11141
   CANAL_CLUSTER_NODE3_PORT_11112=11142
   CANAL_CLUSTER_NODE3_REGISTER_IP=example.canal.com
   ```

3. Make sure you are in the same directory as canal-cluster.yml and start Canal:

   ```shell
   $ docker-compose -f canal-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Alibaba Canal](https://hub.docker.com/r/bitnami/canal)