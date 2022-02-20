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
 
