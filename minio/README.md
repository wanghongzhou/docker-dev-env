# Minio

MinIO is a High Performance Object Storage released under GNU Affero General Public License v3.0. It is API compatible
with Amazon S3 cloud storage service. It can handle unstructured data such as photos, videos, log files, backups, and
container images with (currently) the maximum supported object size of 5TB.

---

## Install Minio standalone mode using Docker Compose

1. Before setting everything else, create a directory for Minio home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/minio/{data,certs}
   $ chown -R 1001:root ~/docker/minio
   ```

4. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MINIO_IMAGE=bitnami/minio:2022
   MINIO_ROOT_USER=minio
   MINIO_ROOT_PASSWORD=minio123   # The password must be at least eight characters long

   # minio standalone mode
   MINIO_HOME=~/docker/minio
   MINIO_PORT_9000=9000
   MINIO_PORT_9001=9001
   ```

4. Make sure you are in the same directory as minio.yml and start Minio:

   ```shell
   $ docker-compose -f minio.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Minio](https://hub.docker.com/r/bitnami/minio)

## Install Minio cluster mode using Docker Compose

1. Before setting everything else, create a directory for Minio home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/minio-cluster/{node1/{data,certs},node2/{data,certs},node3/{data,certs},node4/{data,certs}}
   $ chown -R 1001:root ~/docker/minio-cluster
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # common
   MINIO_IMAGE=bitnami/minio:2022
   MINIO_ROOT_USER=minio
   MINIO_ROOT_PASSWORD=minio123
   
   # minio cluster mode
   MINIO_CLUSTER_HOME=~/docker/minio-cluster
   MINIO_CLUSTER_NODE1_PORT_9000=19000
   MINIO_CLUSTER_NODE1_PORT_9001=19001
   MINIO_CLUSTER_NODE2_PORT_9000=29000
   MINIO_CLUSTER_NODE2_PORT_9001=29001
   MINIO_CLUSTER_NODE3_PORT_9000=39000
   MINIO_CLUSTER_NODE3_PORT_9001=39001
   MINIO_CLUSTER_NODE4_PORT_9000=49000
   MINIO_CLUSTER_NODE4_PORT_9001=49001
   ```

3. Make sure you are in the same directory as minio-cluster.yml and start Minio:

   ```shell
   $ docker-compose -f minio-cluster.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Minio](https://hub.docker.com/r/bitnami/minio)
