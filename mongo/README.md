# MongoDB

MongoDB® is a relational open source NoSQL database. Easy to use, it stores data in JSON-like documents. Automated scalability and high-performance. Ideal for developing cloud native applications.

---

## Install MongoDB Standalone Mode Using Docker Compose

1. Before setting everything else, create a directory for MongoDB home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/mongodb
   $ chown -R 1001:root ~/docker/mongodb
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MONGODB_IMAGE=bitnami/mongodb:6.0
   MONGODB_ALLOW_EMPTY_PASSWORD=no
   MONGODB_ENABLE_JOURNAL=true
   MONGODB_ENABLE_NUMACTL=true
   MONGODB_ENABLE_DIRECTORY_PER_DB=yes
   MONGODB_EXTRA_FLAGS=--wiredTigerCacheSizeGB=2
   MONGODB_USERNAME=mongodb
   MONGODB_PASSWORD=mongodb
   MONGODB_DATABASE=app-social
   MONGODB_ROOT_PASSWORD=mongodb
   
   # mongodb standalone mode
   MONGODB_HOME=~/docker/mongodb
   MONGODB_PORT_27017=27017
   ```

3. Make sure you are in the same directory as mongodb.yml and start MongoDB:

   ```shell
   $ docker-compose -f mongodb.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami MongoDB](https://hub.docker.com/r/bitnami/mongodb)

## Install MongoDB Replica set Mode Using Docker Compose

1. Before setting everything else, create a directory for MongoDB home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/mongo-replica/{primary,secondary,arbiter}
   $ chown -R 1001:root ~/docker/mongo-replica
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MONGODB_IMAGE=bitnami/mongodb:6.0
   MONGODB_ALLOW_EMPTY_PASSWORD=no
   MONGODB_ENABLE_JOURNAL=true
   MONGODB_ENABLE_NUMACTL=true
   MONGODB_ENABLE_DIRECTORY_PER_DB=yes
   MONGODB_EXTRA_FLAGS=--wiredTigerCacheSizeGB=2
   MONGODB_USERNAME=mongodb
   MONGODB_PASSWORD=mongodb
   MONGODB_DATABASE=app-social
   MONGODB_ROOT_PASSWORD=mongodb
   
   # mongodb replica mode
   MONGODB_REPLICA_SET_KEY=mongodb_replica_set_key
   MONGODB_REPLICA_PRIMARY_PORT_27017=27017
   MONGODB_REPLICA_PRIMARY_HOME=~/docker/mongodb-replica/primary
   MONGODB_REPLICA_PRIMARY_ADVERTISED_HOSTNAME=mongodb-primary
   MONGODB_REPLICA_SECONDARY_PORT_27017=27018
   MONGODB_REPLICA_SECONDARY_HOME=~/docker/mongodb-replica/secondary
   MONGODB_REPLICA_SECONDARY_ADVERTISED_HOSTNAME=mongodb-secondary
   MONGODB_REPLICA_ARBITER_PORT_27017=27019
   MONGODB_REPLICA_ARBITER_HOME=~/docker/mongodb-replica/arbiter
   MONGODB_REPLICA_ARBITER_ADVERTISED_HOSTNAME=mongodb-arbiter
   ```

3. Make sure you are in the same directory as mongodb-replica.yml and start MongoDB:

   ```shell
   $ docker-compose -f mongodb-replica.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami MongoDB](https://hub.docker.com/r/bitnami/mongodb)
