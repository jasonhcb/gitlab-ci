# Create Shared Runner

* ##### 两种方式的对比：

  Specific Runner（指定型） 为特定的项目创建单独的运行 , Shared Runner（共享型） 可以为有相同需求的不同授权项目，便于升级，只要系统管理员能创建。

 1 . 什么情况下需要注册Shared Runner？
```
比如，GitLab上面所有的工程都有可能需要在公司的服务器上进行编译、测试、部署等工作，这个时候注册一个Shared Runner 供所有工程使用就很合适。
```
 2 . 什么情况下需要注册Specific Runner？
```
 比如，我可能需要在我个人的电脑或者服务器上自动构建我参与的某个工程，这个时候注册一个Specific Runner就很合适。
```

* ##### 创建Shared Runner

###### 1. 打开admin-&gt;setting-&gt;runners![](/assets/WX20170401-151214@2x.png)

###### 2.gitlab-runner register一个shared runner

```bash
sudo gitlab-runner register
```



