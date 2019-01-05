---
title: react-intl + React Context API 实现多语言切换
date: 2019-01-05 23:21:28
tags: [React, Webpack, OnsenUI]
toc: true
---


紧接着上一次的实战项目，这次要来完善多语言切换功能，使用 [react-intl](https://github.com/yahoo/react-intl) 来处理国际化，下面开始

### 1.安装 react-intl

```shell
npm install react-intl --save
```

> 先安装react-intl插件；这个库提供了 React 组件和Api两种方式来格式化日期，数字和字符串等。
为了兼容Safari各个版本，可能需要同时安装 intl，`npm install intl --save` ，intl在大部分的『现代』浏览器中是默认自带的，但是Safari和IE11以下的版本就没有了。

### 2.建立多语言文件
在src目录新增一个目录叫i18n，并建立两个文件en_US.js、zh_CN.js，代码如下：

en_US.js
```js
export const en_US = {
    'intl.signin.phone': "phone",
    'intl.signin.account': "account",
    'intl.signin.great': 'welcome to {city}'
};
```

zh_CN.js
```js
export const zh_CN = {
    'intl.signin.phone': "手机登录",
    'intl.signin.account': "账号登录",
    'intl.signin.great': '{city}欢迎你'
};
```

### 3.引入多语言文件
首先，紧接之前的项目，在PageContainer.jsx里面引入语言文件：

```js
// ...
import { Page, BackButton, ToolbarButton, Toolbar, Icon  } from 'react-onsenui';

//react 国际化
import {IntlProvider, addLocaleData} from 'react-intl';
import {zh_CN} from '../i18n/zh_CN'; // 中文
import {en_US} from '../i18n/en_US'; // 英文
import zh from 'react-intl/locale-data/zh';// react-intl语言包
import en from 'react-intl/locale-data/en';// react-intl语言包
addLocaleData([...en, ...zh]); // 需要放入本地数据库

class PageContainer extends React.Component {
   
    constructor(props) {
        super(props);
        this.state = {
            language: navigator.language
        };
    }
    // ...
    
    // 选择多语言
    chooseLocale(){
        const language = this.state.language;
        switch(language){
            case 'en-US':
                return en_US;
            case 'zh-CN':
                return zh_CN;
            default:
                return en_US;
        }
    }
    
    // ...
    render() {
        let { page, noBack, noHeader} = this.props;
        let { language } = this.state;
        // ...
        let tool = "";

        return <Page renderToolbar={() => tool} className="PageContainer">
            <IntlProvider locale={language} key={language} messages={this.chooseLocale()}>
                {page?page:null}
            </IntlProvider>
        </Page>
    }
    
}
```

上面的代码中

IntlProvider 有三个配置参数：

- locale, <string>, 语言标记，例如 'zh-CN' 'en-US'
- messages, <object>, 国际化所需的 key-value 对象
- formats, <object>, 自定义 format，比如日期格式、货币等

在定义好 IntlProvider 之后，我们就可以在页面使用它提供的 api 或者组件来进行国际化了。



#### 踩坑备注



**问题**

切换多语言时不能重新加载语言的问题，[FormattedMessage not updating live](https://github.com/yahoo/react-intl/issues/1103)

**解决方案**

 it looks like you need to add a `key` to `IntlProvider` to force an update when locale changes, e.g:

```jsx
<IntlProvider 
        locale={ lang } 
        messages={messages[lang]}
        key={ lang }
      >
```

As a warning, doing so means everything under `IntlProvider` is going to be re-rendered.

If you have time to dig in to solutions to everything getting re-rendered, read through:

[#1106](https://github.com/yahoo/react-intl/issues/1106) 

[#695](https://github.com/yahoo/react-intl/issues/695) 



### 4.使用多语言文件

以登录页举例，src/page/SignInPage.jsx：

```js
// ...

import {FormattedMessage} from 'react-intl'; // 引入FormattedMessage


export default class SignInPage extends React.Component {
    constructor(props) {
        // ...
    }
    componentWillMount() {}

    // ...
    
    render() {
        // ...
        const page = (<div className="SignInPage">
            <div className="sign-in-form">
                <FormattedMessage
                    tagName="p"
                    id='intl.signin.great'
                    description='欢迎'
                    defaultMessage='{city} 欢迎你'
                    values={{
                        city: '厦门'
                    }}
                />
                <Segment>
                    <button onClick={() => {this.changeType(1)}}><FormattedMessage id='intl.signin.phone' /></button>
                    <button onClick={() => {this.changeType(2)}}><FormattedMessage id='intl.signin.account' /></button>
                </Segment>
                
                // ...
            </div>
        </div>);
        return <PageContainer page={page} noBack={true} noHeader={true} />
    }
};
```
上面的代码中
 - id 指代的是这个字符串在配置文件中的属性名
 - description 指的是对于这个位置替代的字符串的描述，便于维护代码，不写的话也不会影响输出的结果
 - defaultMessage 当在locale配置文件中没有找到这个id的时候，输出的默认值
 - tagName 实际生成的标签，默认是 span
 - values 动态参数. 格式为对象

启动项目即可查看效果了。

### 5.Context API——切换语言

为了实现在子组件里面也能切换语言，这里使用了最新的React [Context API](https://reactjs.org/docs/context.html)  来实现。

####5.1创建context

新建一个src/context目录，并且在里面新建一个文件，LangContext.jsx：

```jsx
import React from 'react';
// 创建一个LangContext并且导出
export const LangContext = React.createContext({
    changeLanguage: () => {
    }
});
```

#### 5.2使用context

回过头来修改PageContainer.jsx：

```js
// ...

//react 国际化
import {IntlProvider, addLocaleData} from 'react-intl';
import {zh_CN} from '../i18n/zh_CN'; // 中文
import {en_US} from '../i18n/en_US'; // 英文
import zh from 'react-intl/locale-data/zh';// react-intl语言包
import en from 'react-intl/locale-data/en';// react-intl语言包
addLocaleData([...en, ...zh]); // 需要放入本地数据库

import {LangContext} from '../context/LangContext.jsx'; // 引入LangContext


class PageContainer extends React.Component {
   
    constructor(props) {
        super(props);
        this.state = {
            language: navigator.language
        };
    }
    // ...
    
    // 选择多语言
    chooseLocale(){
        const language = this.state.language;
        switch(language){
            case 'en-US':
                return en_US;
            case 'zh-CN':
                return zh_CN;
            default:
                return en_US;
        }
    }
    
    
    // 选择语言
    changeLanguage(language = 'zh-CN') {
        this.setState({
            language
        })
    }
    
    // ...
    render() {
        let { page, noBack, noHeader} = this.props;
        let { language } = this.state;
        // ...
        let tool = "";

        return <Page renderToolbar={() => tool} className="PageContainer">
            <IntlProvider locale={language} key={language} messages={this.chooseLocale()}>
                <LangContext.Provider value={{changeLanguage: this.changeLanguage.bind(this)}}>
                    {page?page:null}
                </LangContext.Provider>
            </IntlProvider>
        </Page>
    }
    
}
```

接着新建一个组件用来修改语言，创建组件的文件夹src/component，并且新建切换语言的组件 LanguageComponent.jsx ：

```jsx
import React from 'react';

import {LangContext} from '../context/LangContext.jsx' // 引入LangContext

export default class LanguageComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            showSelector: false
        };
        this.showLangSelector = this.showLangSelector.bind(this);
    }
    componentWillMount() {}

    // 显示语言选择选项
    showLangSelector() {
        this.setState({
            showSelector: true
        })
    }

    render() {

        const { showSelector } = this.state;

        return <LangContext.Consumer>
            {context => (
                <div>
                    {
                        showSelector
                            ?<div>
                                <div onClick={() => {context.changeLanguage('en-US')}}>英文</div>
                                <div onClick={() => {context.changeLanguage('zh-CN')}}>中文</div>
                            </div>
                            : <div onClick={this.showLangSelector}>
                                切换语言
                            </div>
                    }
                </div>
            )}
        </LangContext.Consumer>

    }
};
```

最后就是在需要使用切换多语言的地方引入组件即可。以注册页面举例：

SignInPage.jsx

```jsx
import React from 'react';

import PageContainer from '../container/PageContainer.jsx'; // 引入页面的容器
// ...

import {FormattedMessage} from 'react-intl';

import LanguageComponent from '../component/LanguageComponent.jsx'; // 引入组件

export default class SignInPage extends React.Component {
    constructor(props) {
        // ...
    }
    // ...
    render() {
        const { loginType } = this.state;
        const page = (<div className="SignInPage">
            <div className="sign-in-form">
                <LanguageComponent />
                <FormattedMessage
                    tagName="p"
                    id='intl.signin.great'
                    description='欢迎'
                    defaultMessage='{city} 欢迎你'
                    values={{
                        city: '厦门'
                    }}
                />
                // ...
            </div>
        </div>);
        return <PageContainer page={page} noBack={true} noHeader={true} />
    }
};

```

OK，现在可以尝试切换多语言了。



### 6.了解更多

关于react-intl 和 Context API 可以参考以下详细资料：

- [react-intl 实现 React 国际化多语言](https://juejin.im/post/59f96d7ef265da430f316997)

- [react-intl-api](https://github.com/yahoo/react-intl/wiki/API#react-intl-api)

- [React Context](https://reactjs.org/docs/context.html)
- [聊一聊我对 React Context 的理解以及应用](https://www.jianshu.com/p/eba2b76b290b)
- [How to update React Context from inside a child component?](https://stackoverflow.com/questions/41030361/how-to-update-react-context-from-inside-a-child-component)

