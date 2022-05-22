# Solr

Solr is highly reliable, scalable and fault tolerant, providing distributed indexing, replication and load-balanced
querying, automated failover and recovery, centralized configuration and more. Solr powers the search and navigation
features of many of the world's largest internet sites.

---

## Install Solr standalone mode using Docker Compose

1. Before setting everything else, create a directory for Solr home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/solr
   $ chown -R 1001:root ~/docker/solr
   ```
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   SOLR_IMAGE=bitnami/solr:9.0.0
   SOLR_OPTS=-Xms512m -Xmx512m -XX:+AggressiveOpts -XX:G1HeapRegionSize=8m
   SOLR_ENABLE_AUTHENTICATION=yes
   SOLR_ADMIN_USERNAME=admin
   SOLR_ADMIN_PASSWORD=admin
   
   # solr standalone mode
   SOLR_HOME=~/docker/solr
   SOLR_PORT_8983=8983
   ```

4. Make sure you are in the same directory as solr.yml and start Solr:

   ```shell
   $ docker-compose -f solr.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Solr](https://hub.docker.com/r/bitnami/solr)

## Install Solr cluster mode using Docker Compose

- **Before installing Solr cluster, you must [install the Zookeeper](../zookeeper/README.md).**
- **You have to make sure that the attributes `ZOO_ALLOW_ANONYMOUS_LOGIN=yes`
  and `ZOO_4LW_COMMANDS_WHITELIST=srvr, mntr, conf,ruok`**

1. Before setting everything else, create a directory for Solr home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/solr-cluster/{node1/,node2/,node3}
   $ chown -R 1001:root ~/docker/solr-cluster
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   SOLR_IMAGE=bitnami/solr:8.0.29
   SOLR_CHARACTER_SET=utf8mb4
   SOLR_COLLATE=utf8mb4_general_ci
   SOLR_ROOT_PASSWORD=123456
   SOLR_AUTHENTICATION_PLUGIN=solr_native_password
   
   # solr cluster mode
   SOLR_REPLICA_HOME=~/docker/solr-replica
   SOLR_REPLICA_MASTER_PORT_3306=3307
   SOLR_REPLICA_SLAVE1_PORT_3306=3308
   SOLR_REPLICA_SLAVE2_PORT_3306=3309
   SOLR_REPLICA_SLAVE3_PORT_3306=3310
   ```

3. Make sure you are in the same directory as solr-replica.yml and start Solr:

   ```shell
   $ docker-compose -f solr-replica.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Solr](https://hub.docker.com/r/bitnami/solr)
