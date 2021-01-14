---
title: 快速创建Linux用户
date: 2021-01-14 18:04:25
tags: [Linux, 全栈]
---

### 连接Linux服务器

```
IP：xxx.xxx.xxx.xxx
端口：一般是22
root用户名：xxxxx
root登录密码：**********
```
### 执行创建用户命令

假设需要创建的用户为 tom, 在终端执行： `useradd -m 用户名` 创建一个 admin 用户和 home 目录
```
$ useradd -m tom
```

### 查看用户

执行 `compgen -u` 查看用户

```
$ compgen -u
```

### 为用户设置密码

使用 `passwd 用户名` 为用户设置密码

```
$ passwd tom
```

**键盘输入密码两次**

### 添加用户到root组

使用 `usermod -a -G root 用户名` 添加用户到root组， 其中 `-a` 表示添加，`-G` 表示添加到组

```
$ usermod -a -G root tom
```

### 修改bash外壳

使用命令 `chsh -s /bin/bash 用户名`

```
$ chsh -s /bin/bash tom
```

### 补充说明

**家目录**

一般用户，家目录是 `/home/用户名` ，而 root 用户，家目录是 `/root` 。

root登录系统，执行如下命令进入root的家目录：

```
cd ~
```

进入家目录后执行如下命令获取具体路径：

```
$ pwd
```


**切换用户**

执行 `su` 命令切换用户，从普通用户切到 root 用户需要密码：

```
$ su

或者

$ su root
```
从 root 用户切到普通用户不需要密码：

```
$ su 用户名

或者

ctl+d 
```