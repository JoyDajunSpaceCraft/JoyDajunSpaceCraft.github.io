---
layout: post
title: ' Redux详解 '
subtitle: 'react practice'
date: 2020--09-21
author: 'Joy'
header-img: 'img/about-bg3.jpg'
tags:
  - react
  - self-learning
  - saga
  - redux
---


### what is state?
influence what I look in the screen 

### 需要了解的Action Reducer
+ reducers: pure function,receive action and update state. 
+ action: predefine information package
其实Redux不光可以在React中使用，在只有node的情况下，也可以通过设置初始化的state。
Redux的意义是实现各个component之间参数传递，传统的props如果要传递参数需要写入很多重复代码，Redux相当于一个云数据库，存储了许多state。


### Redux中主要组成部分


+ Reducer 初始化state 绑定action
```
const initialState = {
    counter: 0
}
const rootReducer = (state = initialState, action) => {
    if (action.type === 'INC_COUNTER') {
        return {
            ...state,
            counter: state.counter + 1
        };
    }
    if (action.type === 'ADD_COUNTER') {

        return {
            ...state,
            counter: state.counter + action.value
        };
    }
    return state;
}
```

+ Store 定义需要保存的对象
```
const store = createStore();
```

+ Subscription 确认所有的绑定的action执行之后的结果
```
// 不需要manually called  console.log(store.getState)
store.subscribe(() =>{
    console.log('[Subscribtion]', store.getState());// 
})// 每次dispatch的实现都能通过getState获得
```

+ Dispatching action 对每个action实现的具体内容，type是一定要定义的，利用type来绑定reducer中对state执行怎样的操作
```
store.dispatch({ type: 'INC_COUNTER' })
store.dispatch({ type: 'ADD_COUNTER', value: 10 }) // 这里就是传递了value作为参数。
```

**总结** reducer通过dispatch定义的action的type对state进行操作，而store是能获得改变之后state的对象，可以通过调用store来获得改变之后的state，subscribe是可以观察到state改变状态的store里的方法。

### 创建react-redux步骤

+ 在index.js中初始化redux

`import { createStore } from 'redux';`


在主界面中定义redux-base.js
实现代码
首先要加载属于react的redux
`npm install --save react-redux`

安装时遇到问题 对应最新版本的react-redux安装不支持，需要在package.json中将其版本号换成**5.0.6**并且对应的react版本应该小于**16.3**

```
import { Provider } from 'react-redux';
const store = createStore(reducer)
```


在index.js中将`<Provider store={store}><App /></Provider>` 作为Component外层包裹到App中，并且传入定义好的store

引入connect概念和mapStateToProps()
connect 是高阶组件，但是其引入方式是
```
import {connect} from 'react-redux';

...

export default connect(mapStateToProps,mapDispatchToProps)(XXX);
```

### mapStateToProps 和 mapDispatchToProps
整体上redux的参数传入机制是依靠这两个来实现

+ **mapStateToProps** 类似于在class component 中定义初始化state

```
state = {
  counter: 0
} //正常格式

// 处于class component之外的格式
const mapStateToProps = state => {
    return {
        ctr: state.counter
    }
}
```

+ **mapDispatchToProps** 定义获得的state通过什么type的dispatch

##### 在class component中实现的形式

```
counterChangedHandler = (action, value) => {
        switch (action) {
             case 'inc':
                this.setState((prevState) => { return { counter: prevState.counter + 1 })
      ...
   
```

这两个的使用顺序是固定的，在`connect(mapStateToProps,mapDispatchToProps)()`如果没有mapStateToProps则要变为`connect(null,mapDispatchToProps)()`同理，没有mapDispatchToProps就默认connect中只传入一个参数。

#### 在class component外实现的形式

```
const mapDispatchToProps = dispatch => {
    return {
        onIncreamentCounter: () => dispatch({type:"INCREMENT"}),
        onDecrementCounter:() =>dispatch({type:"DECREMENT"}),
        onAddCounter: () => dispatch({type:"ADD", value:5}),
        onSubtractCounter: () => dispatch({type:"SUBTRACT", value:5})
        
    };
}
```

#### 在redux文件中定义的格式

```
const initialState = {
    counter: 0
}

const reducer = (state = initialState, action) => {
    switch (action.type) {
        case ('INCREMENT'):
            return {
                ...state,
                counter: state.counter + 1
            };
            break;
        case ('DECREMENT'):
            return {
                ...state,
                counter: state.counter - 1
            };
            break;

        case ('ADD'):
            return {
                ...state,
                counter: state.counter + action.value
            };
            break;

        case ('SUBTRACT'):
            return {
                ...state,
                counter: state.counter - action.value
            };
            break;
    }

    return state;
};
export default reducer;
```

### redux 传参
playload作为参数传递于需要的component和reducer之间，对于需要在dispatch中传入参数的函数，可以通过在定义type属性之后添加进行定义。

如果想要在不同的class component中动态传参其实需要props的配合以及在dispatch中也放入对应参数来传递state。

### 整合多个reducer
首先定义多个reducer为一个模块，在此基础上，在index.js文件中，对其中的引入redux模块代码进行添加
`import { createStore, combineReducers } from 'redux';`

在index中写入
```
import counterReducer from './store/reducers/counter';
import resultReducer from './store/reducers/result';

const rootReducer = combineReducers({
    ctr:counterReducer,
    res:resultReducer
})
const store = createStore(rootReducer)
```
表示告知react 整合这两个模块，并且将新的rootReducer传入到store中，对应到具体的component中，不同的reducer对应的state也改变了，所以需要再重新加上不同的模块，这里定义state的地方在mapStateToProps中，在原先的state的基础上重新加上.新定义的reducer名。 
```
const mapStateToProps = state => {
    return {
        ctr: state.ctr.counter,
        storeResult: state.res.results
    }
}
```




### 什么情况下使用redux来存储state
1. type : local UI 如 hide/show backdrop 不需要
2. type: persistant state 如 users info 不需要，是存储于server端的
3. client state: 用户是否登录，需要


## Advance Redux
高级redux
### Add midware 
什么是midware 在reducer和action中间的一段代码或者说是function

#### 导入midWare
在index.js中导入

```
import { createStore, combineReducers, applyMiddleware } from 'redux';
```

并且写入midware的定义方法(定义在index.js中)
```
// 自己创建midware 监视每次dispatch之后state的值
const logger = store=>{
    return next =>{
        return action =>{
            console.log('[MidWare] Dispatching', action);
            const result =  next(action);
            console.log('[Midware] next state', store.getState())
            return result;
        }
    }
}
```

实现midWare注册，在index.js中因为已经导入`applyMiddleware`所以在createStore时多传入参数`applyMiddleware`。
`const store = createStore(rootReducer, applyMiddleware(logger))`

可以看到`applyMiddleware`传入刚刚定义的midWare，也就是`logger`，**其实可以传入多个midWare**

#### 使用redux dev tools
google安装插件，打开这个github页面<a href="https://github.com/zalmoxisus/redux-devtools-extension" target="_blank"> redux dev tools</a>

定位到![](/img/advance-redux.jpg)这个位置，修改`rootRedux`
改变为
```
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
const store = createStore(rootReducer, /* preloadedState, */ composeEnhancers(applyMiddleware(logger )))
```
并且在`import redux`的时候加上`compose`， `compose`和`combineReducers`是一个类似用法，将多个enhance聚合。在redux tool中可以看到对应redux数据流。

### 为什么redux需要使用async code
在action中传入的方法如果需要设置时间属性，比如setTimeout会返回一个Promise，但是redux中不支持使用Promise所以需要在其中调用async

#### 开始实现asynchron code 
+ 首先修改action.js中的内容，原先的action中使为了防止输入出错，所以const export 的大写字母，但现在可以将其和所有的dispatch整合

```
export const subtract = (value) =>{
    return {
        type:SUBTRACT,
        value:value
    }
}
```

大写的变量变成方法，在mapDispatchToProps的时候return的dispatch就由只能传入`dispatch => ({type:...})`变成`dispatch =>(action.js中定义的各个方法名)`，其实是基本的参数变化。

+ 载入**redux thunk**
`npm install --save redux-thunk`
在createStore的时候再加上一个midWare

`const store = createStore(rootReducer, composeEnhancers(applyMiddleware(logger, thunk)));`

在action中使用setTimeout()包裹住一个dispatch，之后通过主界面调用`storeResult`中的方法实现async的dispatch
```
export const saveResult = ( res ) => {
    return {
        type: actionTypes.STORE_RESULT,
        result: res
    };
}

export const storeResult = ( res ) => {
    return dispatch => {
        setTimeout( () => {
            dispatch(saveResult(res));
        }, 2000 );
    }
};
```

#### 使用midWare之后怎样修改代码中的逻辑

 action |reducer
-- |:--:
can run Async code| pure Sync code **Only**
should not prepare the state update too much| core redux concept : reducers update the state

所以需要修改逻辑的代码大都放置在reducer中

#### use action creators 和 getState
在进行dispatch的async中，可以使用getState
```
export const storeResult = ( res ) => {
    return (dispatch, getState) => {
        setTimeout( () => {
            const oldCounter = getState().ctr.counter;
            dispatch(saveResult(res));
        }, 2000 );
    }
};
```
这个可以在进行dispatch之前将原先的state获取，注意加入的getState参数是为了可以获取state，而且要加上对应定义state的reducer在App.js中的注册名。 **注意，不推荐多次使用getState获得redux，尽量使用传参**

### 使用自定义的utility简化reducer中代码
```
const updateObject = (oldObject, updatedValues) => {
    return {
        ...oldObject,
        ...updatedValues
    }
}
// 调用时是
updateObject(state,{counter: state.counter - 1})
```

action reducer 

### 实际应用redux的注意事项
通过redux来判定页面上哪些内容应该显示，但是有时因为调用的dipatch方法执行没有render快，所以导致时间延迟比如，希望在页面中传入某个在render执行之后就马上执行的dispatch，但是在其后的调用中发现这个dispatch加载比render慢，因为这个dispatch是执行在`componentWillMount`中，但是比render慢，所以无法达到要求，可以通过在上一个界面中执行这个dispatch实现相同效果，不会比render慢。


### Saga 使用
redux-saga是帮助旧版的react的async的action做出监听， handle all side effection on action, don't directly manipulate the redux store具体的实现步骤如下
### 在store文件夹下创建saga文件夹匹配/auth.js

**创建Generater**也就是`function*`，这里是为了在logout时清除localStorage。

在actions/auth.js的logout中的原始code是

```
export const logout = () => {
    localStorage.removeItem('token');
    localStorage.removeItem('expirationDate');
    localStorage.removeItem('userId')
    return {
        type: actionTypes.AUTH_LOGOUT,
    }
}
```

在经过saga时变为
```
import { put } from 'redux-saga/effects'; // dispatch new action 
import axios from 'axios';
export function* logoutSaga(action) {
    // generate
    yield localStorage.removeItem('token');
    yield localStorage.removeItem('expirationDate');
    yield localStorage.removeItem('userId');
    put({ type: actionTypes.AUTH_LOGOUT})
}
```

+ put表示dispatch新action，注意这个put如果要dispatch action 中的function时需要加上()表示实现这个function eg.`put(actions.logout())` 
+ yield表示只有在这一步执行完成之后下一步才开始执行
+ yield + put 想当于 async 的 dispatch
+ 此外`redux-saga/effects`还有属性 delay传入的是时间，表示经过多长时间之后才执行yield之后的code，具体使用形式
`yield delay(action.expirationTime);`
**delay是millionsecond所以需要注意在axios中expireIn是second**

+ `redux-saga/effects`有call 属性，使generator更加可测试，如原先的删除localStorage
`yield localStorage.removeItem('userId');`通过 call 可以变成`yield call ([localStorage, "removeItem"],"userId")`
+ `redux-saga/effects`有all 属性使用时减少`yield`使用
```
yield takeEvery(actionTypes.AUTH_INITITATE_LOGOUT, logoutSaga);
yield takeEvery(actionTypes.AUTH_CHECK_TIMEOUT, checkoutTimeoutSaga);
yield takeEvery(actionTypes.AUTH_USER, authUserSaga);
yield takeEvery(actionTypes.AUTH_CHECK_STATE, authCheckStateSaga);
```
在all属性下转换为
```
yield all([
    takeEvery(actionTypes.AUTH_INITITATE_LOGOUT, logoutSaga),
    takeEvery(actionTypes.AUTH_CHECK_TIMEOUT, checkoutTimeoutSaga),
    takeEvery(actionTypes.AUTH_USER, authUserSaga),
    takeEvery(actionTypes.AUTH_CHECK_STATE, authCheckStateSaga)
])
```
可以对几个generator simultanously
+ `redux-saga/effects`有takeLatest属性，表示只获取最新的saga而不是每一次saga都要检测到。
+ 更多API详见<a href="https://redux-saga.js.org/" target="_blank">redux-saga</a>


### 配置saga

将创建好的saga配置入已经弄好的action中，此项目在在index.js中定义了action

```
import createSagaMiddleware from 'redux-saga';
import { logoutSaga } from './store/sagas/auth';
```

register saga,在原有的thunk midware基础上加上了sagaMiddleware并且注册了sagaMiddleware的run
```
const sagaMiddleware = createSagaMiddleware();
const store = createStore(rootReducer, /* preloadedState, */ composeEnhancers(
    applyMiddleware(thunk, sagaMiddleware)
));
sagaMiddleware.run(logoutSaga)
```

### 监听action
 因为已经出现了logout这样的方法并且执行，可能会出现执行重复的情况，所以利用saga的特性实现对logout这个action的监听。在此之前，将原先的actions/auth.js中的logout中的return `actionType`变为`AUTH_INITITATE_LOGOUT`，**通过绑定在saga中定义的Generate和action实现的`actionTypes.AUTH_CHECK_TIMEOUT`实现绑定。**

建立sagas/index.js文件夹保存对saga的监听
```
import {takeEvery} from 'redux-saga/effects'; // listen certain action 

import * as actionTypes from '../actions/actionTypes';
import {logoutSaga} from './auth';

export  function* watchAuth () {
    yield takeEvery(actionTypes.AUTH_INITITATE_LOGOUT, logoutSaga);// 每次执行这个generation都会监听AUTH_INITITATE_LOGOUT
}
```
每次执行这个generation都会监听`AUTH_INITITATE_LOGOUT`从而实现由AUTH_INITITATE_LOGOUT到AUTH_LOGOUT的绑定。


同时在index注册时将`sagaMiddleware.run(logoutSaga)`变为`sagaMiddleware.run(watchAuth)`

**注意，在注册saga的地方可以定义多个sagaMiddleware**

+ `takeEvery`是能够监听每一次saga执行的控件，每次点击saga

### 什么时候应该使用saga
redux本质是创建一个store保存component中共有的数据，定时删除localStorage和从服务器获取数据这样的要求其实是redux的side effect，所以要对这样的方法做处理就引入Saga。