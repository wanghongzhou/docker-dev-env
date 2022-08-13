# MySQL

MySQL is an open source SQL relational database management system that’s developed and supported by Oracle.

---

## Install MySQL Standalone Mode Using Docker Compose

1. Before setting everything else, create a directory for MySQL home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/mysql/{data,conf,initdb.d}
   $ touch ~/docker/mysql/conf/my_custom.cnf
   $ chown -R 1001:root ~/docker/mysql
   ```
   
2. Modify the `~/docker/mysql/conf/my_custom.cnf` file according to your requirements.

   ```properties
   [mysqld]
   character-set-server=utf8mb4
   collation-server=utf8mb4_general_ci
   default-storage-engine=INNODB
   sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
   default-time-zone='+8:00'
   max_connections=2048
   skip-host-cache
   skip-name-resolve
   log-bin=mysql-bin
   binlog_format=mixed
   
   [mysql]
   default-character-set=utf8mb4
   
   [client]
   port=3306
   ```

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MYSQL_IMAGE=bitnami/mysql:8.0
   MYSQL_CHARACTER_SET=utf8mb4
   MYSQL_COLLATE=utf8mb4_general_ci
   MYSQL_ROOT_PASSWORD=123456
   MYSQL_AUTHENTICATION_PLUGIN=mysql_native_password
   
   # mysql standalone mode
   MYSQL_HOME=~/docker/mysql
   MYSQL_PORT_3306=3306
   ```

4. Make sure you are in the same directory as mysql.yml and start MySQL:

   ```shell
   $ docker-compose -f mysql.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami MySQL](https://hub.docker.com/r/bitnami/mysql)

## Install MySQL Replication Mode Using Docker Compose

1. Before setting everything else, create a directory for MySQL home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/mysql-replica/{master/{data,conf,initdb.d},{slave-1,slave-2,slave-3}/{data,conf}}
   $ touch ~/docker/mysql-replica/{master,slave-1,slave-2,slave-3}/conf/my_custom.cnf
   $ chown -R 1001:root ~/docker/mysql-replica
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MYSQL_IMAGE=bitnami/mysql:8.0
   MYSQL_CHARACTER_SET=utf8mb4
   MYSQL_COLLATE=utf8mb4_general_ci
   MYSQL_ROOT_PASSWORD=123456
   MYSQL_AUTHENTICATION_PLUGIN=mysql_native_password
   
   # mysql cluster mode
   MYSQL_REPLICA_HOME=~/docker/mysql-replica
   MYSQL_REPLICA_MASTER_PORT_3306=3307
   MYSQL_REPLICA_SLAVE1_PORT_3306=3308
   MYSQL_REPLICA_SLAVE2_PORT_3306=3309
   MYSQL_REPLICA_SLAVE3_PORT_3306=3310
   ```

3. Make sure you are in the same directory as mysql-replica.yml and start MySQL:

   ```shell
   $ docker-compose -f mysql-replica.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami MySQL](https://hub.docker.com/r/bitnami/mysql)
