# GitLab with GitLab Runner by Docker Compose


## Getting Started

### Clone the Repository

Clone this repository using `git clone`

```
$ git clone https://github.com/Pick1a1username/gitlab_with_gitlab-runner_by_docker-compose.git
```

### Create Directories

Create Directories required for apps.

```
$ cd gitlab_with_gitlab-runner_by_docker-compose
$ mkdir -p gitlab/etc/gitlab gitlab/var/log/gitlab gitlab/var/opt/gitlab
$ mkdir -p gitlab-runner/etc/gitlab-runner
```


### Create a Network for Apps

```
$ docker network create gitlab
$ docker network ls
```


### Run the Apps

Run the apps and browse to `http://localhost:8929`.
(It takes minutes to be completely up)

```
$ docker-compose up -d
```

You will be asked to set the password of `root`.


### Register GitLab Runner

Referring to the following link, register the GitLab Runner to GitLab.

* https://docs.gitlab.com/ee/ci/runners/

Note that when you set `your GitLab instance URL` during registring the GitLab Runner, set `http://gitlab`, not `http://localhost:8929`.

You need to get into the GitLab Runner container to register the GitLab Runner.

Example:

```
$ docker ps
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                   PORTS                                                 NAMES
bd2e5136dfbc        gitlab/gitlab-runner:latest    "/usr/bin/dumb-init …"   3 minutes ago       Up 3 minutes                                                                   gitlab_with_gitlab-runner_by_docker-compose_gitlab-runner_1
d8d6189f8c4d        gitlab/gitlab-ce:11.6.2-ce.0   "/assets/wrapper"        3 minutes ago       Up 3 minutes (healthy)   443/tcp, 0.0.0.0:2289->22/tcp, 0.0.0.0:8929->80/tcp   gitlab_with_gitlab-runner_by_docker-compose_gitlab_1
$
$ docker exec -it bd2e5136dfbc bash
root@bd2e5136dfbc:/# ls
bin  boot  dev  entrypoint  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@bd2e5136dfbc:/# gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=34 revision=8d829975 version=11.6.1
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab
Please enter the gitlab-ci token for this runner:
2kyys_7QqPnswWvbb7F9
Please enter the gitlab-ci description for this runner:
[bd2e5136dfbc]:
Please enter the gitlab-ci tags for this runner (comma separated):
docker
Registering runner... succeeded                     runner=2kyys_7Q
Please enter the executor: shell, virtualbox, kubernetes, docker, docker-ssh, parallels, ssh, docker+machine, docker-ssh+machine:
docker
Please enter the default Docker image (e.g. ruby:2.1):
alpine:3
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
root@bd2e5136dfbc:/#
root@bd2e5136dfbc:/#
root@bd2e5136dfbc:/# gitlab-runner start
Runtime platform                                    arch=amd64 os=linux pid=42 revision=8d829975 version=11.6.1
root@bd2e5136dfbc:/#
```


### Configure `config.toml` in the GitLab Runner container.

Since GitLab Runner in this procedure will try to clone repositories through `http://localhost:8929`, you need to specify a variable called `clone_url` in `/etc/gitlab-runner/config.toml`.

After editing `config.toml`, restart the runner service.

Example:

```
root@bd2e5136dfbc:/# vi /etc/gitlab-runner/config.toml
root@bd2e5136dfbc:/#
root@bd2e5136dfbc:/#
root@bd2e5136dfbc:/# cat /etc/gitlab-runner/config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "bd2e5136dfbc"
  url = "http://gitlab"
  clone_url = "http://gitlab"
  token = "UKdGM9rvejkzx31NbwfE"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine:3"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
root@bd2e5136dfbc:/# gitlab-runner restart
Runtime platform                                    arch=amd64 os=linux pid=71 revision=8d829975 version=11.6.1
root@bd2e5136dfbc:/#
```

## Notes

If you set `docker:dind` as a service in `.gitlab-ci.yml`, it may not work properly.
The following tickets are related to this issue:

* https://gitlab.com/gitlab-org/gitlab-runner/issues/2699
* https://gitlab.com/gitlab-org/gitlab-runner/issues/1592

Consider to use docker:dind directly as a image. If you want to use docker:dind directly, you should mount `/var/run/docker.sock` to the container by editing `/etc/gitlab-runner/config.toml` in the runner container.


## References

* https://docs.gitlab.com/omnibus/docker/
* https://docs.gitlab.com/runner/configuration/advanced-configuration.html


## Contributing

Any suggestions are welcome!


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
