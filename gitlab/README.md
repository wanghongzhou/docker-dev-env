# Gitlab 
GitLab is a management platform for Git repositories that provides integrated features like continuous integration, issue tracking, team support, and wiki documentation.

---

## Install GitLab using Docker Compose
- The GitLab Docker images are monolithic images of GitLab running all the necessary services in a single container.
- Before you install GitLab, be sure to review the [system requirements](https://docs.gitlab.com/ee/install/requirements.html). 
The system requirements include details about the **minimum hardware** to support GitLab.

1. Before setting everything else, create a directory where the configuration, logs, and data files will reside. Ensure that the directory exists and appropriate permission have been granted.
```shell
$ mkdir ~/docker/gitlab
``` 

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.
```properties 
GITLAB_HOME=/root/docker/gitlab  # Gitlab mount path, the GitLab container uses host mounted volumes to store persistent data.
GITLAB_IMAGE=gitlab/gitlab-ce:14.10.2-ce.0
GITLAB_HOSTNAME=gitlab.example.com  # Domain name or public IP address.
GITLAB_PORT_22=22   # SSH port, port 22 cannot be used because it conflicts with the host.
GITLAB_PORT_80=80   # Nginx http port.
GITLAB_PORT_443=443 # Nginx https port.
GITLAB_SHM_SIZE=256m
```

3. Make sure you are in the same directory as docker-compose.yml and start GitLab:
```shell 
$ sudo docker-compose up -d
```

4. Modify the `${GITLAB_HOME}/config/gitlab.rb` file according to your requirements.
```properties 
external_url 'http://gitlab.example.com'
gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
gitlab_rails['gitlab_shell_ssh_port'] = 22
```

5. Reboot gitlab container
```shell
$ sudo docker-compose restart
```

6. Visit the GitLab URL, and log in with username `root` and the password from the following command:
```shell 
$ sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

> ***The password file will be automatically deleted in the first reconfigure run after 24 hours.***

7. If something else goes wrong, for more detailed tutorial can be found on the [GitLab Website](https://docs.gitlab.com/ee/install/docker.html)