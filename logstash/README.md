# Logstash

Logstash is a light-weight, open-source, server-side data processing pipeline that allows you to collect data from a
variety of sources, transform it on the fly, and send it to your desired destination. It is most often used as a data
pipeline for Elasticsearch, an open-source analytics and search engine.

---

## Install Logstash standalone mode using Docker Compose

1. Before setting everything else, create a directory for Logstash home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/logstash
   $ chown -R 1001:root ~/docker/logstash
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # logstash
   LOGSTASH_IMAGE=bitnami/logstash:8.3.2
   LOGSTASH_HOME=~/docker/kibana
   LOGSTASH_HEAP_SIZE=512m
   LOGSTASH_EXPOSE_API=yes
   LOGSTASH_API_PORT_NUMBER=9090
   LOGSTASH_ENABLE_BEATS_INPUT
   LOGSTASH_BEATS_PORT_NUMBER
   LOGSTASH_ENABLE_GELF_INPUT
   LOGSTASH_GELF_PORT_NUMBER
   LOGSTASH_ENABLE_HTTP_INPUT=true
   LOGSTASH_HTTP_PORT_NUMBER=8080
   LOGSTASH_ENABLE_TCP_INPUT
   LOGSTASH_TCP_PORT_NUMBER
   LOGSTASH_ENABLE_UDP_INPUT
   LOGSTASH_UDP_PORT_NUMBER
   LOGSTASH_ENABLE_STDOUT_OUTPUT
   LOGSTASH_ENABLE_ELASTICSEARCH_OUTPUT=true
   LOGSTASH_ELASTICSEARCH_HOST=elasticsearch
   LOGSTASH_ELASTICSEARCH_PORT_NUMBER=9200
   LOGSTASH_ELASTICSEARCH_NETWORK=dev_elasticsearch
   ```

3. Make sure you are in the same directory as logstash.yml and start Logstash:

   ```shell
   $ docker-compose -f logstash.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Logstash](https://hub.docker.com/r/bitnami/logstash)
