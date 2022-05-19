# GitLab Runner 
GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.

---

## Install GitLab Runner using Docker Compose

1. Before setting everything else, create a directory where the configuration files will reside. Ensure that the directory exists and appropriate permission have been granted.
```shell
$ mkdir ~/docker/gitlab-runner
``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.
```properties 
GITLAB_RUNNER_HOME=/root/docker/gitlab-runner
GITLAB_RUNNER_IMAGE=gitlab/gitlab-runner:v14.10.1
```

3. Make sure you are in the same directory as docker-compose.yml and start GitLab:
```shell 
$ docker-compose up -d
```

4. If something else goes wrong, for more detailed tutorial can be found on the [GitLab Runner Website](https://docs.gitlab.com/runner/install/docker.html)