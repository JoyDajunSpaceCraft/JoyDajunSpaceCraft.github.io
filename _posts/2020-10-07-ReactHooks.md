---
layout: post
title: ' React Hooks '
subtitle: ' react '
date: 2020--09-30
author: 'Joy'
header-img: 'img/react-hook.jpg'
tags:
  - react hooks
  - self-learning
---
## React 的class-based  和 functional-based 的component
在实现时，functional-based可以实现class-based功能，一般情况下lifecycle可以用hook代替
+ 命名规则 useXYZ()
+ 只在react 16.8以上版本能用
## 开始使用

### 使用样例，通过输入ingredient的载入ingredient的描述和数量，并且能点击下面的IngredientList 删除 ingredient
+ **导入useState**,这个相当于class-base中的`state = {}`不同之处在于useState可以实现传入各种类型的数据，其中有两个参数，一个是当前state的 snapshot 另一个就是如何更新这个state
+ 使用ES6的extract方法实现的state赋值，`const [inputState,setInputState]=useState({ title: '', amount: '' });
`
+ 也可以替换成以下的形式
```
const [enteredTitle, setEnteredTitle] = useState("");
const [enteredAmount, setEnteredAmount] = useState("");
```
+ 怎样use hooks 
  + 必须是在function component或者自定义hook中使用
  + 必须在root level使用hook，不能再一个function component中嵌套使用 hook，也不能在if语句中使用hook


+ fetch Data from firebase
通过创建新的real time firebase 使用`fetch()`是一种browser能识别的代码表示从http中获取数据。
**小知识点** 
  + json => js code `jsonCode.json()`
  + js code => json `JSON.stringify(jsCode)`

```
  fetch("https://react-hook-example-xxxee.firebaseio.com/ingredient.json",//样例URL
      {
        method: "POST",
        body: JSON.stringify(ingredient),
        headers: { 'Content-Type': 'application.json' }
      }).then(response => {
        return response.json()
      }).then(responseData => {// responseData是response 中的属性名 包含 firebase中赋予的id
        setIngredients(prevIngredients => [...prevIngredients,
        {
          id: responseData.name,// responseData来源于firebase
          ...ingredient
        }
        ])
      })
```

这里使用两个then因为return response.json()返回的是一个promise所以对其内部做处理需要再加一个then。第一个then获得的数据如下图
![](/img/react-hook-fetch.jpg)

+ **useEffect hook**
这个hook表示在页面每一次render完成之后再加载的动作，用于消除**side effect**，没有加入`[]`依赖，相当于`componentDidUpdate`
传入一个匿名函数以及函数中变量用到的依赖，只有这个依赖改变，才会rerun这个页面。所以加上`[]`表示`componentDidMount`所以***一定要加[],哪怕没有依赖也要加***
**注意，如果是因为在props中的传入的function改变，需要加入depend中需要转化格式**：

```
const {propsValue} = props;
...
useEffect = (()=>{...},[propsValue]);
```

### 案例分析，如何给页面加上一个可搜索的方法？

+ 首先需要对useEffect 中设置fetch保证每次输入都能匹配数据库中的某个固定字段
+ 在数据库（后端）配置这个字段，也就是firebase中在rules中将这个地方加上传入数据库的字段，以下内容表示在ingredient传入时也就是fetch的url最后加上的`ingredient.json`中post的title字段为搜索中需要修改的地方。
```
{
  "rules": {
    ".read": "now < 1605024000000",  // 2020-11-11
    ".write": "now < 1605024000000",  // 2020-11-11
      "ingredients":{
        ".indexOn":["title"]
      }
  }
}
```
在useEffect中写入需要到数据库中匹配的字段，每次输入的值是通过html中onChange执行的。

在以下的代码中注意query字段，表示判断输入值是否为空，如果不为空就需要到数据库中取数据。

```
useEffect = (() => {
    const query = enteredFilter.length === 0 ? '' 
    : `?orderBy="title"&equalTo="${enteredFilter}"`;//注意不要丢到=号

    fetch("https://react-hook-example-xxxee.firebaseio.com/ingredient.json" + query)
      .then(response => {
        return response.json();
      })
      .then(responseData => {
        const loadResponseData = [];
        for (const key in responseData) {
          loadResponseData.push({
            id: key,
            title: responseData[key].title,
            amount: responseData[key].amount
          });
        }
        onLoadingIngredients(loadResponseData);
      })

  }, [enteredFilter,onLoadingIngredients]);//每次enteredFilter改变就会执行
```

**onLoadingIngredients这个依赖**是调用component时赋给的一个props，已经通过`const {propsValue} = props;`取出，表示如果这个component被重新调用就再次调用useEffect,parant component re-render

### **useCallback hook**
作用于会导致两个component一直渲染的页面，因为子页面利用父页面传过来的方法作为useEffect的依赖，只要useEffect依赖改变，子页面也就会重新渲染，但是父页面中传入到子页面的方法在被子页面调用之后，在父页面进行了重新渲染，从而带动子页面的渲染形成闭环，useCallback就是保持父页面中的内容没有因为子页面重新渲染而改变，作为一个cache。

useCallback 用法和useEffect相同，也需要加上依赖参数，如果是useState作为依赖参数则可以忽略不加。

### **useRef hook**
useRef是为了获得最新的state中的数据，在useEffect中setTimeout时希望判断两次输入时间间隔不小于多少秒，而且需要一边输入一边对比，就需要用到useRef。setTimeout内部获得的useState表示在运行到setTimeout时获得的state，但对于一直更新的state需要通过useRef获得。
使用方式如下
  + 先引入useRef  `import {useRef} from 'react'`
  + 定义一个useRef `const inputRef = useRef()`
  + 在html处定义ref `<input ref={inputRef}/>`
  + 在setTimeout处使用ref 
  ```
  setTimeout(() => {
    if (enteredFilter === inputRef.current.value) {
      ...
    }
  ```
  主要用于解决用户输入时为了不要一直获取用户输入而频繁的render到后端的问题。

### **useEffect的cleanUp**
在上面的例子中，如果用户输入时间过长但是一直匹配到正确结果的情况下，需要clean useEffect的使用，（setTimeout 是在useEffect中的）所以每次depend改变需要re-render时，才执行return，也就是clean
具体使用方式如下: return 一定是一个func，当depend为空也就是`[]`时，return发生时间变为component为unmount时。

```
useEffect(() => {
    const timer = setTimeout(() => {
      if (enteredFilter === inputRef.current.value) {
        ...
      }
    },5000);
    return ()=>{cleanTimeout(timer)};
},[]);
```

return中执行的就是clean useEffect的方法。

***小知识点***
+ 在代码中的三段表达式：` A ? B: C`如果C是null则可以写成`A && B`形式
+ 在代码中两个setState一起执行不会导致死循环情况，因为state会预先存在batch中，原先的会保留并且在同一个render中执行，两个state同时执行，并且有交叉关系再执行时是没有冲突的。对于两个setState同时执行的情况<a href="https://github.com/facebook/react/issues/10231#issuecomment-316644950" target="_blank">github issue</a>
具体解释如下:

#### more on state batching && state update
```
setName('Max');
setAge(30);
```
in the same synchronous (!) execution cycle (e.g. in the same function) will NOT trigger two component re-render cycles.

Instead, the component will only re-render once and both state updates will be applied simultaneously.

Not directly related, but also sometimes misunderstood, is when the new state value is available.

Consider this code:

```
console.log(name); // prints name state, e.g. 'Manu'
setName('Max');
console.log(name); // ??? what gets printed? 'Max'?
```

You could think that accessing the name state after setName('Max'); should yield the new value (e.g. 'Max') but this is NOT the case. Keep in mind, that the new state value is **only available in the next component render cycle** (which gets scheduled by calling setName()).

Both concepts (batching and when new state is available) behave in the same way for both functional components with hooks as well as class-based components with this.setState()!

### **useReducer**
类似于reducer的一种方式，因为定义了大量的useState，所以可以将多个reducer整合。
+ 首先在component外层定义useReducer，因为每次外层的定义是不会随着component一起render的。useReducer和基础的redux的reducer定义类似。

```
const ingredientReducer = (currentIngredients, action)=>{
  switch(action.type){
    case "SET":
      return action.ingredient
    case "ADD":
      return [...currentIngredients, action.ingredient]
    case "DELETE":
      return currentIngredients.filter(ing =>ing.id !== action.id)
    default:
      throw new Error("should not go hear!")
  }
}

const ExampleComponent = ()=>{...}
```
每个action后面跟的就是在调用dispatch时需要传入的参数

+ 在component内部导入刚刚定义的reducer 
```
const ExampleComponent = ()=>{
  const [ingredients, dispatch] = useReducer(ingredientReducer,[])
}
```

传入的两个参数第一个是前面定义的array第二个是start state，注意从useReducer中提取的第二个参数，是用来代替setState方法的，名字可以自行定义。

+ 代替setState，以下是几个state案例

**ADD**
```
// setIngredients(prevIngredients => [...prevIngredients,
  // {
  //   id: responseData.name,
  //   ...ingredient
  // }
  // ])
  dispatch({
    type: "ADD", ingredient: {
      id: responseData.name,
      ...ingredient
    }
})
```

**DELETE**

```
// setIngredients(prevIngredients => prevIngredients.filter(
//   (ingredient) => ingredient.id !== ingredientId))
dispatch({type:"DELETE",id:ingredientId})
```

**SET**

```
const filterIngredientsHandler = useCallback((filterIngredients) => {
    // setIngredients(filterIngredients)
    dispatch({ type: "SET", ingredients: filterIngredients });
  }, [])
```

**综上，useReducer可以使用的情况是在每个state有不同的使用状态时，或者一个方法组要改多个state时，放在一起定义。**

### **useContext**
用法是将需要层层传递的某个state利用这种形式获得，类似于redux但是是react自带的方法，主要用于function base component中详见react笔记那篇文章。

```
import React, { useState } from 'react';
export const AuthContext = React.createContext({
isAuth: false,
login: () => {}
})

const AuthContextProvider = (props) => {
    const [isAuth, setIsAuth] = useState(false);
    const loginHandler = () => {
        setIsAuth(true)
    }
    return (
        <AuthContext.Provider value={{login:loginHandler,isAuth:isAuth}}>
            {props.children}
        </AuthContext.Provider>
    )
}
export default AuthContextProvider;
```

在index.js中利用`<AuthContextProvider>`包裹住`<App />`表示所有需要用到AuthContext地方被表示为Provider,在需要用到的是否login的地方写入以下代码（此处是在App主界面），注意定义中先export的并不是default的`AuthContextProvider`而是包含默认值的`AuthContext`所以在导入的时候是`{AuthContext}`

```
import { AuthContext } from './context/auth-context';   

const App = props => {
const authContext = useContext(AuthContext);
let content = <Auth />
if (authContext.isAuth) {
    content = <Ingredients/>
}
return (
    // <AuthContext.Consumer>
    // <Auth />
    // </AuthContext.Consumer>
    {content}
    )
};
```

在需要auth的界面中显示
```
import {AuthContext} from '../context/auth-context';

const Auth = props => {
const authContext = useContext(AuthContext);
const loginHandler = () => {
    authContext.login();//改变login中的state从而在App主界面转换为<Ingredients/>
};
```

### **useMemo**
在react笔记中介绍过`React.memo`的用法，这里是利用useMemo进行扩展。原来是作用在component定义中，useMemo是直接作用在component调用的时候。传入的是一个方法，作用是保存一些需要花费时间渲染的component，只有在其中有值改变的时候才回去渲染。
```
const ingredientList = useMemo(() => {
  return (
    <IngredientList ingredients={ingredients} onRemoveItem={removeIngredientHandler} />
  )
}, [ingredients])//依赖
...
在render中
{ingredientList}
```

如果是用到`React.memo`则是
```
const IngredientList =React.memo(props => {
  return (...)})
```

## 自制Hook
最重要一点需要以use作为开头

自制的Hook首先是一个匿名方法，对渲染的优先顺序可以使用useCallback来限制

## 将hook作用到实际的开发中
+ 首先利用React.lazy()代替原先的自定义懒加载，毕竟原先的代码中包含了class base的component，具体代码见react-axios的那篇文章。

+ 和React.lazy包裹搭配使用的是`<Suspend>`，是对route中在没有实现加载时在页面上显示的内容，具体是:
```
<Suspense fallback={<p>Loading...</p>}>
        {routes}
</Suspense>
```
包裹住自定义的route，并且将router修改为render跳转而不是component跳转>
+ 如果是要执行`componentWillMount`表示在component加载之前执行，所以把代码前移，放在return之前。
+ `componentWillUnmount`作为放在最后作为页面加载完毕清除缓存的控件可以利用useEffect的return实现。
