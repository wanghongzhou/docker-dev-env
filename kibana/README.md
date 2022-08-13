# Kibana

Kibana is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. Users can create bar, line and scatter plots, or pie charts and maps on top of large volumes of data.

---

## Install Kibana Using Docker Compose

1. Before setting everything else, create a directory for Kibana home mount. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/kibana/
   $ chown -R 1001:root ~/docker/kibana
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   # kibana
   KIBANA_IMAGE=bitnami/kibana:8.3.3
   KIBANA_HOME=~/docker/kibana
   KIBANA_PORT_5601=5601
   KIBANA_ELASTICSEARCH_NETWORK=dev_elasticsearch
   KIBANA_ELASTICSEARCH_URL=elasticsearch
   KIBANA_ELASTICSEARCH_PORT_NUMBER=9200
   ```

3. Make sure you are in the same directory as kibana.yml and start Kibana:

   ```shell
   $ docker-compose -f kibana.yml up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Bitnami Kibana](https://hub.docker.com/r/bitnami/kibana)