# study-react-code
Let's study react source code together!
文档说明
https://github.com/nannongrousong/blog/issues/1
```javascript
cd study-react-code
yarn run eject
yarn start
```
之后可以在对应react代码目录进行打印，调试操作


原文
最近开始阅读react源码，最重要的方法就是断点调试了。如果简单一点我们可以用发布好的react.production.js进行调试。但这样做带来一些问题：

全局变量混乱
代码没有模块化，结构不清晰
参数变量难以知晓意义，而有时候可以通过import的js文件初步判断参数类型和意义
如果我们直接能在本地调试react未发布版源码，那会对调试工作带来巨大的便捷性。搭建本地调试环境有点繁琐，不想手动搭建可以跳过下面步骤，直接去用我已经构建好的。react本地源码调试环境

使用create-react-app搭建react环境，并暴露webpack配置

create-react-app study-react-code
cd study-react-code
yarn run eject
yarn start
竟然提示没有模块@babel/plugin-transform-react-jsx，继续

yarn add -D @babel/plugin-transform-react-jsx
到git上拉下react源码，使用v16.8.6。将react/packages中内容拷贝到项目src/下

修改/config/webpack.config.js

resolve: {
    alias: {
        'react-native': 'react-native-web',
        'react': path.resolve(__dirname, '../src/react/packages/react'),
        'react-dom': path.resolve(__dirname, '../src/react/packages/react-dom'),
        'shared': path.resolve(__dirname, '../src/react/packages/shared'),
        'react-reconciler': path.resolve(__dirname, '../src/react/packages/react-reconciler'),
    }    
}
webpack和eslint中为全局变量赋值（__DEV__等）

修改/config/env.js

const stringified = {
    ...,
    "__DEV__": true,
    "__PROFILE__": true,
    "__UMD__": true
};
根目录创建.eslintrc.json文件

{
  "extends": "react-app",
  "globals": {
      "__DEV__": true,
      "__PROFILE__": true,
      "__UMD__": true
  }
}
忽略flow下type

yarn add @babel/plugin-transform-flow-strip-types -D
同时在/config/webpack.config.js中babel-loader的plugins中添加该插件

发现events模块并没有在alias中写，是因为项目中已存在同名events模块，会引起冲突，那只有手改react源码了。我们将events模块重命名为react-events中，并修改react源码中引用events的地方

首先在/config/webpack.config.js alias中再添加一行

'react-events', path.resolve(__dirname, '../src/react/packages/events')
很多修改点，哪些文件编译报错就改
直接将import XX from 'events/...'改为import XX from 'react-events/...'

修改文件/src/react/packages/react-reconciler/src/ReactFiberHostConfig.js。注释中说明，这块还需要根据环境去导出HostConfig

//  invariant(false, 'This module must be shimmed by a specific renderer.');
export * from './forks/ReactFiberHostConfig.dom';
保持import first，根据编译信息修改

修改文件/src/react/packages/shared/ReactSharedInternals.js。react此时未export内容，直接从ReactSharedInternals拿值

//  import React from 'react';
import ReactSharedInternals from '../react/src/ReactSharedInternals';

//  const ReactSharedInternals = React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED;
至此，基本的调试环境已经搭建完毕，开始学习咯~~
