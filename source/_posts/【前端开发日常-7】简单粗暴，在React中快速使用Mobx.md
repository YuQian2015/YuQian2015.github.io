---
title: 【前端开发日常 - 7】简单粗暴，在React中快速使用Mobx
date: 2020-09-17 17:05:33
tags: [React, 前端]
---

### 需求背景
最近在做 React 项目，是一个简单的编辑页，刚开始没有考虑到复杂的场景，随着新的想法加入，现在需要增加多个页面，因此使用了 react-router ，为了在每个页面之间共享数据，实现状态管理，决定在项目中加入 Mobx 。

### 解决方案
1、安装 mobx 和 mobx-react ；
2、创建 store 并且使用 ；
3、使用@装饰器，在 [【前端开发日常 - 3】让create-react-app支持@装饰器语法](http://moyufed.com/2020/09/17/%E3%80%90%E5%89%8D%E7%AB%AF%E5%BC%80%E5%8F%91%E6%97%A5%E5%B8%B8-3%E3%80%91%E8%AE%A9create-react-app%E6%94%AF%E6%8C%81-%E8%A3%85%E9%A5%B0%E5%99%A8%E8%AF%AD%E6%B3%95/) 文章中有介绍。

### 安装mobx

``` shell
$ yarn add mobx mobx-react
```
### 创建 store

在 `src` 目录下增加一个 `store` 文件夹， 创建一个 `index.js` 文件，用来导出所有的 `store` ， 例子：
``` javascript
// src/store/index.js

import HomeStore from "./homeStore";
import UserStore from "./userStore";
import ArticleStore from "./articleStore";
let homeStore = new HomeStore();
let userStore = new UserStore();
let articleStore = new ArticleStore();
const stores = {
  homeStore, userStore, articleStore
};
// 默认导出 
export default stores;
```
关于 `store` 的创建，有以下几个部分：
1、在 `mobx` 引入 `observable` 、 `action` ，导出一个 `class` ；
2、使用装饰器， 用 `@observable` 定义变量 ， 使用 `@action` 定义方法来修改变量。
以 `articleStore` 为例：

``` javascript
// src/store/articleStore.js

import { observable, action } from "mobx";

import { ApiService } from "../services";

class ArticleStore {
    @observable articleList = [];

    @observable articlePage = 1;
    articlePageSize = 10;
    @observable articleCount = 0;

    @action async getArticleList(page = 1) {
        this.articlePage = page;
        const data = {
            page: this.articlePage,
            pageSize: this.articlePageSize
        }
        const result = await ApiService.getArticleList(data);
        if (result && result.success) {
            this.articleList = result.data.list;
            this.articleCount = result.data.count;
        }
    }
}
export default ArticleStore;
```
### 使用 store
`store` 创建好了之后，就是在入口部分使用，需要从 `mobx-react` 引入 `Provider` ，代码示例：

``` jsx
//
// ...
import { Provider } from "mobx-react";
import stores from "./store";
// ...
function App() {
  return (
    <Provider {...stores}>
      <div className="App">
        <Router><RouterPage /></Router>
      </div>
    </Provider>
  );
}

export default App;

```
引入 `Provider` 和 `store` ，从而创建一个数据的容器，子组件放在容器中实现跨组件数据共享， 类似于 react 的 `context` 。
然后，在需要的地方使用 `store` ，从 `mobx-react` 引入 `observer` 和  `inject` ， 使用装饰器，之后从组件的 `props` 上取值：

``` jsx
import React, { Component } from 'react';
import { observer, inject } from "mobx-react";

import Article from './Article.jsx';

@inject("articleStore")
@observer
class ArticleList extends Component {
    constructor(props) {
        super(props); 
    }
  
  	async componentDidMount() {
        this.getPage();
    }

    getPage = async (page = 1) => {
        const { articleStore } = this.props;
        await articleStore.getArticleList(page);
    }
    
    render() {
        
        const { articleStore: { articleList } } = this.props;
        return (
            <div>
          		{
          			articleList.map((item, index) => <Article key={item._id} data={item} />)
    			}
			</div>
        )
    }
}

export default ArticleList;
```
`@observer` 将组件变成响应式组件，数据改变时候可以重新渲染。
`@inject("articleStore")` 将数据注册到组件，可以在 props 取到。