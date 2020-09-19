---
layout: post
title: ' React 制作一个burger '
subtitle: 'react practice'
date: 2020--09-11
author: 'Joy'
header-img: 'img/react-burger.jpg'
tags:
  - react
  - self-learning
  - class-based
---


### Planning a React App怎样构建一个 app 
 1. component Tree / structure 
 2. application state
 3. components vs containers
 structure:
 APP 
    - toolbar 
        - drower toggle
        - logo 
        - navigation items


    - sideDrawer
        - logo 
        - navigation items

    - backDrop
    - props.children
        - burger build control ...order button
        - burger 
        - modal
 State 
    ingredient （meat * 2, cheese *2）
    purchase :true /false
    price 

在开始项目之前可以先将 css设定好
Optional:

If you still want to eject and manually adjust the Webpack config (as we do it in the new videos - which you don't need to do if you follow the approach described in the link above), you should take the below comments into account in case your webpack config (after ejecting) doesn't look the same as it does in my videos:

After ejecting, we edit a Webpack config file that's made available by ejecting. This file might look slightly different for you.

In the video, I'll look for an entry that starts like this (in the webpack.config.js file):

{
  test: /\.css$/,
  ...
}
and I then edit this entry.

This entry now looks slightly different. You'll have to find the following part in your webpack.config.js file:

{
  test: cssRegex,
  exclude: cssModuleRegex,
  ...
}
and then edit that entry.

Finally, it should look like this:

{
  test: cssRegex,
  exclude: cssModuleRegex,
  use: getStyleLoaders({
      importLoaders: 1,
      modules: true,
      localIdentName: '[name]__[local]__[hash:base64:5]'
  }),
}
You can ignore me editing the webpack.config.prod.js file - with the latest version of create-react-app, ejecting only gives you ONE webpack config file (which you edit as described above).
