# Gitlab-ci

### 简介：

基于 Gitlab 9.0.0 编写,gitlab-ci 在8.0以上支持，只需要支持.gitlab-ci.yml文件，添加一个Runner，就可以进行持续集成。

### 概念：

#### Pipeline

一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。

任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：

  


