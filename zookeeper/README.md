# Zookeeper

ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed
synchronization, and providing group services.

---

## Install Zookeeper standalone mode using Docker Compose

1. Before setting everything else, create a directory for Zookeeper home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/zookeeper
   $ chown -R 1001:root ~/docker/zookeeper
   ```

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ZOO_IMAGE=bitnami/zookeeper:3.8.0
   ZOO_HEAP_SIZE=256
   ZOO_4LW_COMMANDS_WHITELIST=srvr, mntr, conf,ruok
   ZOO_ALLOW_ANONYMOUS_LOGIN=yes        # Set to yes, the user Settings are invalid
   ZOO_ENABLE_AUTH=false                # Set to false, ZOO_ALLOW_ANONYMOUS_LOGIN must is yes
   ZOO_ENABLE_ADMIN_SERVER=true         # Whether to start the admin server, http://ip:8080/commands
   ZOO_SERVER_USERS=user1,user2,admin
   ZOO_SERVER_PASSWORDS=pass4user1,pass4user2,pass4admin
   ZOO_CLIENT_USER=user1
   ZOO_CLIENT_PASSWORD=pass4user1

   # zookeeper standalone mode
   ZOO_HOME=~/docker/zookeeper
   ZOO_PORT_2181=2181
   ZOO_PORT_8080=8080
   ```

4. Make sure you are in the same directory as zookeeper.yml and start Zookeeper:

   ```shell
   $ docker-compose -f zookeeper.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Zookeeper](https://hub.docker.com/r/bitnami/zookeeper)

## Install Zookeeper cluster mode using Docker Compose

1. Before setting everything else, create a directory for Zookeeper home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/zookeeper-cluster/{node1/,node2/,node3/}
   $ chown -R 1001:root ~/docker/zookeeper-cluster
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ZOO_IMAGE=bitnami/zookeeper:3.8.0
   ZOO_HEAP_SIZE=256
   ZOO_4LW_COMMANDS_WHITELIST=srvr, mntr, conf,ruok
   ZOO_ALLOW_ANONYMOUS_LOGIN=yes        # Set to yes, the user Settings are invalid
   ZOO_ENABLE_AUTH=false                # Set to false, ZOO_ALLOW_ANONYMOUS_LOGIN must is yes
   ZOO_ENABLE_ADMIN_SERVER=true         # Whether to start the admin server, http://ip:8080/commands
   ZOO_SERVER_USERS=user1,user2,admin
   ZOO_SERVER_PASSWORDS=pass4user1,pass4user2,pass4admin
   ZOO_CLIENT_USER=user1
   ZOO_CLIENT_PASSWORD=pass4user1
   
   # zookeeper cluster mode
   ZOO_CLUSTER_HOME=~/docker/zookeeper-cluster
   ZOO_CLUSTER_NODE1_2181=12181
   ZOO_CLUSTER_NODE1_8080=18080
   ZOO_CLUSTER_NODE2_2181=22181
   ZOO_CLUSTER_NODE2_8080=28080
   ZOO_CLUSTER_NODE3_2181=32181
   ZOO_CLUSTER_NODE3_8080=38080
   ```

3. Make sure you are in the same directory as zookeeper-cluster.yml and start Zookeeper:

   ```shell
   $ docker-compose -f zookeeper-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Zookeeper](https://hub.docker.com/r/bitnami/zookeeper)
