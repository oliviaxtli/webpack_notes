## 主要部分
项目中使用的每个文件都是一个模块
- ./index.js
```
import app from './app.js';
```
- ./app.js
```
export default 'the app';
```
通过互相引用，这些模块会形成一个图数据结构。

在打包过程中，模块会被合并成chunk。chunk合并成chunk组，并形成一个通过模块互相连接的图。
- ./webpack.config.js
```
module.exports = {
  entry: './index.js',
};
```
这会创建出一个名为main的chunk组（main是入口起点的默认名称）。此chunk组包含`./index.js`模块。随着parser处理`./index.js`内部的import时，新模块会被添加到此chunk中

- ./webpack.config.js
```
module.exports = {
  entry: {
    home: './home.js',
    about: './about.js',
  },
};
```
这会创建出两个名为home和about的chunk组，每个chunk组都有一个包含一个模块的chunk：./home.js对应home，./about.js对应about

## chunk
chunk有两种形式：
- initial（初始化）是入口起点的main chunk。此chunk包含为入口起点指定的所有模块及其依赖项
- non-initial是可以延迟加载的块，可能会出现在使用动态导入或者SplitChunksPlugin时

每个chunk都有对应的assets（输出文件，即打包结果）

- webpack.config.js
```
module.exports = {
  entry: './src/index.jsx',
};
```
- ./src/index.jsx
```
import React from 'react';
import ReactDOM from 'react-dom';

import('./app.jsx').then((App) => {
  ReactDOM.render(<App />, root);
});
```
这会创建出一个名为main的initial chunk，包含
- ./src/index.jsx
- react
- react-dom
- 以及除./app.jsx外的所有依赖
然后会为./app.jsx创建non-initial chunk，这是因为./app.jsx是动态导入的

输出件：
- /dist/main.js：一个initial chunk
- /dist/394.js：一个non-initial chunk

默认情况下，这些non-initial chunk没有名称，因此会使用唯一ID来替代名称。可以在使用动态导入时用magic comment来显示指定chunk名称
```
import(
  /* webpackChunkName: "app" */
  './app.jsx'
).then((App) => {
  ReactDOM.render(<App />, root);
});
```
此时的输出件：
- /dist/main.js：一个initial chunk
- /dist/app.js：一个non-initial chunk

## output（输出）
输出文件的名称会受配置中的两个字段影响：
- output.filename：用于initial chunk文件
- output.chunkFilename：用于non-initial chunk文件
- 在某些情况下，使用initial和non-initial的chunk时，可以使用output.filename

常用的占位符：
- `[id]`：chunk id（如`[id].js`->`485.js`）
- `[name]`：chunk name（如`[name].js`->`app.js`）。如果chunk没有名称，会使用其id作为名称
- `[contenthash]`：输出文件内容的md4-hash（如`[contenthash].js`=>`4ea6ff1de66c537eb9b2.js`）

### 实践
```
output: {
            // fixed: 用auto，在ie下会出现请求js的地址不正确的bug，必须要指定publicPath，不能用auto
            publicPath: publicPath,
            filename: isEnvProduction
                ? "static/js/[name].[contenthash:8].js"
                : "static/js/[name].js",
            chunkFilename: isEnvProduction
                ? "static/js/[name].[contenthash:8].chunk.js"
                : "static/js/[name].chunk.js",
        },
```
