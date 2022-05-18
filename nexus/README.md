# Nexus

---

## Install Nexus using Docker Compose
1. Create a directory for nexus home mount
```shell
$ mkdir ~/docker/nexus 
$ chown -R 200 ~/docker/nexus
``` 

2. Modify the `.env` file according to your requirements.
```properties 
NEXUS_HOME=/root/docker/nexus
NEXUS_IMAGE=sonatype/nexus3:3.38.1
NEXUS_PORT_8081=8081  # Nexus http port 
NEXUS_INSTALL4J_ADD_VM_PARAMS="-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m" # Jvm Options
```

3. Make sure you are in the same directory as docker-compose.yml and start Nexus:
```shell 
$ sudo docker-compose up -d
```

4. Visit the Nexus URL,  and log in with username `admin` and the password from the following command:
```shell 
$ cat /root/docker/nexus/admin.password
```

5. A more detailed tutorial can be found on the [Nexus](https://help.sonatype.com/repomanager3)