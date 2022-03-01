## target
- webpack.config.js
```
module.exports = {
  target: 'node',
};
```
target设置为node，webpack将在类Node.js环境编译代码。（使用Node.js的require加载chunk，而不加载任何内置模块，如fs或path）。每个 target 都包含各种 deployment（部署）/environment（环境）特定的附加项

## 多target
虽然webpack不支持项target属性传入多个字符串，但是可以通过设置两个独立配置来构建对library进行同构
- webpack.config.js
```
const path = require('path');
const serverConfig = {
  target: 'node',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.node.js',
  },
  //…
};

const clientConfig = {
  target: 'web', // <=== 默认为 'web'，可省略
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.js',
  },
  //…
};

module.exports = [serverConfig, clientConfig];
```
上述示例中，将会在 dist 文件夹下创建 lib.js 和 lib.node.js 文件。
