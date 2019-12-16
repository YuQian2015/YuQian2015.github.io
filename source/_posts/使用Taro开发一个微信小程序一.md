---
title: 使用Taro开发一个微信小程序一
date: 2019-12-16 21:02:46
tags:
---

# 使用Taro开发一个微信小程序（一），小程序注册、项目创建

视频地址：<https://www.bilibili.com/video/av76872374/> 

### 为什么使用Taro

- Taro是一个支持多端开发的框架，写一套代码可以编译成各主流小程序，并且包括H5和RN。
- 和React接轨，方便在React和Taro之间无缝切换，学习门槛不会太高。

关于Taro技术选型，这里有一篇介绍，[去查看](http://blog.moyufed.com/2019/11/23/Taro%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B%E4%B8%8E%E6%9D%83%E8%A1%A1/) 。

### 注册小程序

1. 注册小程序需要一个电子邮箱，然后进入[微信公众平台](https://mp.weixin.qq.com/>)，点击 【立即注册】。
2. 进入到账号类型选择，点击【小程序】，注意，**一个邮箱只能注册一个账号类型，这些账号可以关联到同一个微信号** 。
3. 完善注册信息，需要输入邮箱地址、密码并且确认密码，填入验证码之后点击【注册】。
4. 注册之后，会提示验证邮箱，点击【登录邮箱】去到收件箱找到微信发送的邮件，点击详情里面的链接完成验证。
5. 完善微信小程序主体信息，包括账号地区、主体类型（个人）、以及主体的信息，然后使用微信二维码扫描验证。
6. 提交信息完成之后，即可点击【前往小程序】跳转到小程序管理页面，到这步，我们已经完成了小程序的注册。

### 下载安装微信开发工具

在进行微信小程序开发的过程中，我们需要使用到微信开发工具，[点击进入](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)，我们可以选择自己喜欢的版本进行下载，下载完成之后双击执行安装，不多做介绍。

### 下载Taro

详细详情，可以参考[Taro文档](https://taro-docs.jd.com/taro/docs/GETTING-STARTED.html) ，下面做要点介绍：

首先检查 `nodejs` 版本，打开命令行工具（本人使用 ConEmu ），在命令行执行：

```powershell
$ node -v
v8.10.0
```

> Taro 项目基于 node，请确保已具备较新的 node 环境（>=8.0.0），推荐使用 node 版本管理工具 [nvm](https://github.com/creationix/nvm) 来管理 node，这样不仅可以很方便地切换 node 版本，而且全局安装时候也不用加 sudo 了。 

关于 nvm 安装，这里有一个介绍视频，[点击查看](https://www.bilibili.com/video/av71367431/) 。接着执行命令安装Taro：

```powershell
$ npm install -g @tarojs/cli
```

为了避免我们在使用 `sass` 时出现安装安装错误，可以再全局安装[`mirror-config-china`](https://www.npmjs.com/package/mirror-config-china)：

```powershell
$ npm install -g mirror-config-china
```

安装完成之后，可以执行命令查看Taro的信息：

```powershell
$ taro info
👽 Taro v1.3.25


  Taro CLI 1.3.25 environment info:
    System:
      OS: Windows 10
    Binaries:
      Node: 8.10.0 - C:\Program Files\nodejs\node.EXE
      Yarn: 1.19.1 - C:\Program Files (x86)\Yarn\bin\yarn.CMD
      npm: 5.6.0 - C:\Program Files\nodejs\npm.CMD
```

### 创建第一个 Taro 项目

执行命令创建项目：

```powershell
$ taro init 项目名称

# 以本次实战项目为例
$ taro init moyufed
√ 拉取远程模板仓库成功！
? 请输入项目名称！ moyufed
? 请输入项目介绍！ 项目介绍
? 是否需要使用 TypeScript ？ No
? 请选择 CSS 预处理器（Sass/Less/Stylus） Sass
? 请选择模板 mobx
```

执行 `init` 之后，Taro 会自动执行依赖的安装。如果执行安装失败，我们可以按 ctrl+c 结束，手动执行命令安装：

```powershell
# 使用 yarn 安装依赖
$ yarn install
# 或者使用 npm 安装依赖
$ npm install
```

执行命令启动：

```powershell
# 使用 yarn
$ yarn dev:weapp
$ yarn build:weapp
# 使用 npm script
$ npm run dev:weapp
$ npm run build:weapp
```

### 运行和预览

打开已经下载安装好的微信开发工具，用自己的微信扫描登录。然后进入项目目录，执行命令启动dev ：

```powershell
# 使用 yarn
$ yarn dev:weapp
# 使用 npm script
$ npm run dev:weapp
```

为了能够使用刚注册好的小程序进行预览，先去完善小程序信息：

1. 进入微信公众平台小程序首页，点击小程序信息栏目的【填写】前往信息完善页面，完善和提交小程序信息之后，点击右侧【开发】菜单，选择【开发设置】标签，可以看到小程序ID，复制小程序ID，填入项目根目录的 `project.config.json`  ，修改`appid` 的值为小程序的ID。
2. 在小程序开发工具中，点击新建小程序，选择【导入项目】，下拉选择我们创建的小程序项目根文件夹，点击【导入】，这时我们就可以边修改代码边查看小程序效果了。

更多详情，可以点击文章开头的视频连接查看。