## 认识Webpack
webpack是一个现代JavaScript应用程序的静态模块打包器。当webpack处理应用程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个bundle

### module、chunk和bundle的区别
module、chunk和bundle实质上是同一份逻辑代码在不同的三种转换场景下的状态名称。
- modlue：指的是各个源码文件，webpack看来一切皆模块
- chunk：是多个模块合并成，包括的类型有：
  - webpack配置中入口文件entry是chunk
  - 入口文件以及它的依赖文件通过动态加载import()/require.ensure()出来的模块是chunk
  - 通过SplitChunks抽取出的公有代码也是chunk
- bundle：最终的输出文件，也可以经过打包压缩和编译的源文件，可以直接在浏览器上运行

## Webpack的工作流程
1. webpack配置参数，合并从shell传入和webpack.config.js文件里配置的参数，生成出最后的配置结果
2. 注册所有配置的插件，好让插件监听webpack构建生命周期的事件节点，以做出对应的反应
3. 从配置的entry入口文件开始解析文件构建AST语法树，找出每个文件所依赖的文件，递归下去
4. 在解析文件递归的过程中根据文件类型和loader配置，找出合适的loader用来对文件进行转换
> 从entry开始读取文件，根据文件类型和配置的Loader对文件进行贬义词，将Loader处理后的文件通过acorn抽象成AST，然后便遍历AST，递归分析构建该模块的所有依赖
5. 递归完后得到每个文件的最终结果，根据entry配置生成代码块chunk
6. 输出所有chunk到文件系统

### AST树
抽象语法树，以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构
#### AST应用场景
- es6转es5，TypeScript、JSX转原生JavaScript
- 优化变更代码、代码压缩
- 代码风格，语法的检查，IDE中的错误提示，格式化，自动补全等

## 认识EventSource
webpack热加载中利用了EventSource来进行通信。EventSource是服务器推送的一个网络事件接口，一个EventSource实例会对HTTP服务开启一个持久化的连接，以text/event-stream格式发送事件，会一直保持开启直到被要求关闭。（单向通信，只能服务端推送给客户端，不能客户端推送给服务端）

### EventSource与websocket的区别
- webSocket基于TCP协议，EventSource基于http协议
- EventSource是单向通信，websocket是双向通信
- EventSource只能发送文本，websocket支持发送二进制数据
- 在实现上EventSource比websocket简单
- EventSource有自动重连接以及发送随机事件的能力
- websocket的资源占用过大，EventSource更轻量
- websocket可以跨域，EventSource基于http跨域需要服务端设置请求头

## Webpack热更新原理
1. webpack编译期，为需要热更新的entry注入热更新代码（EventSource通信）
2. 页面首次打开后，服务端与客户端通过EventSource建立通信渠道，把下一次的hash返回给前端
3. 修改页面代码后，webpack监听到文件修改后，开始编译，编译完成后，发送build信息（hash）给客户端
4. 客户端获取到hash，构造请求hot-update.json的ajax链接，获取成功后将hot-update.js插入到主文档中
5. hot-update.js插入成功后，执行hotAPI的createRecord和reload方法，获取到（Vue/React）组件的render方法，重新render组件，继而实现UI无刷新更新

