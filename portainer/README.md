# Portainer

Portainer consists of two elements, the Portainer Server, and the Portainer Agent. Both elements run as lightweight Docker containers on a Docker engine. This document will help you install the Portainer Server container on your Linux environment.

---

## Install Portainer Using Docker Compose

1. Before setting everything else, create a directory where the configuration files will reside. Ensure that the directory exists and appropriate permission have been granted.
   
   ```shell
   $ mkdir -vp ~/docker/portainer/data
   ```
   
2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   PORTAINER_IMAGE=portainer/portainer-ce:2.14.2
   PORTAINER_HOME=~/docker/portainer
   PORTAINER_PORT_8000=8000
   PORTAINER_PORT_9443=9443
   ```

3. Make sure you are in the same directory as docker-compose.yml and start Portainer:

   ```shell
   $ docker-compose up -d
   ```

4. If something else goes wrong, for more detailed tutorial can be found on the [Portainer Website](https://docs.portainer.io/v/ce-2.9/start/install)