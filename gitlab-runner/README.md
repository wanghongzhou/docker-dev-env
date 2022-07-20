# GitLab Runner

GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.

---

## Install GitLab Runner using Docker Compose

1. Before setting everything else, create a directory where the configuration files will reside. Ensure that the
   directory exists and appropriate permission have been granted.

   ```shell
   $ mkdir -vp ~/docker/gitlab-runner
   ```

2. Modify the `.env` file, you can fine tune these configurations to meet your requirements.

   ```properties
   GITLAB_RUNNER_IMAGE=gitlab/gitlab-runner:v15.1.1
   GITLAB_RUNNER_HOME=~/docker/gitlab-runner
   ```

3. Make sure you are in the same directory as docker-compose.yml and start GitLab Runner:

   ```shell
   $ docker-compose up -d
   ```
4. The final step is to register a new runner. The GitLab Runner container doesn’t pick up any jobs until it’s
   registered

   ```shell
   $ docker exec -it gitlab-runner gitlab-runner register   
   ```

5. If something else goes wrong, for more detailed tutorial can be found on
   the [GitLab Runner Website](https://docs.gitlab.com/runner/install/docker.html)