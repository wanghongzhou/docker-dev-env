# Solr

Solr is highly reliable, scalable and fault tolerant, providing distributed indexing, replication and load-balanced querying, automated failover and recovery, centralized configuration and more. Solr powers the search and navigation features of many of the world's largest internet sites.

---

## Install Solr Standalone Mode Using Docker Compose

1. Before setting everything else, create a directory for Solr home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/solr
   $ chown -R 1001:root ~/docker/solr
   ```
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   SOLR_IMAGE=bitnami/solr:9
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

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami Solr](https://hub.docker.com/r/bitnami/solr)

## Install Solr Cluster Mode Using Docker Compose

- ### Prerequisite
    - **Before installing Solr cluster, you must [install the Zookeeper](../zookeeper), you have to make sure that the attributes `ZOO_ALLOW_ANONYMOUS_LOGIN=yes` and `ZOO_4LW_COMMANDS_WHITELIST=srvr, mntr, conf,ruok`**

1. Before setting everything else, create a directory for Solr home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/solr-cluster/{node1,node2,node3}
   $ chown -R 1001:root ~/docker/solr-cluster
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   SOLR_IMAGE=bitnami/solr:9
   SOLR_OPTS=-Xms512m -Xmx512m -XX:+AggressiveOpts -XX:G1HeapRegionSize=8m
   SOLR_ENABLE_AUTHENTICATION=yes
   SOLR_ADMIN_USERNAME=admin
   SOLR_ADMIN_PASSWORD=admin
   
   # solr cluster mode
   SOLR_CLUSTER_HOME=~/docker/solr-cluster
   SOLR_CLUSTER_NODE1_HOST=solr-cluster-node1
   SOLR_CLUSTER_NODE1_PORT_8983=18983
   SOLR_CLUSTER_NODE2_HOST=solr-cluster-node2
   SOLR_CLUSTER_NODE2_PORT_8983=28983
   SOLR_CLUSTER_NODE3_HOST=solr-cluster-node3
   SOLR_CLUSTER_NODE3_PORT_8983=38983
   SOLR_CLUSTER_ZOO_SERVICE_NETWORK=dev_zookeeper   # In Zookeeper cluster mode, set it to dev_zookeeper_cluster
   SOLR_CLUSTER_ZOO_SERVICE_HOSTS=zookeeper:2181    # In Zookeeper cluster mode, set it to zookeeper-cluster-node1:2181,zookeeper-cluster-node2:2181,zookeeper-cluster-node3:2181
   ```

3. Make sure you are in the same directory as solr-cluster.yml and start Solr:

   ```shell
   $ docker-compose -f solr-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami Solr](https://hub.docker.com/r/bitnami/solr)
