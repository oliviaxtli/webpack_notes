## Configuration
Webpack的配置文件是JavaScript文件，文件内导出了一个webpack配置的对象。webpack遵循CommonJS模块规范，可以在配置中使用
- 通过`require(...)`引入其他文件
- 通过`require(...)`使用npm下载的工具函数
- 使用JavaScript控制流表达式，例如`?:`操作符
- 对value使用常量或变量赋值
- 编写并执行函数，生成部分配置

### 基本配置
webpack 会假定项目的入口起点为 src/index.js，然后会在 dist/main.js 输出结果，并且在生产环境开启压缩和优化。通常你的项目还需要继续扩展此能力，为此你可以在项目根目录下创建一个 webpack.config.js 文件，然后 webpack 会自动使用它。

webpack.config.js
```
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './foo.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'foo.bundle.js',
  },
};
```
#### 使用不同的配置文件
如果需要根据特定情况使用不同的配置文件，可以通过在命令行中使用`--config`标志修改

package.json
```
"scripts": {
  "build": "webpack --config prod.config.js"
}
```
#### 选项
[查看官网](https://webpack.docschina.org/configuration/#options****)

