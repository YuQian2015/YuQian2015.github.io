---
title: Windows下安装NVM
date: 2019-11-23 22:30:15
tags: [Nodejs]
toc: true
---


# Windows下安装NVM

视频地址：<https://www.bilibili.com/video/av71367431/> 

### 1、事先准备

卸载已有的 `nodejs` ，并移除原有的 `nodejs` 目录（重要）

### 2、下载和安装nvm

访问 [下载地址](<https://github.com/coreybutler/nvm-windows/releases> ) 选择自己需要的nvm版本进行下载，下载 [nvm-setup.zip](https://github.com/coreybutler/nvm-windows/releases/download/1.1.7/nvm-setup.zip)  ，解压并执行里面的 `.exe` 文件进行安装

### 3、修改为淘宝镜像

以本机为例，修改 `C:\Users\Administrator\AppData\Roaming\nvm\settings.txt` 文件增加淘宝下载镜像地址：

```
root: C:\Users\Administrator\AppData\Roaming\nvm
path: C:\Program Files\nodejs
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

### 4、打开CMD，执行nvm查看是否安装成功

1.nvm 安装，直接下载github 安装包即可。

2.nvm 安装成功后，需要手动的开启。

nvm on 

3.不进行第二步，nvm install 8.9.0 ，输入node，依旧提示

```powershell
'node' is not recognized as an internal or external command,
operable program or batch file.
```

### 参考资料

[Win10 nvm 安装node 之 nvm on](https://juejin.im/post/5cfde431e51d45775b419bac)
[NVM for Windows not working?](https://stackoverflow.com/questions/28313372/nvm-for-windows-not-working)
[`nvm node_mirror [url]` not work](https://github.com/coreybutler/nvm-windows/issues/216)
[nvm介绍及使用](https://www.jianshu.com/p/d0e0935b150a)
