主要是通过文件体积大小入手，主要的措施有分包、减少http请求次数等。

常见的优化手段：
- JS代码压缩
- CSS代码压缩
- Html文件代码压缩
- 文件大小压缩
- 图片压缩
- Tree Shaking
- 代码分离
- 内联chunk

## JS代码压缩
terser是一个JavaScript的解释、绞肉机、压缩机的工具集，可以帮助我们压缩、丑化我们的代码，让bundle更小。在production模式下，webpack默认就是使用TerserPlugin来处理我们的代码。如果要自定义配置，配置方法如下：
```
const TerserPlugin = require('terser-webpack-plugin')
module.export = {
    ...
    optimizatin:{
        minmize:true,
        minmizer:[
            new TerserPlugin({
                paraller:true  // 电脑cpu核数-1
            })
        ]
    }
}
```
TerserPlugin常用的属性：
- extractComments：默认值为true，表示会将注释抽取到一个单独的文件中，开发阶段可以设置为false，不保留注释
- paraller：使用多进程并发运行提高构建的速度，默认值是true，并发运行的默认数量：os.cpus().length - 1
- terserOptions：设置我们的terser相关的配置
- compress：设置压缩相关的选项，可以直接设置为true
- mangle：设置丑化相关的选项，可以直接设置为true
- toplevel：底层变量是否进行转换
- keep_classnames：保留类的名称
- keep_fnames：保留函数的名称

## CSS代码压缩
CSS压缩通常用于去除无用的空格等，不过因为很难去修改选择器、属性的名称、值等，所以我们可以使用另外一个插件：css-minimizer-webpack-plugin
```
const CssMiniminzerPlugin = require('css-minimizer-webpack-plugin')
module.exports = {
    // ...
    optimization: {
        minimize:true,
        minimizer:[
            new CssMinimizerPlugin({
                paraller:true
            })
        ]
    }
}
```

## html文件代码压缩
使用HtmlWebpackPlugin插件来生成HTML模板的时候，可以通过配置属性minify进行html优化
```
module.export = {
    ...
    plugin:[
        new HtmlwebpackPlugin({
            ...
            minify:{
                minifyCSS:false,// 是否压缩css
                collapseWhitespce:false,// 是否折叠空格
                removeComments:true, //是否移除注释
            }
        })
    ]
}
```
 
 ## 文件大小压缩
 
对文件大小进行压缩，可以有效减少http传输过程中宽带的损耗，文件压缩需要用到compression-webpack-plugin插件
```
new CompressionPlugin({
    test:/\.(css|js)$/, // 哪些文件需要压缩
    threshold:500, // 设置文件多大开始压缩
    minRtio:0.7, //至少压缩的比例
    algorithm:"gzip", // 采用的压缩算法
})
```

## 图片压缩
如果我们对bundle包进行分析，会发现图片等多媒体文件的大小是远远要比js、css文件要大的，所以图片压缩在打包方面也是很重要的。
```
module:{
    rules:[
        {
        test: /\.(png|jpg|gif)$/,
        use:[
          {
          loader:'filer-loader',
          options:{
            name:'[name]_[hash].[ext]',
            outputPath:'images/',
          }
       },
       {
       loader:'image-webpack-loader',
       options:{
       // 压缩jpeg的配置
       mozjpeg:{
        progressive:true,
        quality:65
      },
      // 使用imagemin**-optipng压缩png，enbale：false为关闭
      optipng:{
        enable:false,
      },
      // 使用imagemin-pngquant压缩png
      pngquant:{
        quality:'65-90',
        speed:4
      }
      // 压缩gif的配置
      gifsicle:{
        interlaced:false,
      },
      //开启webp，会把jpg和png图片压缩为webp格式
      webp:{
        quality:75
      }
    }
  }
}
```

## Tree Shaking
Tree Shaking是一个术语，在计算机中表示消除死代码，依赖于ES Module的静态语法分析。在webpack实现Tree Shaking有两种不同的方案：
- usedExports：通过标记某些函数是否被使用，之后通过Terser来进行优化的
- sideEffects：跳过整个模块/文件，直接查看该文件是否有副作用

usedExports的配置方法很简单，只需要将usedExports设为true即可
```
module.exports = {
    ...
    optimization:{
        usedExports
    }
}
```
而sideEffects则用于告知webpackcompiler在编译时哪些模块有副作用，配置方法是在package.json中设置sideEffects属性。如果sideEffects设置为false，就是告知webpack可以安全删除未用到的exports，如果有些文件需要保留，可以设置为数组的形式
```
"sideEffects":["./src/util/foramt.js","*.css" // 所有css文件]
```

## 代码分离
默认情况下，所有的JavaScript代码（业务代码、第三方依赖、暂时没有用到的模块）在首页全部都加载，就会影响首页的加载速度。如果可以分出更小的bundle，以及控制资源加载优先级，从而优化加载性能。

代码分离可以通过splitChunksPlugin来实现，该插件webpack已经默认安装和集成，只要配置即可
```
module.exports = {
    ...
    optimization:{
        splitChunks:{
            chunks:"all"
        }
    }
}
```
splitChunks有如下几个属性：
- chunks：对同步代码还是异步代码进行处理
- minSize：拆分包的大小，至少为minSize。如果包的大小不超过minSize，这个包不会拆分
- maxSize：将大于maxSize的包，拆分为不小于minSize的包
- minChunks：被引入的次数，默认是1

## 内联chunk
可以通过InlineChunkHtmlPlugin插件将一些chunk的模块内联到html，如runtime的代码（对模块进行解析/加载/模块信息相关的代码），代码量不大但是必须加载的，比如
```
const InlineChunkHtmlPlugin = require("react-dev-utils/InlineChunkHtmlPlugin')
const HtmlWebpackPlugin= require('html-webpack-plugin')
module.exports = {
    ...
    plugin:[
        new InlineChunkHtmlPlugin(HtmlWebpackPlugin,[/runtime.+\.js/]}
    ]
}
```
