# Gitlab 
The GitLab Docker images are monolithic images of GitLab running all the necessary services in a single container.

---

## Install GitLab using Docker Compose
1. Modify the `.env` file according to your requirements.
```properties 
GITLAB_HOME=/root/docker/gitlab  # Gitlab home path, the GitLab container uses host mounted volumes to store persistent data.
GITLAB_IMAGE=gitlab/gitlab-ce:14.10.2-ce.0
GITLAB_HOSTNAME=gitlab.example.com  # Domain name or public IP address.
GITLAB_PORT_22=22   # SSH port.
GITLAB_PORT_80=80   # Nginx http port.
GITLAB_PORT_443=443 # Nginx https port.
GITLAB_SHM_SIZE=256m
```

2. Make sure you are in the same directory as docker-compose.yml and start GitLab:
```shell 
$ sudo docker-compose up -d
```

3. Modify the `${GITLAB_HOME}/config/gitlab.rb` file according to your requirements.
```properties 
external_url 'http://gitlab.example.com'
gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
gitlab_rails['gitlab_shell_ssh_port'] = 22
```

4. Reboot gitlab
```shell
$ sudo docker-compose down
$ sudo docker-compose up -d
```

5. Visit the GitLab URL, and log in with username root and the password from the following command:
```shell 
$ sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

> ***The password file will be automatically deleted in the first reconfigure run after 24 hours.***

6. A more detailed tutorial can be found on the [GitLab Website](https://docs.gitlab.com/ee/install/docker.html)