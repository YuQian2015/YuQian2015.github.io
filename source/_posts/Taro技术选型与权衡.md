---
title: Taro技术选型与权衡
date: 2019-11-23 23:03:38
tags: [前端]
toc: true
---


Taro 是由凹凸实验室打造的一套遵循 React 语法规范的多端统一开发框架。

今市面上应用的形态多种多样，Web、Native、微信小程序等，当业务要求同时在不同的端都要求有所表现的时候，针对不同的端去编写多套代码的成本显然非常高，这时候只编写一套代码就能够适配到多端的能力就显得极为需要。使用 Taro，我们可以只书写一套代码，再通过 Taro 的编译工具，将源代码分别编译出可以在不同端（微信小程序、H5、App 端等）运行的代码。

### Taro 有哪些特性

- 一键生成可以在微信小程序/H5/ReactNative等端运行的代码

- 采用React语法标准，支持JSX书写，现代的编辑器默认都对 JSX 进行了支持 

- 支持组件化开发，让代码可以复用，让功能开发更加清晰
- 全面支持TypeScript
- 代码提示，实时代码检查，提升 开发效率
- 配套的开发工具Taro CLI实现开发流程自动化

### Taro支持的平台

- 微信小程序
- H5 
- React Native 
- 支付宝小程序 
- 百度智能小程序 
- 字节跳动小程序 
- 快应用（未上线）
- QQ 浏览器轻应用 （未上线）

### 代码示例

Taro 全都是 `Component` 组件，并且和 React 的生命周期完全一致。掌握了 React 就几乎掌握了 Taro。

```jsx
import Taro, { Component } from '@tarojs/taro'
import { View, Button } from '@tarojs/components'

export default class Index extends Component {
  constructor () {
    super(...arguments)
    this.state = {
      title: '首页',
      list: [1, 2, 3]
    }
  }
  
  componentWillMount () {}
  
  componentDidMount () {}
  
  componentWillUpdate (nextProps, nextState) {}
  
  componentDidUpdate (prevProps, prevState) {}
  
  shouldComponentUpdate (nextProps, nextState) {
    return true
  }

  add = (e) => {
    // dosth
  }

  render () {
    return (
      <View className='index'>
        <View className='title'>{this.state.title}</View>
        <View className='content'>
          {this.state.list.map(item => {
            return (
              <View className='item'>{item}</View>
            )
          })}
          <Button className='add' onClick={this.add}>添加</Button>
        </View>
      </View>
    )
  }
}
```

### 技术选型与权衡

#### 优点

- 一键生成可以在微信小程序/H5/ReactNative等端运行的代码，小程序端已支持微信小程序、支付宝小程序，但无法提供在线版，H5 端、RN 端可在线预览。
- 使用 `TypeScript` 来对代码的可靠性进一步增强，或使用 `PropsType` 在运行时进一步保障代码可靠性。 
- Taro 的所有 API（包括微信小程序等端能力接口）都有智能的提醒和自动补全，包括接口的参数和返回值。 
- Taro语法规范遵循React规范，熟悉React就能使用Taro进行开发。
- 使用JSX作为模板和业务代码系统，JSX 的本质就是 JavaScript 的语法增强，所以例如没有 `import` 组件等语法错误在编译期就能发现。
- 支持sass、less、postcss。
- 支持React 组件化规范，组件可复用多端。
- 内建构建系统+webpack自动构建
- 全面支持TypeScript

#### 缺点

##### React-like不完全等同 React 

- 小程序中受限于 WXML 语法 ，组件调用不支持扩展操作符
- JSX暂时只能写在render中

##### 多端差异不可避免

- 某些功能在相应端上没有支持 
- 样式支持、写法上存在差异 

##### 没有真正支持多端的UI组件库

- Taro UI 是基于 Taro 框架开发的多端 UI 组件 
- Taro UI 暂不支持 React Native 

#### 存在的坑

##### 样式差异管理

- 因为 React Native 与一般 Web 样式支持度差异较大，用了大量 RN 不支持的样式再要去兼容 RN 无异于重写页面。
- 样式上 H5 最为灵活，小程序次之，RN 最弱，统一多端样式即是对齐短板，也就是要以 RN 的约束来管理样式，同时兼顾小程序的限制。

##### H5 端的坑

- Taro 对H5 端的支持度尚可，在 CSS 特性上就不用像 RN 那样拘束。
- 内置组件不够完善、端能力缺失较多，Taro 的设计是以微信小程序为基准。
- 小程序、RN 都没有跨域问题，但 H5 会有，这个可通过 devServer.proxy 解决
- 以及编译打包的静态资源是固定文件名，改成带 hash 值方便缓存管理，这些配置在项目里的 src/config 。

##### 对React Native 支持可能存在部分问题

- React Native 不支持 text-overflow，而是提供原生支持 （Text 组件传入 numberOfLines={num} 属性） 
- React Native 的 view 不支持 click 事件，需要用 Touchable 组件 

##### 多端要求较高 

- 对不同端的具体差异有所了解 
- 样式实现较为苛刻，需兼顾多端、有所取舍 
- 端能力差异可能需要自己填坑 
- 需要对隔断差异做出取舍

##### 新手使用

- 需要上手React、了解Redux数据流管理
- 和现有 ionic 组件会有出入的地方需要兼顾和取舍

### 多端填坑方式 

- Taro 支持环境判断编译时只保留相应端代码 
- Taro 支持引入相应端的原生代码 

```js
function foo() {
  if (process.env.TARO_ENV === 'weapp') {
    // 微信小程序逻辑
  }
  if (process.env.TARO_ENV === 'h5') {
    // H5 逻辑
  }
  if (process.env.TARO_ENV === 'rn') {
    // RN 逻辑
  }
}
```

### 技术和开发

- 学习 React/Redux
- 采用自有UI组件和Taro UI，减少多端成本。
- 适配着重于兼容RN，小程序，根据多端差异协调取舍。
- 以H5为主、小程序为辅、RN次之的适配方案，能够快速应对项目二开，从了解到熟悉，循序渐进。

### 总结

Taro 虽然是以多端为设计目标，但重心是小程序端，RN 端目前的支持情况不算特别理想。但充分理解多端差异、掌握正确的多端开发的方式，在简单的项目上是完全值得尝试。 
