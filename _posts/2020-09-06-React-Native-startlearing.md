---
layout:     post
title:      " React Native 建立一个购物App "
subtitle:   "react native"
date:       2020--09-06
author:     "Joy"
header-img: "img/react-native-got7.jpg"
tags:
- react native
- self-learning
---
### React Native 概述

是一种适用于手机端的前端编程语言， 适用于IOS和Android，但是在Android的使用中应该是不能很好兼容。

***React Native以下称为RN***

编程软件 Visual Studio, 需要预先加载的是npm，可以自动帮助渲染，学习资料来源于Udemy上的React Native教程，老师是德国人*Maximilian Schwarzmüller*， 讲述很详实，带着学生一点一点搭建一个购物网站。<a href="http://udemy.com/course/react-native-the-practical-guide" target="_blank">udemy</a>

### 简介

首先要了解RN作为一个前端编程语言能够实现的功能。在开始学习前要至少掌握JS最基本的语法，<a href="https://es6.ruanyifeng.com/?search=AsyncStorage&x=0&y=0#" target="_blank">ES6(阮一峰)</a>，这个链接在遇到未知的语法时可以随时查看。
React 知识也需要知道，能够理解props和state作用即可。


### 实现源码 
<a href="https://github.com/JoyDajunSpaceCraft/react-native-for-a-shop" target="_blank">JoyDajunSpaceCraft/react-native-for-a-shop</a>

### 初始化
1. expo init *你的RN文件名*
2. 选择template 选择 第一个blank即可
3. 进入以刚刚RN文件命名的 dir 执行 npm start

作为基本条件，需要安装npm，npm作为web端打包软件可以帮助加载RN中需要包。

以下就是执行成功

```
- cd rn-device-feature-app
- npm start # you can open iOS, Android, or web from here, or run them directly with the commands below.
- npm run android
- npm run ios
- npm run web
```
网页端自动出现http://localhost:19002/ 界面可以点击里面的 Run on XXX 在电脑端打开模拟界面。如果是手机需要模拟，需要下载expo，而且确保手机和电脑处于一个网络环境下。

如果想要调试在电脑模拟界面(mac -> ios) *cmd + D* 点击 *Debug remote JS*，在浏览器中能够实现debug，如果是要界面精美，有data flow的debug，可下载 react native debugger实现精细化debug。当然对我比较适用的是笨办法，打开Termial，console.log。


#### 前期准备

 如果有多页面需求 可以在 <a href="https://reactnavigation.org/docs/getting-started" target="_blank">react-navigation/doc/getstart</a>中找到加载navigation的包。
 我把需要导入的包都写入App.js开头注释中，在其他页面的开头注释中也会有相应要导入的依赖包。

 1. 导入 redux 请按照以下形式在终端中导入
 ```
 npm install --save redux react-redux redux-thunk
 ```
 2. 利用RN作为前端，redux实现component中相互传递参数，其中用到了useEffect, useState,useCallback等在"react-native"中需要传入的参数。这里用到的代码引入是：
 ```
 import React, {...}  from 'react';
 import {...} from 'react-native';
 ```
 3. 将要用到的数据库 database：firebase 用来获得token等和存储数据。firebase是google开发所以登录时需要能够**VPN**连接。

#### 整体结构
 在已经创建好的基础上显示的结构，我们能够用到的是App.js作为主文件，其他结构需要自己创建。
 需要自己增加的结构如下：
 + components           *在较大的页面中需要加载的小部分，一个页面中的一小块*
 + constants            *放置常用的常量，比如整体的颜色设定*
 + model                *放入定义的数据的格式，例如一件商品的名称、价格、描述和图片链接*
 + navigation           *页面导航*
 + screens              *主要页面*
 + store                *存放reduer和action* 

     **这里的模块都是本项目所使用的，并不代表一定需要这样创建**

### 手动实践
- 界面的加载
- 类似于css的style渲染，在RN中的页面布局不是用CSS，而是用和其类似的StyleSheet 导入方式是

```
import { StyleSheet } from 'react-native';
```

是输入RN原生组件一部分，同样可以导入的有 { View, Text, TextInput, Button,...}遇到详细的可以细说，首先要掌握的是RN中的StyleSheet使用方式。

```
import { View, Button, StyleSheet, Image } from 'react-native';

<View style={styles.imagePicker}>
            <View style={styles.imagePreview}>
                <Text>
                    No image pick yet
                </Text>
                <Image style={styles.image}/>
            </View>
            <Button
                title="Take Image"
                color={Colors.primary}
                onPress={takeImageHandler} />
        </View>

const styles = StyleSheet.create({
    imagePicker:{

    },
    imagePreview:{
        width:'100%',
        height:200,
        marginBottom:10,
        justifyContent:'center',
        alignItems:'center',
        borderColor:'#ccc',
        borderWidth:1

    },
    image:{
        width:'100%',
        height:'100%'
    }
});

```
其中的布局方式可以看这篇<a href="https://www.jianshu.com/p/c390042d6140" target="_blank">react-native中flexDirection、justifyContent、alignItems的简单使用</a>
剩下的内容只需要看style逻辑就好。

- 需要了解ES6中异步的相关知识，主要会用到的是async 和 await，因为涉及到Promiss中的内容但是我还没有好好看：）但是可以先了解一下异步操作，<a href="https://www.jianshu.com/p/4e91c4be2843" target="_blank">ES6之async和await</a>

- 理清RN中action和reducer的关系，在这个项目中每个页面功能分别对应一个action和reducer，action中存放的是界面中可以调用执行缓存的方法，返回的type标定了从页面上传来的数据是作用于什么场合；之后action会将数据return给reducer，reducer是真正意义上对缓存操作，相对于数据库更为灵活，可以根据从action中传入的type不同对数据进行不同的处理。如果想要拿到缓存中的数据，可以利用useSelector实现，同样需要搭配一些设置。这里是简单了解。




 







