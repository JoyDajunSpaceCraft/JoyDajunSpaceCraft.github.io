---
layout: post
title: ' React Native 设备加载 '
subtitle: 'react native device feature (camera, location, map, SQLite ...)'
date: 2020--09-06
author: 'Joy'
header-img: 'img/react-native-device.jpg'
tags:
  - react native
  - self-learning
  - permission
---

## React Native 实现手机端的相机加载

需要了解到 expo 作为一个基础平台提供了很多工具，比如加载数据库，获取手机的权限，打开相机并拍照等功能，所以利用 expo 提供的 API 实现一个能打开相机，保存拍摄的图片，获得当前位置，在地图上展现出当前位置的 App。

## 基础界面

- 分为 4 个界面
  - MapScreen _地图界面_
  - NewPlaceScreen _创建新的图片和位置界面_
  - PlaceDetailScreen _位置详细信息界面_
  - PlacesListScreen _所有位置的渲染界面_


## React Native 实现设备拍摄照片

用户在`ImagePicker.launchCameraAsync({...})`中获得的图像的URI即存储路径，同样其导入是`expo install expo-image-picker`可以在官方文档的ImagePicker中找到用法,同样不光需要拍摄，在拍摄前需要获取device的权限，需要用到Permissions API
```
import * as ImagePicker from 'expo-image-picker'; 
import * as Permissions from 'expo-permissions';

const verfiypermissions = async () => {
        // 这个方法在用户确认后会自动保存在设备上， 用户不需要再次使用
        const result = await Permissions.askAsync(Permissions.CAMERA, Permissions.CAMERA_ROLL)// return 一个Promise
        if (result.status !== 'granted') {
            // 如果没有授权 
            Alert.alert('Insufficient Permissions!',
                'You need to grant camera permissions to use this app.',
                [{ text: 'okey' }]
            );
            return false;
        }
        return true;
    }
```

## React Native 实现保存照片到本地device

## React Native 实现 SQLite 数据库连接

首先对数据库的连接是在创建 shopApp 中做的，这里是将 fetch 的网页连接变成实际的数据库，当然调用的是用原生 Sql 语言编写，expo 的 SQLite API 提供了事务性。

1.  首先定义一个 helpers 包和 App.js 同级，这个是为了更好的调用数据库，其中写入 db.js 文件
    在终端中导入包 `expo install expo-sqlite`可以在官方文档的SQLite中找到用法
2.  下面代码定义了创建一个 places 数据库，首先是新建一个 Promise,其中的 SQLite.openDatabase(...) 在官方文档中 Open a database, creating it if it doesn't exist, and return a Database object,利用 Promise 包裹住这个事务是保证有 error 时可以在 await 中 catch 住。

    ```
    import * as SQLite from 'expo-sqlite';

    const db = SQLite.openDatabase('place.db');

    export const init = () => {
        //create basic table
        const promise = new Promise((resolve, reject) => {
            db.transaction((tx) => {
                // transaction保证事务性
                tx.executeSql("CREATE TABLE IF NOT EXISTS places (id INTEGER PRIMARY KEY NOT NULL, title TEXT NOT NULL, imageUri TEXT NOT NULL, address TEXT NOT NULL, lat REAL NOT NULL, lng REAL NOT NULL)",
                    [],
                    () => {
                        // SUCCESS
                        resolve();
                    },
                    (_, err) => {//第一个参数是 repeition of query ,第二个才是需要的error信息
                        // error
                        reject(err);
                    }
                );
            });
        });
        return promise;
    };
    ```

3.  在 store/places-actions.js 中来获得数据库传入的数据
    这里的 FileSystem.documentDirectory 又是 expo 提供的 API 需要先导入 `expo install expo-file-system`可以在官方文档的FileSystem中找到用法作用是：provides access to a file system stored locally on the device. Within the Expo client, each app has a separate file system and has no access to the file system of other Expo apps.这里是为了将拍下来的照片存入device。
    
    其中addPlace是需要在NewPlaceScreen中载入的图片，其中传入的参数，一个是用户输入的title,一个是存入缓存中的imageUri,一路follow其实传递路线是

    ```
    export const addPlace = (title, image) => {
        return async dispatch => {
            // 需要dispatch的方法
            const fileName = image.split('/').pop();
            const newPath = FileSystem.documentDirectory + fileName;
            console.log(newPath)

            try {
                await FileSystem.moveAsync({// 存储位置 从哪里来 到哪里去
                    from: image,
                    to: newPath
                });
                const dbResult = await insertPlace(
                    title,
                    newPath,
                    'Dummey Address',
                    15.6,
                    12.3
                );
                console.log(dbResult);
                dispatch({ type: ADD_PLACE, placeData:
                { id: dbResult.insertId,
                title: title,
                image: newPath } });

            } catch (err) {
                console.log(err);
                throw (err);
            }
        }
    }
    ```

## React Native 实现手机端获得地址

## React Native 百度地图实现静态地图渲染

<a href="http://lbsyun.baidu.com/index.php?title=static" target="_blank">百度地图开发者获得静态地图界面</a>
原始的教程中是 Google Map 获得 API 但是需要绑定 credit card 以及连接 VPN，这里利用百度地图也可以实现类似功能，就是界面有点难找：）


