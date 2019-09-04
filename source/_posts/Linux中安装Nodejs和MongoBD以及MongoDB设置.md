---
title: Linux中安装Nodejs和MongoBD以及MongoDB设置
date: 2019-09-04 18:18:10
tags: [nodejs, mongodb]
---

### 1、安装 nodejs

#### 方式一：使用已编译好的包

首先来新建一个目录，下载nodejs：

```powershell
$ cd /usr
$ mkdir software
$ cd software/
```

下载需要的nodejs版本，本示例使用上传到CDN上的node包：

```powershell
$ wget http://qiniu.moyufed.com/node-v10.16.3-linux-x64.tar.xz
```

解压node包并且测试node版本：

```powershell
$ tar xf  node-v10.16.3-linux-x64.tar.xz      #  解压
$ cd node-v10.16.3-linux-x64                  # 进入解压目录
$ ./bin/node -v                               # 执行node命令 查看版本
```

解压文件的 bin 目录底下包含了 node、npm 等命令，我们可以使用 ln 命令来设置软连接：

```powershell
$ ln -s /usr/software/node-v10.16.3-linux-x64/bin/npm   /usr/local/bin/ 
$ ln -s /usr/software/node-v10.16.3-linux-x64/bin/node   /usr/local/bin/
```

验证版本

```powershell
$ node -v
```

#### 方式二：CentOS下源码安装 Node.js

下载需要的nodejs源码并解压，本示例使用上传到CDN上的node源码：

```powershell
$ cd /usr/local/src/
$ wget http://qiniu.moyufed.com/node-v10.16.3.tar.gz
$ tar zxvf node-v10.16.3.tar.gz
```

进入目录编译安装：

```powershell
$ cd node-v10.16.3
$ ./configure --prefix=/usr/local/src/node-v10.16.3
$ make
$ make install
```

配置NODE_HOME，进入profile编辑环境变量：

```powershell
$ vim /etc/profile
```

找到 `export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL` 对应的行上面添加：

```
# nodejs
export NODE_HOME=/usr/local/src/node-v10.16.3
export PATH=$NODE_HOME/bin:$PATH
```

:wq保存并退出，执行命令使配置生效

```powershell
$ source /etc/profile
```

验证版本

```powershell
$ node -v
```

### 2、安装MongoDB

下载MongoDB，本示例使用上传到CDN的包：

```powershell
$ curl -O http://qiniu.moyufed.com/mongodb-linux-x86_64-4.0.12.tgz    # 下载
```

解压下载下来的包：

```powershell
$ tar -zxvf mongodb-linux-x86_64-4.0.12.tgz
```

将解压包拷贝到指定目录

```powershell
$ mv mongodb-linux-x86_64-4.0.12/ /usr/local/mongodb
```

参考上文进入profile编辑环境变量：

```
export PATH=/usr/local/mongodb/bin:$PATH
```

:wq保存并退出，执行命令使配置生效

```powershell
$ source /etc/profile
```

### 3、MongoDB设置

#### 3.1、MongoDB 启动和关闭

##### 3.1.1、MongoDB 启动基本参数

| 参数        | 描述                                  |
| ----------- | ------------------------------------- |
| -f          | 指定配置文件                          |
| --port      | 指定端口，默认是27017                 |
| --dbpath    | 数据目录路径                          |
| --logpath   | 日志文件路径                          |
| --logappend | 日志append而不是overwrite             |
| --fork      | 以创建子进程的方式运行                |
| --journal   | 日志提交间隔，默认100ms               |
| --nojournal | 关闭日志功能，2.0版本以上是默认开启的 |

##### 3.1.2、MongoDB关闭

**不要使用 kill 直接杀 MongoDB 进程的方式关闭数据节点，会造成数据损坏**

```
killall mongodb
#or
kill -9 mongo-pid
```

上面方法可以关闭 MongoDB，**但不是正确的做法**，MongoDB 提供了关闭数据库的命令，连接到mongodb，切换到admin数据库，然后执行关闭：

```
> use admin
> db.runCommand("shutdown")

也可以使用 db.shutdownServer() 命令
> use admin
> db.shutdownServer()
> db.shutdownServer({force : true}) // 强制关闭Mongod，应对副本集中主从时间差超过10s时不允许关闭主库的情况
```

#### 3.2、设置数据库目录

以本机为例，在share-trace用户的目录下简历一个目录，本例命名为MongoDB：

```powershell
$ pwd
/home/share-trace
$ mkdir mongoDB
$ cd mongoDB/
$ pwd
/home/share-trace/mongoDB
$ vi
```

本例通过文件来设置MongoDB的启动，因此会新建一个文件，文件内容如下：

```powershell
systemLog:
 destination: file
 path: /home/share-trace/mongoDB/log/mongod.log
 logAppend: true
net:
 bindIp: 0.0.0.0
 port: 3001
storage:
 dbPath: /home/share-trace/mongoDB/db
processManagement:
 fork: true
```

执行 `:wq mongod.cfg`  保存文件名为 `mongod.cfg` ：

```powershell
$ ls
mongod.cfg
```

然后创建 MongoDB 的 log 目录和 db 目录，并且尝试启动数据库：

```powershell
ls
mongod.cfg
$ mkdir log
$ ls
log  mongod.cfg
$ mkdir db
$ ls
db  log  mongod.cfg

$ mongod --config /home/share-trace/mongoDB/mongod.cfg
2019-09-02T14:18:05.797+0800 I STORAGE  [main] Max cache overflow file size custom option: 0
about to fork child process, waiting until server is ready for connections.
forked process: 25174
child process started successfully, parent exiting
```

验证数据库进程：

```powershell
$ netstat -tunpl | grep 25174
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:3001            0.0.0.0:*               LISTEN      25174/mongod 
```

#### 3.3、连接数据库

执行 `mongo -port 3001` 命令即可连接数据库：

```powershell
$ mongo -port 3001
MongoDB shell version v4.0.12
connecting to: mongodb://127.0.0.1:3001/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("77bc76f5-905d-4dc3-8bb9-6e64789150f1") }
MongoDB server version: 4.0.12
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2019-09-02T14:18:05.805+0800 I STORAGE  [initandlisten] 
2019-09-02T14:18:05.805+0800 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2019-09-02T14:18:05.805+0800 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] 
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] 
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] 
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] 
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] 
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. rlimits set to 4096 processes, 65535 files. Number of processes should be at least 32767.5 : 0.5 times number of files.
2019-09-02T14:18:07.097+0800 I CONTROL  [initandlisten]
```

#### 3.4、创建管理员用户到admin数据库

执行 `mongo -port 3001` 连接数据库后，切换到admin数据库

```powershell
> use admin
switched to db admin
```

创建一个用户，本例使用的用户名和密码为 admin ，具体根据个人情况修改：

```
> db.createUser(
...   {
...     user: "admin",
...     pwd: "admin",
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
...   }
... )
Successfully added user: {
        "user" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ]
}
```

然后关闭mongoDB：

```powershell
> use admin
> db.shutdownServer()
```

#### 3.5、启动访问控制并创建用户

以启用访问控制的方式来启动MongoDB：

```powershell
$ mongod --config /home/share-trace/mongoDB/mongod.cfg --auth
```

再次连接数据库：

```powershell
$ mongo -port 3001
```

验证admin用户：

```powershell
> use admin
switched to db admin
> db
admin
> db.auth("admin", "admin")
1 # 返回1 授权成功, 否则返回0, 并提示失败
```

为自己的数据库创建用户，本例将使用一个exeShareTrace 数据库，具体的数据库名称以及用户名密码根据个人情况修改：

```powershell
# 执行
> use exeShareTrace
# 结果
switched to db exeShareTrace

# 执行
> db.createUser(
...   {
...     user: "admin",
...     pwd: "exe123",
...     roles: [ { role: "readWrite", db: "exeShareTrace" } ]
...   }
... )
# 结果
Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "exeShareTrace"
		}
	]
}
```

至此，数据库设置完成。

