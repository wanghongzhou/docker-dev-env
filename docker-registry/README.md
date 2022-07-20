# Docker Registry

The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images.
The Registry is open-source, under the permissive Apache license.

---

## Install Docker Registry using Docker Compose

1. Before setting everything else, create a directory where the configuration files will reside. Ensure that the
   directory exists and appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/docker-registry/{auth,certs,registry}
   ```

2. Copy the .crt and .key files from the CA into the certs directory. The following steps assume that the files are
   named domain.crt and domain.key

3. Create a password file with one entry for the user testuser, with password testpassword:

   ```shell
   $ docker run --rm --entrypoint htpasswd httpd:2 -Bbn testuser testpassword > ~/docker/docker-registry/auth/htpasswd
   ```
   
4. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   DOCKER_REGISTRY_IMAGE=registry:2
   DOCKER_REGISTRY_HOME=~/docker/docker-registry
   DOCKER_REGISTRY_PORT_5000=5000
   DOCKER_REGISTRY_AUTH=htpasswd
   DOCKER_REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
   DOCKER_REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
   DOCKER_REGISTRY_HTTP_TLS_KEY=/certs/domain.key
   DOCKER_REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
   ```

5. Make sure you are in the same directory as docker-compose.yml and start Docker Registry:

   ```shell
   $ docker-compose up -d
   ```
   
6. Log in to the registry.

   ```shell
   $ docker login myregistrydomain.com:5000
   ```

7. If something else goes wrong, for more detailed tutorial can be found on
   the [Docker Registry Website](https://docs.docker.com/registry/)