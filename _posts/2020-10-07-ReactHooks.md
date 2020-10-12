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

