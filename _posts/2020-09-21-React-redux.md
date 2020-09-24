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
  - spinner
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

并且写入midware的定义方法
```

```
 


