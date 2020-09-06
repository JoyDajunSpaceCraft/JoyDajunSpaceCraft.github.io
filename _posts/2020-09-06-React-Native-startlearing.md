---
layout:     post
title:      " React Native 学习总结 "
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

编程软件 Visual Studio, 需要预先加载的是npm，可以自动帮助渲染，学习资料来源于Udemy上的React Native教程，老师是德国人*Maximilian Schwarzmüller*， 讲述很详实，带着学生一点一点搭建一个购物网站。

<a href="http://udemy.com/course/react-native-the-practical-guide" target="_blank">udemy</a>

### 实战项目简介

利用rn作为前端，redux实现component中相互传递参数，其中用到了useEffect, useState,useCallback等在"react-native"中需要传入的参数。这里用到的代码引入是：
```
import React, {...}  from 'react';
```
对应的几个需要预先加载的东西 database : firebase获得token等内容，实现存储商品和用户个人信息，firebase是google开发所以登录时需要能够**VPN**连接。





