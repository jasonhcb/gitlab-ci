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


