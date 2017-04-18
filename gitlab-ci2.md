##### only and except
`only`和 `except`是设置限制作业策略构建的两个参数：
  1.`only` 定义了jobs要运行的branch和tags的名称。
  2. `except`定义了jobs不运行哪些branch 和tags的名称。
  下面的几点规则是适用于两者的：
  * `only`和`except`是可以同时存在的，而且都会过滤两者的条件。
  * `only`和`except`允许使用正则表达式。
  * `only`和`except`允许使用特定的关键字：`branches`,`tags`,`triggers`。
  * `only`和`except`允许使用特殊的项目存储路径来作为过滤条件。

以下的例子是job 只针对以正则 `issue-`开头的参数，而所有的分支会被跳过。
```
job:
  # use regexp
  only:
    - /^issue-.*$/
  # use special keyword
  except:
    - branches
```