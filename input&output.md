## 入口起点(entry points)
入口起点指示webpack应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack会找出有哪些模块和库是入口起点依赖的。
### 单个入口（简写）语法
`entry:string|[string]`

##### webpack.config.js
```
module.exports = {
  entry: './path/to/my/entry/file.js',
};
或
module.exports = {
  entry: ['./src/file_1.js', './src/file_2.js'],
};
```

### 对象语法
`entry: { <entryChunkName> string | [string] } | {}`
##### webpack.config.js
```
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js',
  },
};
```
#### 描述入口的对象
- dependOn：当前入口所依赖的入口。它们必须在该入口被加载前被加载
- filename：指定要输出的文件名称
- import：启动时需加载的模块
- library：指定library选项，为当前entry构建一个library
- runtime：运行时chunk的名字，如果设置了，就会创建一个新的运行时chunk
- publicPath：当该入口的输出文件在浏览器中被引用时，为它们指定一个公共URL地址

> runtime 和 dependOn 不应在同一个入口上同时使用
> 
> runtime 不能指向已存在的入口名称

```
module.exports = {
  entry: {
    a2: 'dependingfile.js',
    b2: {
      dependOn: 'a2',
      import: './src/app.js',
    },
  },
};
```


## 输出(output)
可以通过配置output选项，告知webpack如何向硬盘写入编译文件。即使可以存在多个entry起点，也只能指定一个output配置

### 用法
output属性的最低要求是，将它的值设置成一个对象，然后为将输出文件的文件名配置为一个`output.filename`
```
// 此配置将一个单独的 bundle.js 文件输出到 dist 目录中
module.exports = {
  output: {
    filename: 'bundle.js',
  },
};
```

### 多个入口起点
如果配置中创建出多于一个“chunk”（例如，使用多个入口起点或使用像CommonsChunkPlugin这样的插件），则应该使用占位符来确保每个文件具有唯一的名称
```
module.exports = {
  output: {
    filename: '[name].js',
    path: __dirname + '/dist',
  },
};

// 写入到硬盘：./dist/app.js, ./dist/search.js
```

### 高级进阶
```
// 使用CDN和hash
module.exports = {
  //...
  output: {
    path: '/home/proj/cdn/assets/[fullhash]',
    publicPath: 'https://cdn.example.com/assets/[fullhash]/',
  },
};
```
如果在编译时不知道最终输出文件的publicPath是什么地址，可以将其留空，并在运行时通过入口起点文件中的__webpack_public_path动态设置
```
__webpack_public_path__ = myRuntimePublicPath;

// 应用程序入口的其余部分
```

## 实战例子
```
const publicPath = isEnvProduction
        ? env.publicPath
        : `//localhost:${env.port}/`;
        
module.exports = {  
    entry: path.resolve(paths.appSrc, "index"),
    output: {
            // fixed: 用auto，在ie下会出现请求js的地址不正确的bug，必须要指定publicPath，不能用auto
            // publicPath 不指定protocol会导致webpack-dev-serve的HMR功能失效
            publicPath: isEnvProduction || isIE ? publicPath : "auto",
            filename: isEnvProduction
                ? "static/js/[name].[contenthash:8].js"
                : "static/js/[name].js",
            chunkFilename: isEnvProduction
                ? "static/js/[name].[contenthash:8].chunk.js"
                : "static/js/[name].chunk.js",
        },
}
```
