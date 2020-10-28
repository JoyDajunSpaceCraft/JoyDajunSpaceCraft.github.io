---
layout: post
title: ' React burger 页面axios、Router '
subtitle: 'react practice'
date: 2020--09-18
author: 'Joy'
header-img: 'img/react-axios1.jpg'
tags:
  - react
  - self-learning
  - no class basing
  - axios
  - router
  - Do not go gentle into that good night.
---



# 首先了解 axios 和 firebase 是什么

+ axios 是一种 ajax 传参方式，导入的形式是`npm install --save axios`

+ firebase 是google开发的数据库，代码中需要和数据库交互的部分放入firebase中

#### axios使用的方式可以是

在`import axios from 'axios'`之后

```
 componentDidMount() {
        axios.get('https://jsonplaceholder.typicode.com/posts')
            .then(response => {
                const posts = response.data.slice(0, 4);
                const updatePosts = posts.map(post => {
                    return {
                        ...post,
                        author: 'Max'
                    }
                })
                // console.log("response", response)
                this.setState({ posts: updatePosts })
            })
            .catch(error => {
                console.log(error)
                // this.setState({ error: true })
            })
    }
```

以上代码表示，在 component 完成整个页面渲染之后，通过 get 请求获得页面数据

.then 代表页面进行不是线性的，在等待 axios 执行完成之后再执行 then 中的内容，response 返回的请求体，其中返回的是 get 到的内容。.catch 表示的是获取的错误信息，

```
{data: Array(100), status: 200, statusText: "OK", headers: {…}, 
config: {url: "/posts", method: "get", headers: {…}, baseURL: "http://jsonplaceholder.typicode.com", transformRequest: Array(1), …}
data:body: "quia et suscipit↵suscipit recusandae consequuntur expedita et cum↵reprehenderit molestiae ut ut quas totam↵nostrum rerum est autem sunt rem eveniet architecto"
id: 1
title: "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
userId: 1
headers: {cache-control: "max-age=43200", content-type: "application/json; charset=utf-8", expires: "-1", pragma: "no-cache"}
request: XMLHttpRequest {readyState: 4, timeout: 0, withCredentials: false, upload: XMLHttpRequestUpload, onreadystatechange: ƒ, …}
status: 200
statusText: "OK"
```
也可以通过设置create属性的方式对所有的请求设置baseURL这样就可以不用对所有请求都加上同一种url，之后的请求写法就是`axios.get('/post')`

#### 详细的baseURL使用方式如下
```
const instance = axios.create({
    baseURL:'http://jsonplaceholder.typicode.com'
})
instance.defaults.headers.common['Authorization'] = 'AUTH TOKEN from Instance'//给每个header上都会加上一个请求头的默认值
```
其中的headers.common 给这个请求所有的headers都会加上一个 默认的Authorization
```
headers:
Accept: "application/json, text/plain, */*"
Authorization: "AUTH TOKEN from Instance"
```

不只是在headers的commons中，在发送post请求时可以指定接收数据类型`['Content-Type'] = 'application/json';`

#### axios拦截器的使用

拦截器是axios的一种使用方式，表示对获得的response和发出的request做拦截，使用方式是
```
axios.interceptors.request.use(request => {
    console.log(request);
    //Edit request
    return request;//一定要return 否则不能在component中使用
}, error => {
    console.log(error);
    return Promise.reject(error)
})
```
以上代码表示在request中对获得请求头进行处理，不使用return request会导致后面的请求无法收到request所以一定要return。

对于error的处理是 `return Promise.reject(error)`实现，其中Promise是ES6中基本数据类型，会有reject和resolve两个状态，是用来实现异步操作调用的方法。

为了不使interceptor在内存中堆积，要使用eject 对缓存中的interceptors进行清除。 commponentWillMount 和 componentWillUnmount是开始和结束页面渲染标志。因为用了this.来存储了interceptors所以后面删除。
```
componentWillMount () {
            this.reqInterceptor = axios.interceptors.request.use(req => {
                this.setState({error: null});
                return req;
            });
            this.resInterceptor = axios.interceptors.response.use(res => res, error => {
                this.setState({error: error});
            });
        }

componentWillUnmount() {
            axios.interceptors.request.eject(this.reqInterceptor);
            axios.interceptors.response.eject(this.resInterceptor);
        }
```

### 使用route来实现不同URL页面之间相互的传递参数。
 
route是react-router-dom中的实现方法，通过在App.js中引入`import { BrowserRouter, Route } from 'react-router-dom';`实现。

在主程序的return中用`<BrowserRouter></BrowserRouter>`包裹所有的App.js中需要包裹的代码，在导航的component中需要导入`import { Route } from 'react-router-dom'`注意如果要真确使用一定要在App.js中同样导入Route。
#### Route使用方式 

```
<Route path="/" exact render={()=><h1>home</h1>}/> 
<Route path="/"  render={()=><h1>home1 </h1>}/> 
<Route path="/" exact component={Posts}/>
```
`<Route>`中属性代表如下
  + exact: 表示path中的名称不是模糊的，如果没有exact那么所有path中包含"/" 的页面就都会有render之后的页面渲染效果。
  + path: 表示页面到哪里去
  + component: 表示页面render到一个component 
  + render: 返回一个方法，可以是jsx语句，如果要传入参数，需要再加上`<Route path="/auth" component={(props)=><Auth {...props}/>} />`保留props


#### 如何防止每次跳转页面不重新加载js

在导航的component中需要导入`import { Route，Link } from 'react-router-dom'`
使用Link代替`<a>`html语言，使用
`<Link to='/'>`代替了原本的`href='/'`
使用方式加强版

```
<Link to={{
            pathname:'new-post',
            hash:'#submit',
            search:'?quick-sumbit=true'
      }}>New Post</Link>
```
+ 将pathname设置为new-post 表示绝对路径，如果想要显示相对路径  `this.props.match.url + '/new-post'`表示相对路径
+ hash表示可以跳转到页面的摸个部分 
+ search表示可以对这个页面的某个地方实现查找，都是后话了。

**这里的知识点就是 如果将参数传入为search形式，在后面怎样读取出来**
+ 首先理解for of 概念 也就是一种for each 循环形式
`for (let i of arr){...}`
+ 在后面调用的地方使用的是
`const query=new URLSearchParams(this.props.location.search)` 表示通过 `URLSearchParams`将默认存储在location中的search提取。
+ URLSearchParams暴露一个entries方法可以获得当时传入search中的内容,`quick-sumbit`是key`true`是value，最后会返回两个值`"quick-sumbit","true"`

原始获得search中内容代码如下：
```
componentDidMount(){
        const query=new URLSearchParams(this.props.location.search);
        console.log(query)// 显示是一个obj
        for(let param of query.entries()){
            console.log(param);// 暴露出来的 entries方法
        }
    }
```



***总结使用方法：***
  + **route首先定义了跳转也买的path，再用指定的Component表示跳转的模块；**

  + **link表示点击某按键实现页面的跳转，这个按键中的pathname要包括匹配route中的path**



#### 如何实现每次加载页面能够获得最近的一个router页面的信息


在之前使用了Link，表示component之间不重新渲染页面、不重新加载state后的传参魔法，Link到的component中props中有match等详细参数，表示上一个component的信息，但是对于不在route中的component来说，想要获得离其最近的router中的参数需要调用高阶函数(higher order component)，也就是在`export default XX`中对XX进行包裹的函数，其实相当于Aux，通过props.children传递参数，使用方式

```
import { withRouter } from 'react-router-dom';
...
export default withRouter(post);
```
这样的Link如何进行style将这个Link显示成为`NavLink`
```
<NavLink
  to="/"
  exact
  activeClassName="activeProps"
  activeStyle={
    {
      ...
  }
}
>
```
这样就实现了一个页面的默认导航按钮的位置确认

#### 动态传入component页面间的参数 Pass routing params

实现方式:在Route path 的 位置写上
                
```
<Route path="/:id" exact component={FullPost}/>
```
表示转换为动态id传参，这里的id参数来源是动态显示的，不是固定值，需要绑定不同的值在`<Link to={id}>`中的id跟随需求改变。

在跳转到的component界面显示的内容，之后可能会产生有歧义的现象，因为不同的界面可能会出现
```
<Route path="/new-post"  component={NewPost}/>
<Route path="/:id" exact component={FullPost}/>
```
这样的情况下，可能会认为new-post也是一个id也会产生传参问题，解决方式是在外层包裹上`<Switch></Switch`，表示只渲染第一个Route。所以对于Route来说 **位置很重要**

#### 使用nested Route 表示两个componentrender的时候摆在一个页面上

    这里的例子是在主页面上点击某按钮就能显示另一个component中的东西，在主页面要点击的地方加入Route写好对应的path同时注意主页面中不再使用那个Route而且为了防止出现exact不能动态加载的情况不使用exact。

***react小知识，对于两个不同数据类型int string如果只是想比较value可以使用!=而不是!==***

#### conditional redirect

使用`<Redirect from="/new-post" to="/posts/">`实现参数传递，表示页面跳转，比如输入结束之后调用axios的post同时跳转到之前的页面可以使用条件式的跳转机制，或者是直接`this.props.history.replace('/posts')`实现跳转。
同时`this.props.history.goBack()`表示返回原来位置。

#### 使用guards
在没有authentication情况下使用guard，没有什么组件，使用state和dirty的inline判断来guard

#### 使用lazily load route

在bundle文件中会默认加载所有页面上会渲染到的文件，为了不再一开始就全部加载，导致时间过长，可以使用lazy加载方式，也就是Async异步策略。
写入高阶组件，需要传入的是`importComponet`是一个函数
```
import React,{Component} from 'react';
const asyncComponent = (importComponet) =>{
    return class extends Component{
        state = {
            component:null
        }
        componentDidMount(){
            importComponet()
            .then(cmp
            =>{
                this.setState({
                  component:cmp.default  
                });
            });
        }
        render() {
            const C = this.state.component;
            return C ?<C {...this.props} /> :null;
        }
    }
}
export default asyncComponent;
```

使用的格式是在一开始加载的页面中，先导入上面定义的asyncComponent方法，之后再通过`import()`关键字导入需要异步加载的模块。asyncComponent中传入的是匿名函数，也就是前面定义的`importComponet`。
```
import asyncComponent from '../../../hoc/asyncComponent';
const AsyncNewPost = asyncComponent(()=> {
    return import('./NewPost/NewPost');
})
```
这里的`import()`中传入的是需要异步加载的模块的相对位置，也就是`import xx from '..//'`的这种位置。

同时因为加载的AsyncNewPost也是一个component，所以在最后的Route中也是

+ `<Route path="/new-post"  component={AsyncNewPost}/>` 

替换了原先的

+ `<Route path="/new-post" component={NewPost} />`

#### react16.6以上的lazy加载方式

使用suspend
具体实现方式，在主界面引入Suspense 
```
import React, {Component, Suspense} from 'react'
```

在定义class之前定义`React.lazy()`
```
const Posts = React.lazy(()=> import('Posts位置'))
```

在主界面render()位置
```
<Route render={()=>(
    <Suspense fallback={<div>Loading</div>}>
        <Posts/>
    </Suspense>)}/>
```

### 使用Next.js帮助自动配置router
以上的router其实再写的过程中会有冗余感，next.js是一个基于react的路径配置<a href="https://nextjs.frontendx.cn/docs" target="_blank">next.js中文文档</a>
<a href="https://nextjs.org/learn/basics/create-nextjs-app" target="_blank">官方文档</a>

1. `npm install `
2. 修改package.json文件中scripts
3. 写入基本代码
4. 运行`npm run dev`

#### automatic code spilting 自动跳转
```
import React from 'react';
import Link from 'next/link';
import Router from 'next/router';

const indexPage = () => (
    <div>
        <h1>the main page</h1>
        <p>Go to <Link href="/auth">Auth</Link></p>
        <button onClick={() => Router.push("/auth")}> got to auth</button>
    </div> 
)
export default indexPage;
```
点击界面中的`Link` 包裹或者`button`就会实现页面跳转，相比于router简便。

**在一个页面上加载的另外地方的component会自动在跳转时加载**

#### 使用css
next.js中可以使用style和之前使用的radius或者classes导入作为css传递，新的css使用方法是
```
const user = (props) => (
    <div>
        <h1>{props.name}</h1>
        <p>{props.age}</p>
        <style jsx>{`
            div {
                border:1px solid #eee;
                bosx-shadow:0 2px 3px #eee;
                padding:20px;
                text-align:center;
            }
        `}
        </style>
    </div>
);

```
#### 处理error 404 界面
需要定义文件名为`_error.js`的错误处理文件
```
const errorPage = () => (
    <div>
        <h1>something went wrong </h1>
        <p>Try <Link href="/">go back</Link></p>
    </div> 
)
export default errorPage;
```
#### next.js原理部分
以生命周期举例，next.js是在server端的加载，只有其child被点击之后才会开始加载


### react Animation 页面美化相关
在Burger中的modal的作用就是显示一种美化之后的模态对话框。

#### 页面动态美化，使页面能够显示一种modal加载出来是弹跳的状态。

利用`@keyframes`表示多少百分比的时间显示的透明度和下降深度，transform:表示开始位置是页面正上方，这里的相对Y是中心。这里的openModal功能是modal初始化载入，透明度由0变为1，再之后的closeModal是关闭modal。
```
@keyframes openModal{
    0% {
        opacity: 0;
        transform: translateY(-100%);
    }
    50% {
        opacity: 1;
        transform: translateY(90%);
    }
    100% {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes closeModal{
    0% {
        opacity: 1;
        transform: translateY(0);
    }
    50% {
        opacity: 0.8;
        transform: translateY(60%);
    }
    100% {
        opacity: 0;
        transform: translateY(-100%);
    }
}
```

以定义的`@keyframes`名字为属性，在css文件中导入，对应的没有动态弹跳形式的原始css，注意animation的参数，首先是`@keyframes`名字、加载到指定位置的秒数、什么形式加载、加载后是否回到初始位置（也就是`forwards`）如果没有`forwards`，会在弹跳之后变为0%时状态。
```
.ModalOpen{
    /* display: block;
    opacity: 1;
    transform: translateY(0); */
    animation: openModal 0.4s ease-out forwards;
}
.ModalClose{
    /* display: none; 
    opacity: 0;
    transform: translateY(-100%); */
    animation: closeModal 0.4s ease-out forwards;
}
```

#### css加载在react遇到的问题

因为加载顺序，在弹跳界面本来是要做同样的弹跳形式返回到原始界面，但是因为display执行顺序，所以不会显示这样界面，引入<a href="https://reactcommunity.org/react-transition-group/" target="_blank">**reactjs/react-transition-group**组件</a>

`npm install react-transition-group --save`导入

分为四个状态
There are 4 main states a Transition can be in:

'entering'

'entered'

'exiting'

'exited'

解决方法就是在exiting和exited之间切换`<Transition></Transition>`实现css，具体的实现方式可以看官方文档
<a href="https://reactcommunity.org/react-transition-group/transition" target="_blank">Transition</a>

这里注意在载入Transition时会因为react-redux版本冲突加载错误，需要将react设置为16.3以上版本。
















### Do not go gentle into that good night. 不要温驯的走进那个良夜

Do not go gentle into that good night,

Old age should burn and rave at close of day;

Rage, rage against the dying of the light.
![](/img/react-axios.jpg)