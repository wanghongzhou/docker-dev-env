# Redis

Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker.

---

## Install Redis standalone mode using Docker Compose

1. Before setting everything else, create a directory for Redis home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/redis/{data,conf,certs}
   $ chown -R 1001:root ~/docker/redis
   ``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   REDIS_IMAGE=bitnami/redis:7.0
   REDIS_PASSWORD=123456
   REDIS_ALLOW_EMPTY_PASSWORD=no    # If a password is set, set it to no
   
   # Redis standalone mode
   REDIS_HOME=~/docker/redis
   REDIS_PORT_6379=6379
   ```

3. Make sure you are in the same directory as redis.yml and start Redis:

   ```shell
   $ docker-compose -f redis.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Redis](https://hub.docker.com/r/bitnami/redis)

## Install Redis replication mode using Docker Compose

1. Before setting everything else, create a directory for Redis home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/redis-replica/{master/{data,conf,certs},slave/{data,conf,certs}}
   $ chown -R 1001:root ~/docker/redis-replica
   ``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   REDIS_IMAGE=bitnami/redis:7.0
   REDIS_PASSWORD=123456
   REDIS_ALLOW_EMPTY_PASSWORD=no    # If a password is set, set it to no
   
   # Redis replica mode
   REDIS_REPLICA_HOME=~/docker/redis-replica
   REDIS_REPLICA_MASTER_PORT_6379=6381
   REDIS_REPLICA_SLAVE_PORT_6379=6382
   ```

3. Make sure you are in the same directory as redis-replica.yml and start Redis:

   ```shell
   $ docker-compose -f redis-replica.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Redis](https://hub.docker.com/r/bitnami/redis)

## Install Redis sentinel mode using Docker Compose

1. Before setting everything else, create a directory for Redis home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/redis-sentinel/{master/{data,conf,certs},slave-1/{data,conf,certs},slave-2/{data,conf,certs},sentinel-1/conf,sentinel-2/conf,sentinel-3/conf}
   $ chown -R 1001:root ~/docker/redis-sentinel
   ``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   REDIS_IMAGE=bitnami/redis:7.0
   REDIS_PASSWORD=123456
   REDIS_ALLOW_EMPTY_PASSWORD=no    # If a password is set, set it to no
   
   # redis sentinel mode
   REDIS_SENTINEL_IMAGE=bitnami/redis-sentinel:6.2
   REDIS_SENTINEL_HOME=~/docker/redis-sentinel
   REDIS_SENTINEL_IP=redis.example.com             # your host ip or domain name or public IP address
   REDIS_SENTINEL_MASTER_PORT_6379=7001
   REDIS_SENTINEL_SLAVE1_PORT_6379=7002
   REDIS_SENTINEL_SLAVE2_PORT_6379=7003
   REDIS_SENTINEL_SENTINEL1_PORT_26379=27001
   REDIS_SENTINEL_SENTINEL2_PORT_26379=27002
   REDIS_SENTINEL_SENTINEL3_PORT_26379=27003
   ```

3. Make sure you are in the same directory as redis-sentinel.yml and start Redis:

   ```shell
   $ docker-compose -f redis-sentinel.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Redis Sentinel](https://hub.docker.com/r/bitnami/redis-sentinel)

## Install Redis cluster mode using Docker Compose

1. Before setting everything else, create a directory for Redis home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/redis-cluster/{node-1/{data,conf,certs},node-2/{data,conf,certs},node-3/{data,conf,certs},node-4/{data,conf,certs},node-5/{data,conf,certs},node-6/{data,conf,certs}}
   $ chown -R 1001:root ~/docker/redis-cluster
   ``` 

2. Modify the `.env` file, **the password must be set in cluster mode**, you can fine tune these configurations to meet
   your requirements.

   ```properties
   # common
   REDIS_IMAGE=bitnami/redis:7.0
   REDIS_PASSWORD=123456            # The password must be set in cluster mode
   REDIS_ALLOW_EMPTY_PASSWORD=no    # The allow must be set to no in cluster mode
   
   # redis cluster mode
   REDIS_CLUSTER_IMAGE=bitnami/redis-cluster:6.2
   REDIS_CLUSTER_HOME=~/docker/redis-cluster
   REDIS_CLUSTER_NODE1_PORT_6379=6391
   REDIS_CLUSTER_NODE2_PORT_6379=6392
   REDIS_CLUSTER_NODE3_PORT_6379=6393
   REDIS_CLUSTER_NODE4_PORT_6379=6394
   REDIS_CLUSTER_NODE5_PORT_6379=6395
   REDIS_CLUSTER_NODE6_PORT_6379=6396
   ```

3. Make sure you are in the same directory as redis-cluster.yml and start Redis:

   ```shell
   $ docker-compose -f redis-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Redis Cluster](https://hub.docker.com/r/bitnami/redis-cluster)