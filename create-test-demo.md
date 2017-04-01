# Create Test Demo

* ##### 创建.gitlab-ci.yml文件到 项目 根目录

例如：

```yml
#select image from https://hub.docker.com/_/php/
image: registry.saas.hand-china.com/tools/php:5.6
# Select what we should cache
cache:
  paths:
  - vendor/
before_script:
# Install git, the php image doesn't have installed
- apt-get update -yqq
- apt-get install git -yqq
test:
  script:
  - php -v
build:
  script:
  - php -i
```

* ##### 上传到项目仓库

```git
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master
```

上传之后在 project-&gt;pipelines下会有pending的一个构建：![](/assets/WX20170401-140516@2x.png)

* ##### 配置运行的gitlab-runner

##### ![](/assets/WX20170401-141854@2x.png)

######    1. 在运行runner的主机上注册一个runner\(此处是运行 Specific Runners\)

```
sudo gitlab-runner register
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.59.103:10080/ci
Please enter the gitlab-ci token for this runner:
PdMd2jCCHwqHftxu6y7x
Please enter the gitlab-ci description for this runner:
[gitlab]: php
Please enter the gitlab-ci tags for this runner (comma separated):

Whether to lock Runner to current project [true/false]:
[false]: 
Registering runner... succeeded                     runner=PdMd2jCC
Please enter the executor: virtualbox, docker-ssh+machine, kubernetes, docker, parallels, shell, ssh, docker+machine, docker-ssh:
docker
Please enter the default Docker image (e.g. ruby:2.1):
php:5.6
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

配置好如图：![](/assets/WX20170401-142717@2x.png)

######    2. 配置成功Runner后，最后一次的提交padding状态就会变成running , faild , success 状态。

![](/assets/WX20170401-142928@2x.png)

######   3. 点击查看任务详情可以查看运行的终端log和jobs的失败成功原因。

###### ![](/assets/WX20170401-143343@2x.png)

######    4 . 完成一次简单的构建end.



