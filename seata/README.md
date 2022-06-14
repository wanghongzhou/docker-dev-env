# Seata

Seata is an open source distributed transaction solution dedicated to providing high performance and easy to use
distributed transaction services. Seata will provide users with AT, TCC, SAGA, and XA transaction models to create a
one-stop distributed solution for users.

---

## Install Seata standalone mode using Docker Compose

- Before setting everything else, create a directory for Seata home mount. Ensure that the directory exists and
  appropriate permission have been granted.

    ```shell
    $ mkdir -vp ~/docker/seata/
    $ touch ~/docker/seata/{file.conf,registry.conf}
    ```

### No registry and file store mode

1. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

    ```properties 
    # common
    SEATA_IMAGE=seataio/seata-server:1.5.1
    
    # Seata standalone mode
    SEATA_HOME=~/docker/seata
    
    # Seata standalone mode (No registry and file store)
    SEATA_DEFAULT_PORT_8091=8091
    ```
2. Make sure you are in the same directory as seata-default.yml and start Seata:

    ```shell
    $ docker-compose -f seata-default.yml up -d
    ```

### No registry and db store mode

- #### Prerequisite
    - **Before installing Seata, you must [install the MySQL](../mysql).**
    - **You must create a database and import
      the [Seata SQL script](https://github.com/seata/seata/tree/develop/script/server/db).**

1. Write the following configuration to the `~/docker/seata/file.conf` file and modify it as required:

    ```properties  
    ## transaction log store, only used in seata-server
    store {
      ## store mode: file、db、redis
      mode = "db"
      ## rsa decryption public key
      publicKey = ""
      ## file store property
      db {
        ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
        datasource = "druid"
        ## mysql/oracle/postgresql/h2/oceanbase etc.
        dbType = "mysql"
        ## MySQL5 user driver class name is "com.mysql.jdbc.Driver"
        driverClassName = "com.mysql.cj.jdbc.Driver"
        ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
        url = "jdbc:mysql://mysql:3306/seata?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai"
        user = "mysql"
        password = "mysql"
        minConn = 5
        maxConn = 100
        globalTable = "global_table"
        branchTable = "branch_table"
        lockTable = "lock_table"
        queryLimit = 100
        maxWait = 5000
      }
    }
    ```

2. Modify the following attributes of the `.env` file:

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.1
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
       
   # Seata standalone mode
   SEATA_HOME=~/docker/seata
     
   # Seata standalone mode (No registry and file store)
   SEATA_DEFAULT_PORT_8091=8091
   ```

3. Make sure you are in the same directory as seata-db-only.yml and start Seata:

   ```shell
   $ docker-compose -f seata-db-only.yml up -d
   ```

<a id = "jump1"></a>

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
   store.db.url=jdbc:mysql://mysql:3306/seata?rewriteBatchedStatements=true&useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai
   store.db.user=mysql
   store.db.password=mysql
   store.db.minConn=5
   store.db.maxConn=30
   store.db.globalTable=global_table
   store.db.branchTable=branch_table
   store.db.distributedLockTable=distributed_lock
   store.db.queryLimit=100
   store.db.lockTable=lock_table
   store.db.maxWait=5000
   ```
2. Write the following configuration to the `~/docker/seata/registry.conf` file and modify it as required:

   ```properties  
   registry {
     # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
     type = "nacos"
   
     nacos {
       application = "seata-server"
       serverAddr = "nacos:8848"
       group = "SEATA_GROUP"
       namespace = ""
       cluster = "default"
       username = "nacos"
       password = "nacos"
     }
   }
   
   config {
     # file、nacos 、apollo、zk、consul、etcd3
     type = "nacos"
   
     nacos {
       serverAddr = "nacos:8848"
       namespace = ""
       group = "SEATA_GROUP"
       username = "nacos"
       password = "nacos"
       dataId = "seataServer.properties"
     }
   }
   ```

3. Modify the following attributes of the `.env` file:

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.1
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
   SEATA_NACOS_SERVICE_NETWORK=dev_nacos   # In Nacos cluster mode, set it to dev_nacos_cluster
       
   # Seata standalone mode
   SEATA_HOME=~/docker/seata
     
   # Seata standalone mode (Using the Registry and db store mode)
   SEATA_PORT_8091=8091
   SEATA_HOST=seata
   ```

4. Make sure you are in the same directory as seata.yml and start Seata:

   ```shell
   $ docker-compose -f seata.yml up -d
   ```

## Install Seata cluster mode using Docker Compose

- #### Prerequisite
    - This cluster model is based on Nacos and mysql, please know how to [Using the Registry and db store mode](#jump1)

1. Before setting everything else, create a directory for Seata home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/seata-cluster/{node1/,node2/,node3/}
   $ touch ~/docker/seata-cluster/{node1/registry.conf,node2/registry.conf,node3/registry.conf}
   ```
2. **Write the configuration to
   the ` ~/docker/seata-cluster/{node1/registry.conf,node2/registry.conf,node3/registry.conf}` file，
   See [Using the Registry and db store mode](#jump1)**
3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties 
   # common
   SEATA_IMAGE=seataio/seata-server:1.5.1
   SEATA_MYSQL_SERVICE_NETWORK=dev_mysql   # In MySQL replica mode, set it to dev_mysql_replica
   SEATA_NACOS_SERVICE_NETWORK=dev_nacos   # The network for Nacos network

   # Seata cluster mode (Using the Registry and db store mode)
   SEATA_CLUSTER_HOME=~/docker/seata-cluster
   SEATA_CLUSTER_NODE1_PORT_8091=18091
   SEATA_CLUSTER_NODE1_HOST=seata-cluster-node1
   SEATA_CLUSTER_NODE2_PORT_8091=28091
   SEATA_CLUSTER_NODE2_HOST=seata-cluster-node2
   SEATA_CLUSTER_NODE3_PORT_8091=38091
   SEATA_CLUSTER_NODE3_HOST=seata-cluster-node3
   ```

4. Make sure you are in the same directory as seata-cluster.yml and start Seata:

   ```shell
   $ docker-compose -f seata-cluster.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Seata](https://hub.docker.com/r/seata/seata-server)
