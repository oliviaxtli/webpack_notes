## 作用
多个独立的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署他们。这通常被称作微前端

## 底层概念
我们区分本地模块和远程模块。本地模块即为普通模块，是当前构建的一部分。远程模块不属于当前构建，并在运行时从所谓的容器加载。

加载远程模块被认为是异步操作，当使用远程模块时，这些异步操作将被放置在远程模块和入口之间的下一个chunk的加载操作中。如果没有chunk加载操作，就不能使用远程模块。

chunk的加载操作通常是通过调用`import()`实现的，也只是`require.ensure`或`require([...])`

容器是由容器入口创建的，该入口暴露了对特定模块的异步访问。暴露的访问分为两个步骤：
1. 加载模块（异步的）
2. 执行模块（同步的）

步骤1将在chunk加载期间完成，步骤2将在与其他（本地和远程）的模块交错执行期间完成。执行顺序不受模块从本地转换为远程或从远程转为本地的影响。

容器可以嵌套使用，容器可以使用来自其他容器的模块，容器之间也可以循环依赖

## 高级概念
每个构建都充当一个容器，也可将其他构建作为容器，这样，每个构建都能够通过从对应容器中加载模块来访问其他容器暴露出来的模块


共享模块是指既可重写的又可作为向嵌套容器提供重写的模块。他们通常指向每个构建中的相同模块，如相同的库

packageName选项允许通过设置包名来查找所需的版本，默认情况下，会自动推断模块请求，当想禁用自动推断时，请将requiredVersion设置为false

## 构建块
### ContainerPlugin (low level)
该插件使用指定的公开模块来创建一个额外的容器入口

### ContainerReferencePlugin (low level)
该插件将特定的引用添加到作为外部资源（externals）的容器中，并允许从这些容器中导入远程模块

### ModuleFederationPlugin (high level)
ModuleFederationPlugin组合了ContainerPlugin 和 ContainerReferencePlugin。
```ts
const { ModuleFederationPlugin } = require('webpack').container;
module.exports = {
    plugins:[
      new ModuleFederationPlugin({
        runtime:string | fasle,
      })
    ],
};
```
#### Options
##### runtime
```
const { ModuleFederationPlugin } = require('webpack').container;
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      runtime: 'my-runtime-name',
    }),
  ],
};
```
##### 指定依赖包的版本
- 数组依赖
```
const { ModuleFederationPlugin } = require('webpack').container;
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      // adds lodash as shared module
      // version is inferred from package.json
      // there is no version check for the required version
      // so it will always use the higher version found
      shared: ['lodash'],
    }),
  ],
};
```
- 对象依赖
```
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      shared: {
        // adds lodash as shared module
        // version is inferred from package.json
        // it will use the highest lodash version that is >= 4.17 and < 5
        lodash: '^4.17.0',
      },
    }),
  ],
};
```
- 对象依赖（指定参数）
```
const deps = require('./package.json').dependencies;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      shared: {
        // adds react as shared module
        react: {
          requiredVersion: deps.react,
          singleton: true,
        },
      },
    }),
  ],
};
```
> 其中具体配置可参考https://webpack.docschina.org/plugins/module-federation-plugin

## 用例
### 每个页面单独构建
单页应用的每个页面都是在单独的构建中从容器暴露出来的。主体应用程序也是独立构建，会将所有页面作为远程模块来引用。通过这种方式，可以单独部署每个页面，在更新路由或添加路由时部署主体应用程序。主体应用程序将常用库定义为共享模块，以避免在页面构建中出现重复

### 将组件库作为容器
许多应用程序共享一个通用的组件库，可以将其构建成暴露所有组件的容器。每个应用程序使用来自组件库容器的组件，可以单独部署对组件库的更改，而不需要重新部署所有应用程序

## 动态远程容器
该容器接口支持`get`和`init`方法，`init`是一个兼容async的方法，调用时，只含有一个参数：共享作用域对象。此对象在远程容器中用做共享作用域，并由host提供的模块填充，可以利用它在运行时动态地将远程容器连接到host容器
- init.js
```
function loadComponent(scope, module) {
  return async () => {
    // 初始化共享作用域（shared scope）用提供的已知此构建和所有远程的模块填充它
    await __webpack_init_sharing__('default');
    const container = window[scope]; // 或从其他地方获取容器
    // 初始化容器 它可能提供共享模块
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
}

loadComponent('abtests', 'test123');
```

## 基于Promis的动态Remote
一般的remote：
```
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        app1: 'app1@http://localhost:3001/remoteEntry.js',
      },
    }),
  ],
};
```
可以向remote传递一个promise，应该用任何符合上面描述的get/init接口的模块来调用这个promise
```
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        app1: `promise new Promise(resolve => {
      const urlParams = new URLSearchParams(window.location.search)
      const version = urlParams.get('app1VersionParam')
      // This part depends on how you plan on hosting and versioning your federated modules
      const remoteUrlWithVersion = 'http://localhost:3001/' + version + '/remoteEntry.js'
      const script = document.createElement('script')
      script.src = remoteUrlWithVersion
      script.onload = () => {
        // the injected script has loaded and is available on window
        // we can now resolve this Promise
        const proxy = {
          get: (request) => window.app1.get(request),
          init: (arg) => {
            try {
              return window.app1.init(arg)
            } catch(e) {
              console.log('remote container already initialized')
            }
          }
        }
        resolve(proxy)
      }
      // inject this script with the src set to the versioned remoteEntry.js
      document.head.appendChild(script);
    })
    `,
      },
      // ...
    }),
  ],
};
```
> 请注意当使用该 API 时，你 必须 resolve 一个包含 get/init API 的对象。


## 动态Pubilc Path
### 提供一个host api以设置publicPath
可以允许host在运行时通过公开远程模块的方法来设置远程模块的publicPath，特别适用于在host域的子路径上挂载独立部署的子应用程序

#### 案例
你在 https://my-host.com/app/* 上有一个 host 应用，并且在 https://foo-app.com 上有一个子应用。子应用程序也挂载在 host 域上, 因此， https://foo-app.com 可以通过 https://my-host.com/app/foo-app 访问，并且 https://my-host.com/app/foo-app/* 可以通过代理重定向到 https://foo-app.com/*。
- webpack.config.js (remote)
```
module.exports = {
  entry: {
    remote: './public-path',
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'remote', // 该名称必须与入口名称相匹配
      exposes: ['./public-path'],
      // ...
    }),
  ],
};
```
- public-path.js (remote)
```
export function set(value) {
  __webpack_public_path__ = value;
}
```
- src/index.js (host)
```
const publicPath = await import('remote/public-path');
publicPath.set('/your-public-path');

//boostrap app  e.g. import('./boostrap.js')
```


