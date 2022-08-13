# Elasticsearch

Elasticsearch is a distributed, free and open search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. Elasticsearch is built on Apache Lucene and was first released in 2010 by Elasticsearch N.V. (now known as Elastic).

---

## Prerequisite

- **Modify the limit configuration of the Linux system and adjust the number of open file handles and the number of started processes**
  
   ```shell
   $ vi /etc/security/limits.conf 
   * soft nofile 100001
   * hard nofile 100002
   * soft nproc 100001
   * hard nproc 100002
   ```
- **Alter system control permissions for ElasticSearch to create a virtual memory of 65536 bytes or more. By default, Linux does not allow any user or application to create this much virtual memory directly.**
  
   ```shell
   $ vi /etc/sysctl.conf 
   vm.max_map_count=655360
   ```

## Install Elasticsearch Standalone Mode Using Docker Compose

1. Before setting everything else, create a directory for Elasticsearch home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/elasticsearch/{data,plugins,certs,config}
   $ touch ~/docker/elasticsearch/config/elasticsearch.yml
   $ chown -R 1001:root ~/docker/elasticsearch
   ```
   
2. Modify the `~/docker/elasticsearch/config/elasticsearch.yml` file according to your requirements.

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ELASTICSEARCH_IMAGE=bitnami/elasticsearch:8.3.3
   ELASTICSEARCH_HEAP_SIZE=512m
    
   # Elasticsearch standalone mode
   ELASTICSEARCH_HOME=~/docker/elasticsearch
   ELASTICSEARCH_PORT_9200=9200
   ELASTICSEARCH_PORT_9300=9300
   ```

4. Make sure you are in the same directory as elasticsearch.yml and start Elasticsearch:

   ```shell
   $ docker-compose -f elasticsearch.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami Elasticsearch](https://hub.docker.com/r/bitnami/elasticsearch)

## Install Elasticsearch Cluster Mode Using Docker Compose

1. Before setting everything else, create a directory for Elasticsearch home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/elasticsearch-cluster/{node1,node2,node3}/{data,plugins,certs,config}
   $ touch ~/docker/elasticsearch-cluster/{node1,node2,node3}/config/elasticsearch.yml
   $ chown -R 1001:root ~/docker/elasticsearch-cluster
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ELASTICSEARCH_IMAGE=bitnami/elasticsearch:8.3.3
   ELASTICSEARCH_HEAP_SIZE=512m
   
   # Elasticsearch cluster mode
   ELASTICSEARCH_CLUSTER_HOME=~/docker/elasticsearch-cluster
   ELASTICSEARCH_CLUSTER_NODE1_HOSTNAME=elasticsearch-node1
   ELASTICSEARCH_CLUSTER_NODE1_PORT_9200=19200
   ELASTICSEARCH_CLUSTER_NODE1_PORT_9300=19300
   ELASTICSEARCH_CLUSTER_NODE2_HOSTNAME=elasticsearch-node2
   ELASTICSEARCH_CLUSTER_NODE2_PORT_9200=29200
   ELASTICSEARCH_CLUSTER_NODE2_PORT_9300=29300
   ELASTICSEARCH_CLUSTER_NODE3_HOSTNAME=elasticsearch-node3
   ELASTICSEARCH_CLUSTER_NODE3_PORT_9200=39200
   ELASTICSEARCH_CLUSTER_NODE3_PORT_9300=39300
   ```

3. Make sure you are in the same directory as elasticsearch-cluster.yml and start Elasticsearch:

   ```shell
   $ docker-compose -f elasticsearch-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami Elasticsearch](https://hub.docker.com/r/bitnami/elasticsearch)
