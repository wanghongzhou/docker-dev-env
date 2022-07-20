# Seata

Seata is an open source distributed transaction solution dedicated to providing high performance and easy to use
distributed transaction services. Seata will provide users with AT, TCC, SAGA, and XA transaction models to create a
one-stop distributed solution for users.

---

## Install Seata standalone mode using Docker Compose

- Before setting everything else, create a directory for Seata home mount. Ensure that the directory exists and
  appropriate permission have been granted.

    ```shell
    $ mkdir -vp ~/docker/seata/logs
    $ touch ~/docker/seata/application.yml
    ```

### No registry and file store mode

1. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

    ```properties 
    # common
    SEATA_IMAGE=seataio/seata-server:1.5.2
    
    # Seata standalone mode
    SEATA_HOME=~/docker/seata
    
    # Seata standalone mode (No registry and file store)
    SEATA_DEFAULT_PORT_7091=7091
    SEATA_DEFAULT_PORT_8091=8091
    ```
2. Make sure you are in the same directory as seata-default.yml and start Seata:

    ```shell
    $ docker-compose -f seata-default.yml up -d
    ```

### No registry and db store mode

- #### Prerequisite
    - **Before installing Seata, you must [install the MySQL](../mysql).**
    - **You must create a database and import the corresponding version of
      the [Seata Server SQL script](https://github.com/seata/seata/tree/develop/script/server/db)
      and [Seata Client SQL script](https://github.com/seata/seata/tree/develop/script/client).**

1. Write the following configuration to the `~/docker/seata/application.yml` file and modify it as required:

    ```yaml  
    server:
      port: 7091

    spring:
      application:
        name: seata-server

    logging:
      config: classpath:logback-spring.xml
      file:
        path: ${user.home}/logs/seata
   
    console:
      user:
        username: seata
        password: seata
   
    seata:
      config:
        type: file
      registry:
        type: file
      store:
        mode: db
        session:
          mode: db
        lock:
          mode: db
        db:
          datasource: druid
          db-type: mysql
          ## MySQL5 user driver class name is "com.mysql.jdbc.Driver"
          driver-class-name: com.mysql.cj.jdbc.Driver
          ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
          url: jdbc:mysql://mysql:3306/seata?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
          user: root
          password: 123456
          min-conn: 5
          max-conn: 100
          global-table: global_table
          branch-table: branch_table
          lock-table: lock_table
          distributed-lock-table: distributed_lock
          query-limit: 100
          max-wait: 5000
      security:
        secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
        tokenValidityInMilliseconds: 1800000
        ignore:
          urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/api/v1/auth/login
    ```

2. Modify the following attributes of the `.env` file:

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.2
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
       
   # Seata standalone mode
   SEATA_HOME=~/docker/seata
     
   # Seata standalone mode (No registry and db store mode)
   SEATA_DB_ONLY_PORT_7091=7091
   SEATA_DB_ONLY_PORT_8091=8091
   ```

3. Make sure you are in the same directory as seata-db-only.yml and start Seata:

   ```shell
   $ docker-compose -f seata-db-only.yml up -d
   ```

### Using the Registry and db store mode

- #### Prerequisite
    - **Before installing Seata, you must [install the Nacos](../nacos/README.md) and make sure it is started.**

1. Log in to your Nacos and add the following configuration, the Data ID is `seataServer.properties`, See here
   for [more configurations](https://github.com/seata/seata/blob/develop/script/config-center/config.txt):
   ```properties
   #Transaction storage configuration, only for the server. The file, DB, and redis configuration values are optional.
   store.mode=db
   store.lock.mode=db
   store.session.mode=db
   
   #Used for password encryption
   store.publicKey=""
   
   #These configurations are required if the `store mode` is `db`. If `store.mode,store.lock.mode,store.session.mode` are not equal to `db`, you can remove the configuration block.
   store.db.datasource=druid
   store.db.dbType=mysql
   store.db.driverClassName=com.mysql.cj.jdbc.Driver
   store.db.url=jdbc:mysql://mysql:3306/seata?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
   store.db.user=mysql
   store.db.password=123456
   store.db.minConn=5
   store.db.maxConn=30
   store.db.globalTable=global_table
   store.db.branchTable=branch_table
   store.db.distributedLockTable=distributed_lock
   store.db.queryLimit=100
   store.db.lockTable=lock_table
   store.db.maxWait=5000
   ```
2. Write the following configuration to the `~/docker/seata/application.yml` file and modify it as required:

   ```yaml  
   server:
     port: 7091

   spring:
     application:
       name: seata-server

   logging:
     config: classpath:logback-spring.xml
     file:
       path: ${user.home}/logs/seata
   
   console:
     user:
       username: seata
       password: seata
   
   seata:
     config:
       type: nacos
       nacos:
         server-addr: nacos:8848
         namespace:
         group: SEATA_GROUP
         username: nacos
         password: nacos
         data-id: seataServer.properties
     registry:
       type: nacos
       nacos:
         application: seata-server
         server-addr: nacos:8848
         group: SEATA_GROUP
         namespace:
         cluster: default
         username: nacos
         password: nacos
     security:
       secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
       tokenValidityInMilliseconds: 1800000
       ignore:
         urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/api/v1/auth/login
   ```

3. Modify the following attributes of the `.env` file:

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.2
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
   SEATA_NACOS_SERVICE_NETWORK=dev_nacos   # In Nacos cluster mode, set it to dev_nacos_cluster
       
   # Seata standalone mode
   SEATA_HOME=~/docker/seata
     
   # Seata standalone mode (Using the Registry and db store mode)
   SEATA_HOST=seata
   SEATA_PORT_7091=7091
   SEATA_PORT_8091=8091
   ```

4. Make sure you are in the same directory as seata.yml and start Seata:

   ```shell
   $ docker-compose -f seata.yml up -d
   ```

## Install Seata cluster mode using Docker Compose

- #### Prerequisite
    - This cluster model is based on Nacos and mysql, please know how to [Using the Registry and db store mode](#using-the-registry-and-db-store-mode)

1. Before setting everything else, create a directory for Seata home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/seata-cluster/{node1,node2,node3}/logs
   $ touch ~/docker/seata-cluster/{node1,node2,node3}/application.yml
   ```
2. **Write the configuration to
   the ` ~/docker/seata-cluster/{node1/application.yml,node2/application.yml,node3/application.yml}` file，
   See [Using the Registry and db store mode](#jump1)**
3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.2
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
   SEATA_NACOS_SERVICE_NETWORK=dev_nacos   # The network for Nacos network

   # Seata cluster mode (Using the Registry and db store mode)
   SEATA_CLUSTER_HOME=~/docker/seata-cluster
   SEATA_CLUSTER_NODE1_PORT_7091=17091
   SEATA_CLUSTER_NODE1_PORT_8091=18091
   SEATA_CLUSTER_NODE1_HOST=seata-cluster-node1
   SEATA_CLUSTER_NODE2_PORT_7091=27091
   SEATA_CLUSTER_NODE2_PORT_8091=28091
   SEATA_CLUSTER_NODE2_HOST=seata-cluster-node2
   SEATA_CLUSTER_NODE3_PORT_7091=37091
   SEATA_CLUSTER_NODE3_PORT_8091=38091
   SEATA_CLUSTER_NODE3_HOST=seata-cluster-node3
   ```

4. Make sure you are in the same directory as seata-cluster.yml and start Seata:

   ```shell
   $ docker-compose -f seata-cluster.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Seata](https://hub.docker.com/r/seataio/seata-server)
