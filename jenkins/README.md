# Jenkins

Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to
building, testing, and delivering or deploying software.

---

## Install Jenkins using Docker Compose

- Minimum hardware requirements:
    - 256 MB of RAM
    - 1 GB of drive space (although 10 GB is a recommended minimum if running Jenkins as a Docker container)

- Recommended hardware configuration for a small team:
    - 4 GB+ of RAM
    - 50 GB+ of drive space

1. Before setting everything else, create a directory for Jenkins home mount. Ensure that the directory exists and
   appropriate permission have been granted.

```shell 
$ mkdir ~/docker/jenkins
$ chown -R 1000:root ~/docker/jenkins
``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

```properties 
JENKINS_HOME=/root/docker/jenkins # Jenkins home path 
JENKINS_IMAGE=jenkins/jenkins:lts-jdk11
JENKINS_PORT_8080=8080   # Jenkins http port.
JENKINS_PORT_50000=50000 # Jenkins agent port.
```

3. Make sure you are in the same directory as docker-compose.yml and start Jenkins:

```shell 
$ docker-compose up -d
```

4. Visit the Jenkins URL, Please use the following password to proceed to installation:

```shell 
$ cat /root/docker/jenkins/secrets/initialAdminPassword 
```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [Jenkins](https://github.com/jenkinsci/docker)