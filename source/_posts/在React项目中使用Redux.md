---
title: 在React项目中使用Redux
date: 2019-02-05 22:32:34
tags: [react, redux, react-redux]
toc: true
---


### 1.安装

紧接着上一次的实战项目，这次要来完状态管理，使用redux，首先需要安装一些包，执行以下命令安装：

` npm i redux react-redux redux-thunk --save`

redux 和 react并没有太大的关联， react-redux使得redux能和react更好地工作和。redux-thunk是redux的一个中间件，让我们能够直接访问dispatch方法，因此能够在action里面使用异步调用。

<!--more-->

### 2.使用Provider

需要讲所有的组件使用react-redux的provider嵌套起来，在App.jsx中引入provider：

```js
import { Provider } from 'react-redux';
```

使用 Provider  封装整个导出的组件，Provider 使用store属性来存储状态，因此给provider传递store，如下：

```jsx
    render() {
        return (
            <Provider store={store}>
                <Router>
                    <Switch>
                        <Route exact path="/" render={() => (<Redirect to="/exhibition"/>)}/>
                        <Route exact path="/sign-in" component={SignInPage}/>
                        <Route exact path="/exhibition" component={ExhibitionPage}/>
                    </Switch>
                </Router>
            </Provider>
        )
    }
```

### 3.导出store

接下来需要创建store，首先创建一个store文件夹，src/store，并且在这个目录创建一个store.js

文档 https://github.com/reduxjs/redux/tree/master/docs/api ，先看代码示例，store.js：

```js
// 从redux引入createStore，applyMiddleware，分别用来创建store和使用中间件
import { createStore, applyMiddleware } from 'redux';
// 引入thunk中间件
import thunk from 'redux-thunk';

// 引入根reducer
import rootReducer from '../reducer';

// 创建初始状态
const initialState = {};

// 创建一个中间件的数组
const middleware = [thunk];

// Arguments
// reducer (Function): A reducing function that returns the next state tree, given the current state tree and an action to handle.
//
//     [preloadedState] (any): The initial state. You may optionally specify it to hydrate the state from the server in universal apps, or to restore a previously serialized user session. If you produced reducer with combineReducers, this must be a plain object with the same shape as the keys passed to it. Otherwise, you are free to pass anything that your reducer can understand.
//
//     [enhancer] (Function): The store enhancer. You may optionally specify it to enhance the store with third-party capabilities such as middleware, time travel, persistence, etc. The only store enhancer that ships with Redux is applyMiddleware().
//
// Returns
// (Store): An object that holds the complete state of your app. The only way to change its state is by dispatching actions. You may also subscribe to the changes to its state to update the UI.
const store = createStore(rootReducer, initialState, applyMiddleware(...middleware));

// 导出store
export default store;
```

上面的示例已经导出了store，然后在App.jsx引入store：

```js
import { Provider } from 'react-redux';
import store from './store/store';
```

使用createStore(reducer, [preloadedState], [enhancer])创建store， createStore接收三个参数：

- reducer —— 根reducer，可以根据资源创建其他reducer如：postReducer、todoReducer等，然后把这些reducer合并到根reducer并传到store。
- preloadedState——就是初始的state。
- enhancer——扩展，比如使用中间件middleware。

### 3.创建reducer

#### 3.1.创建根reducer

创建一个目录reducer，并且在里面创建一个index.js，然后在上面写的store.js里面把这个reducer引入，

```js
// 导入combineReducers，用来合并reducer
import { combineReducers } from 'redux';

// 从各个reducer文件导入reducer
import animeReducer from './animeReducer';

// 导出reducer 并且为它们都设置一个命名，如：animeReducer命名为anime

export default combineReducers({
    anime: animeReducer
});
```

在rootReducer里面做的事情很简单，就是合并其它的reducer并且把合并的结果导出去。

#### 3.2.创建各个reducer

在创建reducer之前，先来创建type，type内部基本上就是常量，创建一个action目录，src/action，并在里面新建一个文件type.js，代码如下：

```js
// 我们创建的action实际上包含了type，type基本上都是常量。
// 导出定义的action
export const FETCH_ANIME = 'FETCH_ANIME';
```

在reducer目录新建 animeReducer.js

```js
// 从types.js 导入 type
import { FETCH_ANIME } from "../action/type";

// 定义初始state
const initialState = {
    items: [] // 这里的items就代表着从action传来的数据列表，而在cation里面会去获取数据。
};
// 导出一个方法，接收两个参数，第一个就是初始状态，第二个是action，action是一个对象，包含type属性
export default function (state = initialState, action) {
    // 判断action的type，导出state，默认导出初始state
    // 如果type是FETCH_ANIME，则将返回初始state与action返回的的payload的数据
    // payload用来表示返回数据，这些数据会在action里面通过调用dispatch方法返回，关于action会紧接着在后面介绍。
    switch (action.type) {
        case FETCH_ANIME:
            return {
                ... state,
                items: action.payload
            };
        default:
            return state;
    }
}
```

###4.创建action

在src/action目录里面，可以创建action文件，animeAction.js

```js
// 引入type
import { FETCH_ANIME} from "./type";

// 导出一个方法，获取影片
export  function fetchAnime() {
    // 里面再导出一个方法，接收dispatch, dispatch会将获取的数据传到reducer
    return function (dispatch) {
        fetch('https://ghibliapi.herokuapp.com/films')
            .then(res => res.json())
            .then(anime => dispatch({
                type: FETCH_ANIME,
                payload: anime
            }))
    }
}
```

```js
// 简写
export const fetchAnime = () => dispatch => {
    fetch('https://ghibliapi.herokuapp.com/films')
        .then(res => res.json())
        .then(anime => dispatch({
            type: FETCH_ANIME,
            payload: anime
        }))
}
```

每个action都是一个function，react-thunk中间件使得能够直接访问dispatch方法，因此在action里面可以使用异步请求，使用dispatch可以向reducer发送数据。

### 5.在组建中使用

```jsx

import React from 'react';
import PropTypes from 'prop-types';

// 引入connect来使被provider包裹的react组件连接到redux的store
import { connect } from 'react-redux';
// 引入请求数据的action
import { fetchAnime } from '../action/animeAction';

class AnimePage extends React.Component {
    constructor(props) {
        super(props);
        this.state = {};
    }

    // 组件将要加载
    componentWillMount() {
        this.props.fetchAnime();
    }

    // 组件挂载完毕
    componentDidMount() {

    }

    render() {
        return <div className="AnimePage">{
            this.props.anime.map(item => <div key={item.id}>
                <h3>
                    {item.title}
                </h3>
                <p>
                    {item.description}
                </p>
            </div>)
        }</div>
    }
}
// 定义PropTypes
AnimePage.propTypes = {
    fetchAnime: PropTypes.func.isRequired,
    anime: PropTypes.array.isRequired
};

// 创建一个方法将redux的state转换成props
const mapStateToProps = state => ({
    // 这里使用的state.anime 是在 reducer/index.js 文件中的 根reducer里面定义的名称。
    anime: state.anime.items
});

export default connect(mapStateToProps, { fetchAnime })(AnimePage);
```
