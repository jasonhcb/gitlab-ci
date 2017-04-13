# .gitlab-ci.yml配置和参数详解

.gitlab-ci.yml 用来配置项目的CI/CD构建，放在跟目录下，YAML文件使用一些约束条件来定义Jobs该如何工作，比如以下结构：

```
job1:
  script: "execute-script-for-job1"

job2:
  script: "execute-script-for-job2"
```

该例子就是创建两个jobs来创建builds，并且在指定project的runner上运行这些script。更复杂的示例比如：

```
image: ruby:2.1
services:
  - postgres

before_script:
  - bundle install

after_script:
  - rm secrets

stages:
  - build
  - test
  - deploy

job1:
  stage: build
  script:
    - execute-script-for-job1
  only:
    - master
  tags:
    - docker
```

* #### 以下是一些规范针对 `不能`** **作为  job 名称：

  | 关键词 | 必需 | 描述 |
  | :--- | :--- | :--- |
  | image | no | 使用docker image, 已涵盖在[GitLab CI使用Docker](https://docs.gitlab.com.cn/ce/ci/docker/README.html) |
  | services | no | 使用docker services, 已涵盖在[GitLab CI使用Docker](https://docs.gitlab.com.cn/ce/ci/docker/README.html) |
  | stages | no | 定义builds阶段 |
  | types | no | `stages`的别名 |
  | before\_script | no | 定义每个job之前执行的脚本 |
  | after\_script | no | 定义每个job之后执行的脚本 |
  | variables | no | 定义build变量 |
  | cache | no | 定义与后续job之间应缓存的文件 |

* ---

  #### image and services

  这两个关键词允许指定该job需要使用的自定义的Dokcer images和 Docker services。该功能的配置已涵盖在[GitLab CI与Docker的整合](https://docs.gitlab.com.cn/ce/ci/docker/README.html)文档中。

* ---

  #### before\_script

  `before_script`用来定义 所有的jobs开始之前执行的命令， 包括部署任务。他可以是一个数组或多行字符串。

* ---

  #### after\_script

  `after_script`用来定义所有builds完成之后执行的任务，它可以是一个数组或者是多行字符串。

* ---

  #### stages 

  `stages`在jobs中可以用来定义build的各个阶段，`stages`的规范允许灵活的使用多个stage pipelines。

  `stages`中个元素的顺序决定了jobs的执行顺序：  
      1.同一个stage中的元素将并行Builds。  
      2.下一个stage将在上一个stage中所有元素 成功完成后再继续执行job。

  例如如下这个例子：

  ```
  stages:
  - build
  - test
  - deploy
  ```

  1.首先 build 中的jobs将并行执行。  
  2.如果 build 中的jobs都成功执行了, test 中的jobs将并行执行.  
  3.如果 test 中的jobs都成功执行了, deploy 中的jobs将并行执行.  
  4.如果 deploy 中的jobs都成功完成, 该条 commit 将标记为 success.  
  5.如果前面任意一个jobs执行失败，该条 commit 将标记为 failed，并且 下一步的stage中的jobs不会继续下去。

  还有两种特别情况：

  1.如果没有定义任何的`stages`，默认情况下`build`,`test`,`deploy`允许作为预设任务的stage，即stage名为`build`,`test`,`deploy`.  
  2.如果一个job没有执行`stage`,该任务会指派为`test` stage.

* ---

  #### types

  与`stages`



