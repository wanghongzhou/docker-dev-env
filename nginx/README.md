# Nginx

NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more. It
started out as a web server designed for maximum performance and stability. In addition to its HTTP server capabilities,
NGINX can also function as a proxy server for email (IMAP, POP3, and SMTP) and a reverse proxy and load balancer for
HTTP, TCP, and UDP servers.

---

## Install Nginx using Docker Compose

1. Before setting everything else, create a directory for Nginx home mount. Ensure that the directory exists and
   appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/nginx/{data,certs,server_blocks}
   $ chown -R 1001:root ~/docker/nginx
   ```

4. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   NGINX_IMAGE=bitnami/minio:2022.5.19
   NGINX_HOME=~/docker/nginx
   NGINX_PORT_8080=80
   ```

4. Make sure you are in the same directory as nginx.yml and start Nginx:

   ```shell
   $ docker-compose -f nginx.yml up -d
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Bitnami Nginx](https://hub.docker.com/r/bitnami/nginx)