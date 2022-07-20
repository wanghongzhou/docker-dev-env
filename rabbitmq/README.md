# RabbitMQ

RabbitMQ is the most widely deployed open source message broker. it is lightweight and easy to deploy on premises and in
the cloud. It supports multiple messaging protocols. RabbitMQ can be deployed in distributed and federated
configurations to meet high-scale, high-availability requirements.

---

## Install RabbitMQ standalone mode using Docker Compose

1. Before setting everything else, create a directory for RabbitMQ home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/rabbitmq
   $ touch ~/docker/rabbitmq/conf/custom.conf
   $ chown -R 1001:root ~/docker/rabbitmq
   ```

3. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   RABBITMQ_IMAGE=bitnami/rabbitmq:3.10
   RABBITMQ_USERNAME=admin          # Do not use guest, guest can only log in via localhost
   RABBITMQ_PASSWORD=admin
   RABBITMQ_PLUGINS=
   RABBITMQ_COMMUNITY_PLUGINS=
   
   # rabbitmq standalone mode
   RABBITMQ_HOME=~/docker/rabbitmq
   RABBITMQ_NODE_TYPE=stats         # Valid values: stats, queue-ram or queue-disc.
   RABBITMQ_PORT_5671=5671
   RABBITMQ_PORT_5672=5672
   RABBITMQ_PORT_15672=15672
   ```

4. Make sure you are in the same directory as rabbitmq.yml and start RabbitMQ:

   ```shell
   $ docker-compose -f rabbitmq.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami RabbitMQ](https://hub.docker.com/r/bitnami/rabbitmq)

## Install RabbitMQ cluster mode using Docker Compose

1. Before setting everything else, create a directory for RabbitMQ home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/rabbitmq-cluster/{node1,node2,node3}/conf
   $ touch ~/docker/rabbitmq-cluster/{node1,node2,node3}/conf/custom.conf
   $ chown -R 1001:root ~/docker/rabbitmq-cluster
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   RABBITMQ_IMAGE=bitnami/rabbitmq:3.10
   RABBITMQ_USERNAME=admin          # Do not use guest, guest can only log in via localhost
   RABBITMQ_PASSWORD=admin
   RABBITMQ_PLUGINS=
   RABBITMQ_COMMUNITY_PLUGINS=
   
   # rabbitmq cluster mode
   RABBITMQ_CLUSTER_HOME=~/docker/rabbitmq-cluster
   RABBITMQ_CLUSTER_ERL_COOKIE=s3cr3tc00ki3
   
   RABBITMQ_CLUSTER_NODE1_NODE_NAME=rabbit@rabbitmq-cluster-1
   RABBITMQ_CLUSTER_NODE1_NODE_TYPE=stats
   RABBITMQ_CLUSTER_NODE1_PORT_5671=5611
   RABBITMQ_CLUSTER_NODE1_PORT_5672=5612
   RABBITMQ_CLUSTER_NODE1_PORT_15672=15612
   
   RABBITMQ_CLUSTER_NODE2_NODE_NAME=rabbit@rabbitmq-cluster-2
   RABBITMQ_CLUSTER_NODE2_NODE_TYPE=queue-ram
   RABBITMQ_CLUSTER_NODE2_PORT_5671=5621
   RABBITMQ_CLUSTER_NODE2_PORT_5672=5622
   RABBITMQ_CLUSTER_NODE2_PORT_15672=15622
   
   RABBITMQ_CLUSTER_NODE3_NODE_NAME=rabbit@rabbitmq-cluster-3
   RABBITMQ_CLUSTER_NODE3_NODE_TYPE=queue-disc
   RABBITMQ_CLUSTER_NODE3_PORT_5671=5631
   RABBITMQ_CLUSTER_NODE3_PORT_5672=5632
   RABBITMQ_CLUSTER_NODE3_PORT_15672=15632
   ```

3. Make sure you are in the same directory as rabbitmq-cluster.yml and start RabbitMQ:

   ```shell
   $ docker-compose -f rabbitmq-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami RabbitMQ](https://hub.docker.com/r/bitnami/rabbitmq)
