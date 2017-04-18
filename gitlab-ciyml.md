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

#### 以下是一些规范针对 `不能`** **作为  job 名称：

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

---

#### image and services

这两个关键词允许指定该job需要使用的自定义的Dokcer images和 Docker services。该功能的配置已涵盖在[GitLab CI与Docker的整合](https://docs.gitlab.com.cn/ce/ci/docker/README.html)文档中。

---

#### before\_script

`before_script`用来定义 所有的jobs开始之前执行的命令， 包括部署任务。他可以是一个数组或多行字符串。

---

#### after\_script

`after_script`用来定义所有builds完成之后执行的任务，它可以是一个数组或者是多行字符串。

---

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

---

#### types

与`stages`一样，但是在 10.0版本之后就废弃了

---

#### variables

> 该功能自GitLab Runner v0.5.0中引入。

GitLab CI允许在 .gitlab-ci.yml 里面添加变量以便应用到job系统环境中。 变量存储在git仓库中并用于存储non-sensitive（非敏感）的project配置，（尽量不要放置和密钥相关的信息）如下面的例子：

```
  variables:
  DATABASE_URL: "postgres://postgres@postgres/my_database"
```

这个变量可延迟作用于所有已执行的命令和脚本中。 同时通过YAML定义的变量也会设置所有的服务容器， 因此允许微调这些服务容器。 job level也可以定义变量。  
  除了自定义的变量之外，gitlab-runner还提供了一些已经定义的变量对于我们的构建和服务。详细文档请参考：  
  [Learn more about variables.](https://docs.gitlab.com.cn/ce/ci/variables/README.html)

---

#### cache

> 该功能自GitLab Runner v0.7.0引入。

chche 用来指定需要在 builds 之间进行缓存的一组文件、文件夹。 你可以只使用project workspace里面的路径。在Gitlab 9.0已经默认cache 共享在pipelines 和 jobs。如果定义在 jobs 之外就是全局的cache。比如：  
  缓存binaries 目录下的文件和.config文件：

```
  rspec:
  script: test
  cache:
    paths:
    - binaries/
    - .config
```

缓存所有未被Git跟踪的文件：

```
  rspec:
    script: test
    cache:
      untracked: true
```

缓存所有未被Git跟踪的文件以及binaries目录下的文件：

```
  rspec:
  script: test
  cache:
    untracked: true
    paths:
    - binaries/
```

下面这个例子即使定义的全局和局部的区别：

```
  cache:
  paths:
  - my/files

  rspec:
    script: test
    cache:
      key: rspec
      paths:
      - binaries/
```

notice: 如果你使用了共享的cache在jobs中，在不同的jobs使用不同的路径，那么你必须要设置不同的cache:key。不然cache内容就会被重写掉。

##### cache:key 的用法：

> 该功能自 GitLab Runner v1.0.0引入。

这个 key 指令允许定义jobs之间缓存的亲和力， 允许所有的jobs只有单个缓存，也可以是每个per-job caching、 per-branch caching，或者其他你认为合适的方法。

key允许你对缓存进行微调， 也允许在不同Jobs设置不同brances之间缓存数据。

cache:key变量可以使用任何 [预定义变量](https://docs.gitlab.com.cn/ce/ci/variables/README.html)。

例子如下：  
  启用 per-job caching:

```
  cache:
  key: "$CI_JOB_NAME"
  untracked: true
```

启用 per-branch caching:

```
  cache:
  key: "$CI_COMMIT_REF_NAME"
  untracked: true
```

启用 per-job and per-branch caching:

```
  cache:
  key: "$CI_JOB_NAME/$CI_COMMIT_REF_NAME"
  untracked: true
```

启用 per-branch and per-stage caching:

```
  cache:
  key: "$CI_JOB_STAGE/$CI_COMMIT_REF_NAME"
  untracked: true
```

注:  如果你用的是 Windows Batch Windows批处理运行shell scripts， 需要把$替换为%：

```
  cache:
  key: "%CI_JOB_STAGE%/%CI_COMMIT_REF_NAME%"
  untracked: true
```

---

#### Jobs

`.gitlab-ci.yml` 允许你创建无限个jobs，每个job必须有一个独一无二的名字，而且不能是ci的关键字。下面是Job 可定义的一些参数：

```
  job_name:
  script:
    - rake spec
    - coverage
  stage: test
  only:
    - master
  except:
    - develop
  tags:
    - ruby
    - postgres
  allow_failure: true
```

| Keyword | Required | Description |
| :--- | :--- | :--- |
| script | yes | job执行的脚本 |
| image | no | 使用docker-images的作为镜像 [参考使用](https://docs.gitlab.com.cn/ce/ci/docker/using_docker_images.html#define-image-and-services-from-gitlab-ciyml) |
| services | no | 使用dokcer-images作为服务 [参考使用](https://docs.gitlab.com.cn/ce/ci/docker/using_docker_images.html#define-image-and-services-from-gitlab-ciyml) |
| stage | no | 定义一个stages \(default:`test`\) |
| type | no | 别名： `stage` |
| variables | no | 定义job 的变量 |
| only | no | 定义git操作list时 触发job创建 |
| except | no | 定义git操作list时 不触发Job构建 |
| tags | no | 定义用于选择Runner的tags列表 |
| allow\_failure | no | 允许job构建失败. 失败的job不进入提交状态 |
| when | no | 定义什么时候运行job，可能值为：`on_success`,`on_failure`,`always`or`manual` |
| dependencies | no | 定义其他jobs的依赖job，便于构建的传递 |
| artifacts | no | 定义文件 参考：[job artifacts](https://docs.gitlab.com.cn/ce/user/project/pipelines/job_artifacts.html) |
| cache | no | 定义cache缓存 |
| before\_script | no | 重写jobs外定义的before\_script |
| after\_script | no | 重写Jobs外定义的after\_script |
| environment | no | 定义jobs完成的构建的环境 |
| coverage | no | 定义给定jobs的代码覆盖率 |

下面详细介绍以上参数的含义：

---

##### script

`script`是Runner执行的shell脚本，比如：

```
  job:
  script: "bundle exec rspec"
```

此参数还可以包含使用数组形式：

```
  job:
  script:
    - uname -a
    - bundle exec rspec
```

有时候，`script`命令需要用单引号或者双引号括起来，例如，包含冒号（`：`）的命令需要用引号括起来，以便YAML解析的时候知道这不是key：value的形似，一下字符需要小心：`:`,`{`,`}`,`[`,`]`,`,`,`&`,`*`,`#`,`?`,`|`,`-`,`<`,`>`,`=`,`!`,`%`,`@`,\`\`\`\`\`.

---

##### stage

`stage`允许将jobs分组到不同的`stage`状态下，相同的`stage`将被平行执行，更多使用参考本文前面的`stages`语法。

##### only and except
`only`和 `except`是设置限制作业策略构建的两个参数：
  1.`only` 定义了jobs要运行的branch和tags的名称。
  2. `except`定义了jobs不运行哪些branch 和tags的名称。
  下面的几点规则是适用于两者的：
  * `only`和`except`是可以同时存在的，而且都会过滤两者的条件。
  * `only`和`except`允许使用正则表达式。
  * `only`和`except`允许使用特定的关键字：`branches`,`tags`,`triggers`。
  * `only`和`except`允许使用特殊的项目存储路径来作为过滤条件。

以下的例子是`job` 只针对以正则 `issue-`开头的参数，而所有的分支会被跳过。
```
job:
  # use regexp
  only:
    - /^issue-.*$/
  # use special keyword
  except:
    - branches
```
下面的例子是`job` 只对标记的标签起作用，或者通过 API 触发器请求构建。
```
job:
  # use special keywords
  only:
    - tags
    - triggers
```
项目存储路径为过滤条件的jobs将会解释父项目的构建而不会执行forks的构建，下面这个例子就是`job`除去`master`之外所有分支都会触发构建。
```
job:
  only:
    - branches@gitlab-org/gitlab-ce
  except:
    - master@gitlab-org/gitlab-ce
```
---
##### job variables
定义变量给Job级别，和全局变量的定义规则一样，但是可以定义专门针对于此job的`variables`,如果job中去定义全局中定义的关键字时，会覆盖YAML job的变量和预处理变量，如果要在job关闭使用的全局变量的时候，就定义一个空数组。如下：
```
job_name:
  variables: []
```
详细参考：[variables 文档](https://docs.gitlab.com.cn/ce/ci/variables/README.html)

---
##### tags
`tags`应用于从允许运行此项目中选择 特定的Runner，在注册一个Runner的时候，你会给这个特定的标签，例如：`ruby`,`postgres`,`development`.

`tags`允许你取运行job在有特定指定标签的Runner：如下就是job在ruby和postgres标签的Runner上构建。
```
job:
  tags:
    - ruby
    - postgres
```
---
##### allow_failure
`allow_failure`当你允许job失败的时候，而不会影响其他CI套件的使用，失败的job不会影响提交状态。

当允许 job 失败时，pipeline所有的结果将会 变成成功/绿色的状态，但是在提交合并请求或者是提交的时候在job页面就会有一个警告“CI build passed with warning”。允许job 失败的设置后，如果失败那么需要在其他地方下采取一些（手动）操作步骤。

下面这个例子，job1和job2将会运行，如果job1失败了，但是他不会停止下一个stage的运行，一旦设定了
`allow_failure: true`:
```
job1:
  stage: test
  script:
  - execute_script_that_will_fail
  allow_failure: true

job2:
  stage: test
  script:
  - execute_script_that_will_succeed

job3:
  stage: deploy
  script:
  - deploy_to_staging
```

---
##### when
`when`适用于在出现故障或者裕兴失败的job时候的构建。
`when`的设置值为：
1.`on_success` - 只有当前一个阶段的所有工作成功时才​​执行工作。这是默认值。
2.`on_failure` - 仅当前一个阶段的至少一个作业发生故障时才执行作业。
3.`always` - 无论前一阶段的工作状况如何，执行工作。
4.`manual` - 手动执行作业（在GitLab 8.10中添加）。阅读 下面的手动操作。
例如：
```
stages:
- build
- cleanup_build
- test
- deploy
- cleanup

build_job:
  stage: build
  script:
  - make build

cleanup_build_job:
  stage: cleanup_build
  script:
  - cleanup build when failed
  when: on_failure

test_job:
  stage: test
  script:
  - make test

deploy_job:
  stage: deploy
  script:
  - make deploy
  when: manual

cleanup_job:
  stage: cleanup
  script:
  - cleanup after jobs
  when: always
```
上面的例子解释：
1.`cleanup_build_job`仅在build_job失败时执行。
2.始终执行`cleanup_job`作为流水线的最后一步，无论成功或失败。
3.允许您`deploy_job从GitLab`的UI 手动执行。

###### Manual actions
>Gitlab 8.10中介绍

手动操作 是特殊的job类型是不会自动构建的，他们需要用户明确的启动，可以从pipeline,build,environment和deployment views开始操作。

手动操作的示例使用是部署到生成环境。
更多文档请参考:[ environments documentation.](https://docs.gitlab.com.cn/ce/ci/environments.html#manually-deploying-to-environments)

手动操作是可选的和可阻止的，阻止手动操作将会在操作定义pipeline阶段执行。当有人通过按钮执行阻止手动操作切换时候，可以恢复pipeline的执行。

当pipeline构建成功，但是pipeline 被阻止的时候提交合并则不会合并Pipeline，pipeline也有一种特殊的状态叫做 manual

manual操作默认是不阻止的，如果你想让manual操作阻止，就增加`allow_failure:false`添加到`.gitlab-ci.yml`.`allow_failure: true`默认情况下，可选手动操作已设置。
> GitLab 9.0中引入了禁止手动操作




