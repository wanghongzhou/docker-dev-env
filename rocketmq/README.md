# RocketMQ

Apache RocketMQ is a distributed messaging and streaming platform with low latency, high performance and reliability, trillion-level capacity and flexible scalability.

---

## Install RocketMQ Single Master Mode Using Docker Compose

1. Before setting everything else, create a directory for RocketMQ home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/rocketmq/{namesrv,dashboard/logs,broker/{logs,store}}
   $ chown -R 3000 ~/docker/rocketmq
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ROCKETMQ_IMAGE=apache/rocketmq:4.9.4
   ROCKETMQ_DASHBOARD_IMAGE=apacherocketmq/rocketmq-dashboard:1.0.0
   ROCKETMQ_JAVA_OPT_EXT=-server -Xms256m -Xmx512m -Xmn128m
   
   # RocketMQ Single Master mode
   ROCKETMQ_HOME=~/docker/rocketmq
   ROCKETMQ_NAMESRV_PORT_9876=9876
   ROCKETMQ_BROKER_PORT_10909=10909
   ROCKETMQ_BROKER_PORT_10911=10911
   ROCKETMQ_BROKER_PORT_10912=10912
   ROCKETMQ_DASHBOARD_PORT_8080=8080
   ```

3. Make sure you are in the same directory as rocketmq.yml and start RocketMQ:

   ```shell
   $ docker-compose -f rocketmq.yml up -d
   ```

4. Configure the broker IP address. By default, the network adapter address is read, namesrv publishes the container address. Of course, you can skip this step if your program is also in the same container network.
   
   ```shell
   $ docker exec -it rocketmq-broker sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   ```
   
5. If something else goes wrong, for more detailed tutorial can be found on the [RocketMQ](https://rocketmq.apache.org/)

## Install RocketMQ Multi Master Mode Using Docker Compose

1. Before setting everything else, create a directory for RocketMQ home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/rocketmq-2m-0s/{namesrv,dashboard/logs,{broker1,broker2}/{logs,store}}
   $ chown -R 3000 ~/docker/rocketmq-2m-0s
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ROCKETMQ_IMAGE=apache/rocketmq:4.9.4
   ROCKETMQ_DASHBOARD_IMAGE=apacherocketmq/rocketmq-dashboard:1.0.0
   ROCKETMQ_JAVA_OPT_EXT=-server -Xms256m -Xmx512m -Xmn128m
   
   # RocketMQ Multi Master mode
   ROCKETMQ_2M_0S_HOME=~/docker/rocketmq-2m-0s
   ROCKETMQ_2M_0S_NAMESRV_PORT_9876=19876
   ROCKETMQ_2M_0S_BROKER1_PORT_10909=21909
   ROCKETMQ_2M_0S_BROKER1_PORT_10911=21911
   ROCKETMQ_2M_0S_BROKER1_PORT_10912=21912
   ROCKETMQ_2M_0S_BROKER2_PORT_10909=22909
   ROCKETMQ_2M_0S_BROKER2_PORT_10911=22911
   ROCKETMQ_2M_0S_BROKER2_PORT_10912=22912
   ROCKETMQ_2M_0S_DASHBOARD_PORT_8080=8080
   ```

3. Make sure you are in the same directory as rocketmq-2m-noslave.yml and start RocketMQ:

   ```shell
   $ docker-compose -f rocketmq-2m-noslave.yml up -d
   ```

4. Configure the broker IP address. By default, the network adapter address is read, namesrv publishes the container address. Of course, you can skip this step if your program is also in the same container network.
   
   ```shell
   $ docker exec -it rocketmq-2m-0s-broker1 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-0s-broker2 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   ```
   
5. If something else goes wrong, for more detailed tutorial can be found on the [RocketMQ](https://rocketmq.apache.org/)

## Install RocketMQ Multi Master Multi Slave Mode Using Docker Compose - sync replication

1. Before setting everything else, create a directory for RocketMQ home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/rocketmq-2m-2s-sync/{namesrv,dashboard/logs,{broker1,broker2,broker3,broker4}/{logs,store}}
   $ chown -R 3000 ~/docker/rocketmq-2m-2s-sync
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ROCKETMQ_IMAGE=apache/rocketmq:4.9.4
   ROCKETMQ_DASHBOARD_IMAGE=apacherocketmq/rocketmq-dashboard:1.0.0
   ROCKETMQ_JAVA_OPT_EXT=-server -Xms256m -Xmx512m -Xmn128m
   
   # RocketMQ Multi Master Multi Slave mode - sync replication
   ROCKETMQ_2M_2S_SYNC_HOME=~/docker/rocketmq-2m-2s-sync
   ROCKETMQ_2M_2S_SYNC_NAMESRV_PORT_9876=19876
   ROCKETMQ_2M_2S_SYNC_BROKER1_PORT_10909=31909
   ROCKETMQ_2M_2S_SYNC_BROKER1_PORT_10911=31911
   ROCKETMQ_2M_2S_SYNC_BROKER1_PORT_10912=31912
   ROCKETMQ_2M_2S_SYNC_BROKER2_PORT_10909=32909
   ROCKETMQ_2M_2S_SYNC_BROKER2_PORT_10911=32911
   ROCKETMQ_2M_2S_SYNC_BROKER2_PORT_10912=32912
   ROCKETMQ_2M_2S_SYNC_BROKER3_PORT_10909=33909
   ROCKETMQ_2M_2S_SYNC_BROKER3_PORT_10911=33911
   ROCKETMQ_2M_2S_SYNC_BROKER3_PORT_10912=33912
   ROCKETMQ_2M_2S_SYNC_BROKER4_PORT_10909=34909
   ROCKETMQ_2M_2S_SYNC_BROKER4_PORT_10911=34911
   ROCKETMQ_2M_2S_SYNC_BROKER4_PORT_10912=34912
   ROCKETMQ_2M_2S_SYNC_DASHBOARD_PORT_8080=8080
   ```

3. Make sure you are in the same directory as rocketmq-2m-2s-sync.yml and start RocketMQ:

   ```shell
   $ docker-compose -f rocketmq-2m-2s-sync.yml up -d
   ```

4. Configure the broker IP address. By default, the network adapter address is read, namesrv publishes the container address. Of course, you can skip this step if your program is also in the same container network.
   
   ```shell
   $ docker exec -it rocketmq-2m-2s-sync-broker1 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-sync-broker2 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-sync-broker3 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-sync-broker4 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   ```
   
5. If something else goes wrong, for more detailed tutorial can be found on the [RocketMQ](https://rocketmq.apache.org/)

## Install RocketMQ Multi Master Multi Slave Mode Using Docker Compose - async replication

1. Before setting everything else, create a directory for RocketMQ home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/rocketmq-2m-2s-async/{namesrv,dashboard/logs,{broker1,broker2,broker3,broker4}/{logs,store}}
   $ chown -R 3000 ~/docker/rocketmq-2m-2s-async
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   ROCKETMQ_IMAGE=apache/rocketmq:4.9.4
   ROCKETMQ_DASHBOARD_IMAGE=apacherocketmq/rocketmq-dashboard:1.0.0
   ROCKETMQ_JAVA_OPT_EXT=-server -Xms256m -Xmx512m -Xmn128m
   
   # RocketMQ Multi Master Multi Slave mode - async replication
   ROCKETMQ_2M_2S_ASYNC_HOME=~/docker/rocketmq-2m-2s-async
   ROCKETMQ_2M_2S_ASYNC_NAMESRV_PORT_9876=39876
   ROCKETMQ_2M_2S_ASYNC_BROKER1_PORT_10909=41909
   ROCKETMQ_2M_2S_ASYNC_BROKER1_PORT_10911=41911
   ROCKETMQ_2M_2S_ASYNC_BROKER1_PORT_10912=41912
   ROCKETMQ_2M_2S_ASYNC_BROKER2_PORT_10909=42909
   ROCKETMQ_2M_2S_ASYNC_BROKER2_PORT_10911=42911
   ROCKETMQ_2M_2S_ASYNC_BROKER2_PORT_10912=42912
   ROCKETMQ_2M_2S_ASYNC_BROKER3_PORT_10909=43909
   ROCKETMQ_2M_2S_ASYNC_BROKER3_PORT_10911=43911
   ROCKETMQ_2M_2S_ASYNC_BROKER3_PORT_10912=43912
   ROCKETMQ_2M_2S_ASYNC_BROKER4_PORT_10909=44909
   ROCKETMQ_2M_2S_ASYNC_BROKER4_PORT_10911=44911
   ROCKETMQ_2M_2S_ASYNC_BROKER4_PORT_10912=44912
   ROCKETMQ_2M_2S_ASYNC_DASHBOARD_PORT_8080=8080
   ```

3. Make sure you are in the same directory as rocketmq-2m-2s-async.yml and start RocketMQ:

   ```shell
   $ docker-compose -f rocketmq-2m-2s-async.yml up -d
   ```

4. Configure the broker IP address. By default, the network adapter address is read, namesrv publishes the container address. Of course, you can skip this step if your program is also in the same container network.
   
   ```shell
   $ docker exec -it rocketmq-2m-2s-async-broker1 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-async-broker2 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-async-broker3 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   $ docker exec -it rocketmq-2m-2s-async-broker4 sed -i '$abrokerIP1=your ip' ../conf/broker.conf
   ```
   
5. If something else goes wrong, for more detailed tutorial can be found on the [RocketMQ](https://rocketmq.apache.org/)