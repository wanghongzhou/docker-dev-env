# Jenkins
The Jenkins Continuous Integration and Delivery server available on [Docker Hub](https://hub.docker.com/r/jenkins/jenkins).

---

## Install Jenkins using Docker Compose
1. Create a directory for Jenkins home mount
```shell
$ mkdir ~/docker/jenkins
$ chown 1000:root ~/docker/jenkins
``` 

2. Modify the `.env` file according to your requirements.
```properties 
JENKINS_HOME=/root/docker/jenkins # Jenkins home path 
JENKINS_IMAGE=jenkins/jenkins:lts-jdk11
JENKINS_PORT_8080=8080   # Jenkins http port.
JENKINS_PORT_50000=50000 # Jenkins agent port.
```

3. Make sure you are in the same directory as docker-compose.yml and start Jenkins:
```shell 
$ sudo docker-compose up -d
```

4. Visit the Jenkins URL, Please use the following password to proceed to installation:
```shell 
$ cat /root/docker/jenkins/secrets/initialAdminPassword 
```

5. A more detailed tutorial can be found on the [Jenkins](https://github.com/jenkinsci/docker/blob/master/README.md)