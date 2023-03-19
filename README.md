---
title: react+taro+dva开发小程序
date: 2023-3-14 10:00:00
---

taro 是目前支持小程序最多的前端框架，并且支持 ReactNative，我们可以轻易的生成媲美原生的 APP 应用。所以公司的应用如果想全网推广，占用最多的流量入口的话，使用 Taro 是一种不错的方案。
目前 taro 支持的终端

1. 微信小程序、支付宝小程序、百度小程序
2. H5
3. React-native

## 1. Taro 开发环境的安装与项目搭建

### 1.1 必备开发软件

VsCode+微信小程序开发者工具

### 1.2 搭建项目

第一步，安装@tarojs/cli（脚手架工具）

```
npm install -g @tarojs/cli

或者 yarn global add @tarojs/cli
```

第二步，安装完成后，就可以用脚手架创建项目，命令如下：

```
taro init 项目名xxx
```

第三步，配置完项目即可运行

```
cd 项目名xxx

npm run dev:weapp
```

第四步，在小程序开发者工具中导入 dist 目录文件

![](https://pic.imgdb.cn/item/641080dbebf10e5d5390df6e.png)

## 2.目录结构说明

```
├── dist                   编译结果目录,每次进行预览都会根据我们预览的终端不同，编译成不同代码，比如使用yarn dev:h5那生成的就是web的代码，如果使用yarn dev:weapp那生成的就是小程序的代码。
├── config                 配置目录
|   ├── dev.js             开发时配置
|   ├── index.js           默认配置
|   └── prod.js            打包时配置
├── src                    源码目录
|   ├── pages              页面文件目录
|   |   ├── index          index 页面目录
|   |   |   ├── index.js   index 页面逻辑
|   |   |   └── index.css  index 页面样式
|   ├── app.css            项目总通用样式
|   └── app.js             项目入口文件
└── package.json
```

## 补充说明

### 3-6 步为基础知识总结，想直接集成 dva，可从第 7 步开始阅读

## 3.将类组件改为函数组件 + React Hooks 开发方式

以 index 修改为例：

```
import { useState } from "react";
import { View, Text } from "@tarojs/components";
import "./index.scss";

function Index() {
  const [userState, setUserState] = useState("test");

  return (
    <View>
      <Text>{userState}</Text>
    </View>
  );
}

export default Index;
```

此外，taro 提供自定义 hooks，具体参考文档：<a href="https://taro-docs.jd.com/docs/hooks">taro hooks</a>

## 4. 渲染机制

### 4.1 条件渲染

通过&&(或)运算符来达到 v-if 的一样效果。

```
import {useState} from 'react'
 function Index() {
    const [flag,setFlag]=useState(true)
    return (
     <div>
       {flag && <h1>hello</h1>}
     </div>
    )
 }
export default Index
```

### 4.2 列表渲染

通过 map 遍历出标签插入，给绑定的元素添加 key

```
import {useState} from 'react'
 function Index() {
    const [tempList,setFlag]=useState([{name:"foo",age:18},{name:"bar",age:20},{name:"jack",age:22}])
    return (
     <div>
       <ul>
         {
           tempList.map((item,index) => {
             return <li key={index}>{item.name}---{item.age}</li>
           })
         }
       </ul>
     </div>
    )
 }
export default Index
```

## 5. 父子组件的编写与传值

```
// 新建一个子组件
import { View, Text, Button } from "@tarojs/components";

interface PropsTYpe{
    value:string;
    onChange:(value:string)=>void
}
function Child({value,onChange}:PropsTYpe) {

    return (
      <View>
        <Text>{value}</Text>
        <Button className='btn-max-w' plain type='primary' onClick={()=>onChange("我是子组件的值")}>修改</Button>
      </View>
    );
  }

  export default Child;
```

```
// 父组件引入并向子组件传值
import { useState } from "react";
import { View, Text } from "@tarojs/components";
import "./index.scss";
import Child from "../../components/demo";

function Index() {
  const [userState] = useState("test");
  const [childState, setChildState] = useState("parantData");
  return (
    <View>
      <Text>{userState}</Text>
      <Child value={childState} onChange={setChildState}></Child>
    </View>
  );
}

export default Index;

```

![](https://pic.imgdb.cn/item/64108a1aebf10e5d53a2e1a3.png)

## 6.taro 的路由与跳转

### 6.1 配置路由

例如：

在/src/pages/文件夹下，建立一个/user 文件夹，在文件夹下面建立一个 index.tsx 文件，写入下面的代码:

```
import { useState } from "react";
import { View, Text } from "@tarojs/components";
import "./index.scss";

function User() {
  const [userState] = useState("用户信息页面");
  <Button onClick={()=>{Taro.navigateTo({url:'/pages/index/index'})}}>跳转首页</Button>
  return (
    <View>
      <Text>{userState}</Text>
    </View>
  );
}

export default User;

```

有了页面之后就可以到/src/app.config.tsx 下，配置路由

// 注意：路由在最前面运行后页面优先展示该页，其次修改配置后若不生效，则可以尝试重启项目

```
export default defineAppConfig({
  pages: [
    'pages/index/index',
    'pages/user/index',
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'WeChat',
    navigationBarTextStyle: 'black'
  }
})
```

### 6.2 页面间的跳转

Taro 提供了 6 个相关的导航 API，我们可以使用这些 API 进行跳转，需要注意的是这些有些是小程序专用的。

- navigateTo: 最基本的跳转方式，可以返回上级页面。三端都支持的，小程序、H5、React Native。
- redirectTo：不记录上集页面，直接跳转。三端都支持的，小程序、H5、React Native。
- switchTab： Tab 之间进行切换，这个要配合 Taro 的导航栏一起使用，三端都支持的，小程序、H5、React Native。
- navigateBack: 返回上一级页面，这个在小程序中常使用，三端都支持的，小程序、H5、React Native。
- relaunch：关闭所有额面，打开到应用内某个页面。三端都支持的，小程序、H5、React Native。
- getCurrentPages:获取当前页面信息所用，这个 H5 是不支持的。（注意）

### 6.3 页面传参

跳转：Taro.navigateTo+qs

获取 router 实例 ：getCurrentInstance

```
// user.tsx，通过qs库传递多个参数

import { useState } from "react";
import { View, Text, Button } from "@tarojs/components";
import "./index.scss";
import Taro from "@tarojs/taro";
import { stringify } from 'qs';

function User() {
  const [userState] = useState("用户信息页面");
  return (
    <View>
      <Text>{userState}</Text>
      <Button onClick={()=>{Taro.navigateTo({url:'/pages/index/index'})}}>跳转首页</Button>
      <Button onClick={()=>{Taro.navigateTo({url:`/pages/index/index?${stringify({id:'1',name:'zhangsan'})}`})}}>携带参数跳转</Button>
    </View>
  );
}

export default User;

```

```
// index.tsx，通过getCurrentInstance获取实例
import { useState, useEffect } from "react";
import { View, Text } from "@tarojs/components";
import "./index.scss";
import Child from "../../components/demo";
import { getCurrentInstance } from "@tarojs/runtime";

interface UrlStateType {
  id: string;
  name?: string;
}

function Index() {
  const [userState] = useState("test");
  const [childState, setChildState] = useState<string>("parantData");
  const [urlParamState, setUrlParamState] = useState<UrlStateType>();

  useEffect(() => {
    const { id, name } = getCurrentInstance()?.router?.params as any;
    setUrlParamState({ id, name });
  }, []);

  return (
    <View>
      <Text>{userState}</Text>
      <Child value={childState} onChange={setChildState}></Child>
      <View>{JSON.stringify(urlParamState)}</View>
    </View>
  );
}

export default Index;

```

## 7. 封装请求

基于 Taro.request 二次封装请求

在项目 src 目录下创建 utils 文件夹， 并在 utils 目录里创建 request.ts

```
import Taro from '@tarojs/taro';

// 请求连接前缀
export const baseUrl = 'https://itzhengjinhan.giuhub.com';

// 输出日志信息
export const noConsole = false;


const request_data = {
  platform: 'wap',
  rent_mode: 2,
};

export default (options = { method: 'GET', data: {} }) => {
  if (!noConsole) {
    console.log(
      `${new Date().toLocaleString()}【 M=${options.url} 】P=${JSON.stringify(
        options.data
      )}`
    );
  }
  return Taro.request({
    url: baseUrl + options.url,
    data: {
      ...request_data,
      ...options.data,
    },
    header: {
      'Content-Type': 'application/json',
    },
    method: options.method.toUpperCase(),
  }).then(res => {
    const { statusCode, data } = res;
    if (statusCode >= 200 && statusCode < 300) {
      if (!noConsole) {
        console.log(
          `${new Date().toLocaleString()}【 M=${options.url} 】【接口响应：】`,
          res.data
        );
      }
      if (data.status !== 'sucess') {
        Taro.showToast({
          title: `${res.data.error.message}~` || res.data.error.code,
          icon: 'none',
          mask: true,
        });
      }
      return data;
    } else {
      throw new Error(`网络请求错误，状态码${statusCode}`);
    }
  });
};
```

## 8. 集成 dva

dva 首先是一个基于 redux 和 redux-saga 的数据流方案，然后为了简化开发体验，dva 还额外内置了 react-router 和 fetch，所以也可以理解为一个轻量级的应用框架。

### 8.1 安装

安装 react-redux 、react-redux 等依赖包，大部分文档说安装 tarojs/redux，但是这个包在 taro3.x 已经消失了

```
npm i --save react-redux
npm install --save redux @tarojs/redux @tarojs/redux-h5 redux-thunk redux-logger
```

安装 dva-core 和 dva-loading

dva-core 封装了 redux 和 redux-sage 的一个插件;dva-loading 用来管理页面的 loading 状态

```
npm i --save dva-core dva-loading
```

### 8.2 创建 dva

#### 1.在 src 目录下创建 utils 目录，并在 utils 目录里创建 dva.ts 文件

```
import { create } from "dva-core";
import { createLogger } from "redux-logger";
import createLoading from "dva-loading";

let app;
let store;
let dispatch;
let registered;

function createApp(opt) {
  // redux 的日志
  opt.onAction = [createLogger()];
  app = create(opt);
  app.use(createLoading({}));

  if (!registered) {
    opt.models.forEach((model) => app.model(model));
  }
  registered = true;
  app.start();

  store = app._store;
  app.getStore = () => store;
  app.use({
    onError(err) {
      console.log(err);
    },
  });

  dispatch = store.dispatch;
  app.dispatch = dispatch;
  return app;
}

export default {
  createApp,
  getDispatch() {
    return app.dispatch;
  },
};

```

#### 2.将入口文件 app.ts 改为 app.tsx,使用 dva.ts 返回的方法创建一个 app 获取 store ，并将 store 挂载到 Provider 容器里面

```
import React, { Component } from "react";
import { Provider } from "react-redux";
import dva from "./utils/dva";
import models from "./models";
import "./app.scss";

const dvaApp = dva.createApp({
  initialState: {},
  models,
});
const store = dvaApp.getStore();
class App extends Component {
  componentDidMount() {}
  componentDidShow() {}
  componentDidHide() {}
  componentDidCatchError() {}
  // this.props.children 是将要会渲染的页面
  render() {
    return <Provider store={store}>{this.props.children}</Provider>;
  }
}
export default App;

```

#### 3.在 src 目录下创建 models 文件夹，在 models 文件夹下创建 index.ts

每新增一个页面，若要使用 dva 则在这里引用

```
import index from '../pages/index/model'

export default [index]
```

#### 4. 在 src/page/index 目录下创建 model.ts 文件

后期开发项目是每一个页面组件都需要含有一个 model.ts 文件用来做当前页面的异步数据请求和数据流管理, model.ts 文件内容如下:

```
import * as homeApi from "./service";

export default {
  namespace: "home",
  state: {
    data: []
  },
  effects: {
    *load(_, { call, put }) {
      const { status, data } = yield call(homeApi.homepage, {});
      if (status === "sucess") {
        yield put({
          type: "save",
          payload: {
            data: data
          }
        });
      }
    }
  },
  reducers: {
    save(state, { payload }) {
      return { ...state, ...payload };
    }
  }
};

```

#### 5.测试下 dva 是否生效

在 src/page/index 目录下创建 service.ts 文件

```
// 假设返回[1,2,3,4,5,6]；
export const homepage = data => {
  return {
    status: "sucess",
    data: [1, 2, 3, 4, 5, 6]
  };
};
// Request({
//   url: '/getHomepageData',
//   method: 'GET',
//   data,
// });
```

在 src/page/index 目录下修改 index.tsx 内容如下

通过@connect 连接，数据在 this.props 中返回

```
import { Component } from 'react'
import { View, Text } from '@tarojs/components'
import { AtButton } from 'taro-ui'

import "taro-ui/dist/style/components/button.scss" // 按需引入
import './index.scss'


import { connect } from 'react-redux'

@connect(({ home, loading }) => ({
  ...home,
  ...loading,
}))

export default class Index extends Component {

componentWillMount() { }

componentDidMount() {
  this.props.dispatch({
    type: 'home/load',
  });
}

componentWillUnmount() {
}

componentDidShow() { }

componentDidHide() { }

render() {
  return (
    <View className='index'>
      <Text>Hello world!</Text>
      <AtButton type='primary'>I need Taro UI</AtButton>
      dva数据：{JSON.stringify(this.props)}
    </View>
  )
  }
}

```

![](https://pic.imgdb.cn/item/6416c516a682492fcca6bbc8.png)

## 9. 引入 Taro-UI

注意：React-native 不支持 Taro-UI，视情况安装

### 9.1 安装

```
npm i --save taro-ui@3.0.0-alpha.3
```

### 9.2 引入组件样式的方式

全局引入样式(JS 中)，在入口文件引入所有样式：

```
import 'taro-ui/dist/style/index.scss'
```

按需引入，在页面样式或全局样式中 import 需要的组件样式

```
@import "~taro-ui/dist/style/components/button.scss";
```

### 9.3 使用见官方文档

<a href="https://taro-ui.jd.com/#/docs/quickstart">Taro UI 官网</a>
