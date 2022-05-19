# Nexus

Nexus by Sonatype is a repository manager that organizes, stores and distributes artifacts needed for development. With
Nexus, developers can completely control access to, and deployment of, every artifact in an organization from a single
location, making it easier to distribute software.

---

## Install Nexus using Docker Compose

- Before you install Nexus, be sure to review
  the [system requirements](https://help.sonatype.com/repomanager3/product-information/system-requirements).
- Minimum hardware requirements:
    - 8 GB of RAM
    - 4 cores of CPU
    - -Xms2703M -Xmx2703M -XX:MaxDirectMemorySize=2703M

1. Before setting everything else, create a directory for Nexus home mount. Ensure that the directory exists and
   appropriate permission have been granted.

```shell 
$ mkdir -vp ~/docker/nexus 
$ chown -R 200 ~/docker/nexus
``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

```properties 
NEXUS_HOME=~/docker/nexus
NEXUS_PORT_8081=8081  # Nexus http port 
NEXUS_INSTALL4J_ADD_VM_PARAMS="-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m" # Jvm Options
```

3. Make sure you are in the same directory as docker-compose.yml and start Nexus:

```shell 
$ docker-compose up -d
```

4. Visit the Nexus URL, and log in with username `admin` and the password from the following command:

```shell 
$ cat ~/docker/nexus/admin.password
```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Nexus](https://help.sonatype.com/repomanager3)