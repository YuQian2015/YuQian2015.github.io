---
title: 【前端开发日常 - 4】Windows安装Redis及简单使用
date: 2020-09-30 17:08:15
tags: [全栈]
---

## 需求背景
目前在使用阿里 Egg.js 开发后台应用，考虑到一些高并发的接口会经常调用数据库查询，因此增加Redis作为服务端缓存。

## 解决方案

1. 安装 Redis，由于系统是 Windows7，因此需要下载对应 Windows 的版本，下载地址：[https://github.com/MicrosoftArchive/redis/releases](https://github.com/MicrosoftArchive/redis/releases)

2. 简单配置 Redis 启动，为 Redis 的连接增加密码，以确保安全性

## 核心代码
### 安装 Redis

访问上面的【下载地址】，选择需要的版本，展开 Assets 目录，可以选择 .msi 文件或 .zip 文件下载。如果下载 **.msi** 文件，可以直接执行安装（勾选对应的复选框），安装通过之后，Windows 会直接将 Redis 写入服务。

如果下载 **.zip** 文件，还需要对文件进行解压和安装，需要在 redis 目录执行命令：

```shell
$ redis-server.exe --service-install redis.windows.conf
```

上面的命令会安装一个Windows服务，关于服务的命令有：

```shell
$ redis-server.exe --service-uninstall # 卸载服务

$ redis-server.exe --service-start # 开启服务

$ redis-server.exe --service-stop # 停止服务

$ redis-server.exe --service-name name # 重命名服务
```

### 检查 Windows 服务

以 Windows7 为例，可以进入 控制面板\所有控制面板项\管理工具 找到 【服务】，点开之后，在服务列表里面找到 Redis。
### Redis 启动和关闭
安装好之后，我们可以通过服务启动和关闭 Redis 了，我们可以查看系统的任务管理器，检查是否启动了 redis-server，下面是 redis-server 的启动和关闭命令：

``` shell
$ redis-server.exe  --service-start # 启动
$ redis-server.exe  --service-stop # 关闭
```
### 使用 cli
确认 redis-server 启动之后，我们可以执行 redis 目录里面的 redis-cli.exe 文件，这样就可以连接上 Redis 了。当然，我们也可以通过命令行 CMD 来链接 Redis。

可以直接进行 Redis 存取了：

``` shell
$ redis-cli.exe -h 127.0.0.1 -p 6379
127.0.0.1:6379> set firstKey "moyufed"
OK
127.0.0.1:6379> get firstKey
"moyufed"
127.0.0.1:6379>
```
### 增加密码
在 Redis 的安装目录中，有 redis.windows.conf 、redis.windows-service.conf 两个文件，我们使用记事本打开（推荐Notepad++），找到需要设置密码的行，可以 Ctrl + F 查找 `“requirepass”` 的所在的行，分别设置 Redis 密码（去掉前面的 `#`），例子：
``` shell
requirepass moyufed
```
### 使用 Redis
修改好之后重启 redis-server ，此时连接 Redis 之后进行 Redis 存取需要登录，例子：

``` shell
Administrator@WIN-1706081829 C:\Users\Administrator
$ redis-server.exe  --service-stop
[42636] 25 Feb 18:42:53.622 # Redis service successfully stopped.

Administrator@WIN-1706081829 C:\Users\Administrator
$ redis-server.exe  --service-start
[38828] 25 Feb 18:42:58.576 # Redis service successfully started.

Administrator@WIN-1706081829 C:\Users\Administrator
$ redis-cli.exe -h 127.0.0.1 -p 6379
127.0.0.1:6379> set firstKey "moyufed"
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth moyufed
OK
127.0.0.1:6379> set firstKey "moyufed"
OK
127.0.0.1:6379> get firstKey
"moyufed"
127.0.0.1:6379>
```
## 参考文档
Window配置Redis环境和简单使用：https://www.cnblogs.com/wxjnew/p/9160855.html
Windows下Redis安装配置和使用注意事项：https://www.cnblogs.com/LMJBlogs/p/11550170.html