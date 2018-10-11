---
title: CentOS 7安装和配置wcp知识库系统
date: 2018-10-11 09:03:47
tags: [Linux, Wcp]
toc: true
---

### 1.准备工作

#### 1.1 安装解压工具

```shell
yum install -y unzip zip
```

解压方法 `unzip 文件名.zip`。

#### 1.2 下载必要的安装包

我已经将部分配置用的包上传到七牛云储存，使用以下外链即可下载。

- JDK http://pdfhkggfi.bkt.clouddn.com/jdk-8u181-linux-x64.tar.gz
- tomcat http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.90/bin/apache-tomcat-7.0.90.tar.gz

WCP：

- WCP3.2.5免费版 http://pdfhkggfi.bkt.clouddn.com/WCP3.2.5.free.zip
- wcp3.2.5.sql http://pdfhkggfi.bkt.clouddn.com/wcp3.2.5.sql

###2.安装JDK

#### 2.1下载JDK

创建一个目录并获取JDK

```shell
mkdir /usr/java 
cd /usr/java 
wget http://pdfhkggfi.bkt.clouddn.com/jdk-8u181-linux-x64.tar.gz
```

#### 2.2 解压JDK

```shell
tar -zxvf jdk-8u181-linux-x64.tar.gz
```

#### 2.3 设置环境变量

```shell
vi /etc/profile 
```

#### 2.4 在profile中添加如下内容

```
# set java environment
JAVA_HOME=/usr/java/jdk1.7.0_79 
JRE_HOME=/usr/java/jdk1.7.0_79/jre 
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib 
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin 
export JAVA_HOME JRE_HOME CLASS_PATH PATH 
```

#### 2.5 让修改生效

```shell
source /etc/profile
```

#### 2.6 验证JDK有效性

```
java -version 
java version "1.7.0_79" 
Java(TM) SE Runtime Environment (build 1.7.0_79-b15) 
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode) 
```

###3.安装tomcat
#### 3.1 创建目录并下载tomcat

```shell
mkdir -p /web/wcp && cd /web/wcp
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.90/bin/apache-tomcat-7.0.90.tar.gz
```

#### 3.2 解压

```shell
tar -zxvf apache-tomcat-7.0.90.tar.gz
```

#### 3.3 重命名为tomcat文件夹

```shell
mv apache-tomcat-7.0.90 tomcat
```

#### 3.4 设置tomcat

```shell
vim /etc/profile.d/tomcat.sh
```

输入：

```
CATALINA_HOME=/web/wcp/tomcat
PATH=$CATALINA_HOME/bin:$PATH
export CATALINA_HOME PATH
```

#### 3.5 使配置生效

```shell
source /etc/profile.d/tomcat.sh
```

### 4.安装数据库MySQL与配置数据库

#### 4.1 安装数据库

```shell
yum install mariadb mariadb-server mariadb-devel
```

#### 4.2 大小写配置

> WCP 的数据库要使用表不区分大小写，即 lower_case_table_names=1，如果设置区分大小写，程序会因为table不存在而报错

```shell
vim /etc/my.cnf
```

在 `[mysqld]`  一栏下面新增一行:

```
lower_case_table_names=1  
```

使用 `systemctl start mariadb.service` 启动数据库。

#### 4.3 初始化数据库密码

```shell
mysql_secure_installation // 执行这个语句来设置密码
```

设置密码，会提示先输入密码

```shell
Enter current password for root (enter for none):<–初次运行直接回车
```

设置密码

```shell
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
New password: <– 设置root用户的密码
Re-enter new password: <– 再输入一次你设置的密码
```

其他配置

```shell
Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,
Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车
```

初始化MariaDB完成。

#### 4.4 为WCP生成数据库

登录数据库

```
mysql -uroot -p密码
```

```shell
MariaDB [(none)]> CREATE DATABASE wcpdb; // 数据库 wcpdb
MariaDB [(none)]> GRANT ALL PRIVILEGES ON wcpdb.* TO 'wcpuser'@'%' IDENTIFIED BY 'wcpdbpass@1';  // 设置权限
MariaDB [(none)]> FLUSH PRIVILEGES;  // 使权限生效
MariaDB [(none)]> exit
```

### 5.部署页面

#### 5.1 下载项目

进入“tomcat”的“webapps”目录来进行操作

```shell
cd /web/wcp/tomcat/webapps
```

执行以下语句下载项目文件

```shell
wget http://pdfhkggfi.bkt.clouddn.com/WCP3.2.5.free.zip
```

```shell
wget http://pdfhkggfi.bkt.clouddn.com/wcp3.2.5.sql
```

#### 5.2 将WCP知识库的SQL语句导入数据库

```shell
mysql -uwcpuser -pwcpdbpass@1 -h127.0.0.1 wcpdb < /web/wcp/tomcat/webapps/wcp3.2.5.sql
```

#### 5.3 解压下载的项目

```shell
unzip WCP3.2.5.free.zip
```

#### 5.4 修改项目配置的数据库连接名称和密码

```shell
cd /web/wcp/tomcat/webapps/wcp-web/WEB-INF/classes
vi jdbc.properties
```

修改结果：

```shell
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/wcpdb?useUnicode=true&characterEncoding=utf-8
jdbc.username=wcpuser
jdbc.password=wcpdbpass@1
```

#### 5.5 启动tomcat

为了避免冲突，将tomcat的端口修改为8088

```shell
vim /web/wcp/tomcat/conf/server.xml
```

```xml
<Connector port="8088" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

```

启动：

```shell
bash /web/wcp/tomcat/bin/startup.sh
```

启动完成之后访问 http://ip:8088/wcp 即可查看项目。默认登录的用户名密码为：sysadmin 、111111。



### 参考资料

[WCP知识库安装](https://www.sjsir.wang/archives/153)

[wcp知识库系统的安装（Linux） ](http://www.wcpdoc.com/webdoc/view/Pub8a2831b3510abcb2015123b24c1f019d.html)

[Linux安装MariaDB（Mysql）和简单配置](https://www.cnblogs.com/jpfss/p/6568976.html) 

[How to Grant All Privileges on a Database in MySQL](https://chartio.com/resources/tutorials/how-to-grant-all-privileges-on-a-database-in-mysql/) 