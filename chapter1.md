# Gitlab runner

##### 安装主机：unbuntu 16.04

##### 目前支持：

* Debian
* Ubuntu
* RHEL
* CentOS.

##### 安装步骤：

1.

```
curl -sSL https://get.docker.com/ | sh
```

2.Add GitLab's official repository:

```
# For Debian/Ubuntu
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash

# For RHEL/CentOS
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
```

3.Install`gitlab-ci-multi-runner`

```
# For Debian/Ubuntu
sudo apt-get install gitlab-ci-multi-runner

# For RHEL/CentOS
sudo yum install gitlab-ci-multi-runner
```

4.Register the Runner

```
sudo gitlab-ci-multi-runner register
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com

Please enter the gitlab-ci token for this runner
xxx

Please enter the gitlab-ci description for this runner
my-runner

INFO[0034] fcf5c619 Registering runner... succeeded

Please enter the executor: shell, docker, docker-ssh, ssh?
docker

Please enter the Docker image (eg. ruby:2.1):
ruby:2.1

INFO[0037] Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

#### 更新runner

```
# For Debian/Ubuntu
sudo apt-get update
sudo apt-get install gitlab-ci-multi-runner


# For RHEL/CentOS

sudo yum update
sudo yum install gitlab-ci-multi-runner
```



