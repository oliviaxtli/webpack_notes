## loader
loader用于对模块的源代码进行转换。loader可以使你再import或"load"模块时预处理文件。loader可以将文件从不同的语言转换为JavaScript，或将内联图像转换为data URL.

然后指示 webpack 对每个 .css 使用 css-loader，以及对所有 .ts 文件使用 ts-loader：
```
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' },
    ],
  },
};
```
### 使用loader
- 配置方式（推荐）：在webpck.config.js文件中指定loader
- 内联方式：在每个import语句中显示指定loader

#### 配置方式
module.rules 允许你在 webpack 配置中指定多个 loader。loader 从右到左（或从下到上）地取值(evaluate)/执行(execute)。
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // [style-loader](/loaders/style-loader)
          { loader: 'style-loader' },
          // [css-loader](/loaders/css-loader)
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          // [sass-loader](/loaders/sass-loader)
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

#### 内联方式
使用 `!` 将资源中的 loader 分开。每个部分都会相对于当前目录解析。
```
import Styles from 'style-loader!css-loader?modules!./styles.css';
```
通过为内联 import 语句添加前缀，可以覆盖 配置 中的所有 loader, preLoader 和 postLoader
- 使用 `!` 前缀，将禁用所有已配置的 normal loader(普通 loader)
```
import Styles from '!style-loader!css-loader?modules!./styles.css';
```
- 使用 `!!` 前缀，将禁用所有已配置的 loader（preLoader, loader, postLoader）
```
import Styles from '!!style-loader!css-loader?modules!./styles.css';
```
- 用`-!`前缀，将禁用所有已配置的 preLoader 和 loader，但是不禁用 postLoaders
```
import Styles from '-!style-loader!css-loader?modules!./styles.css';
```

### loader特性
- loader支持链式调用，链中的每个loader会将转换应用在已处理过的资源上。一组链式的 loader 将按照相反的顺序执行。链中的第一个 loader 将其结果（也就是应用过转换后的资源）传递给下一个 loader，依此类推。最后，链中的最后一个 loader，返回 webpack 所期望的 JavaScript。

### 实战
```
module: {
            rules: [
                {
                    test: /bootstrap\.tsx$/,
                    use: [
                        {
                            loader: "bundle-loader",
                            options: {
                                lazy: true,
                            },
                        },
                    ],
                },
                {
                    test: /\.(ts|tsx)$/,
                    loader: "babel-loader",
                    // exclude: /node_modules/,
                    options: {
                        presets: [
                            [
                                "@babel/preset-env",
                                {
                                    useBuiltIns: "entry",
                                    targets: {
                                        ie: "11",
                                    },
                                    corejs: {
                                        version: "3.10.1",
                                        proposals: true,
                                    },
                                },
                            ],
                            "@babel/preset-react",
                            "@babel/preset-typescript",
                        ],
                        plugins: [
                            [
                                "@babel/plugin-transform-runtime",
                                {
                                    corejs: false,
                                },
                            ],
                            "@babel/plugin-proposal-class-properties",
                            // Ant Design 按需加载
                            [
                                "import",
                                {
                                    libraryName: "antd",
                                    libraryDirectory: "es",
                                    style: true,
                                },
                            ],
                            isEnvDevelopment &&
                            !isIE &&
                            isSingleModuleDev &&
                            require.resolve("react-refresh/babel"),
                        ].filter(Boolean),
                    },
                },
                {
                    test: /\.js$/,
                    loader: "babel-loader",
                    //fixed: 部分npm包的es6语法在IE11下报错，所以要做语法转换
                    include: /@umijs|fast-deep-equal/,
                    options: {
                        presets: ["@babel/preset-env"],
                    },
                },
                {
                    test: /\.css$/,
                    use: [{ loader: "style-loader" }, { loader: "css-loader" }],
                },
                {
                    test: /\.less$/i,
                    include: /node_modules/,
                    use: [
                        {
                            loader: "style-loader",
                            options: {
                                insert: insertStyle,
                            },
                        },
                        {
                            loader: "css-loader",
                        },
                        {
                            loader: "less-loader",
                            options: {
                                lessOptions: {
                                    javascriptEnabled: true,
                                },
                            },
                        },
                    ],
                },
                {
                    test: /\.less$/i,
                    include: /ui_lib/,
                    use: [
                        {
                            loader: "style-loader",
                            options: {
                                insert: insertStyle,
                            },
                        },
                        {
                            loader: "css-loader",
                        },
                        {
                            loader: "less-loader",
                            options: {
                                lessOptions: {
                                    javascriptEnabled: true,
                                    modifyVars: customTheme,
                                },
                            },
                        },
                    ],
                },
                {
                    test: /\.module\.less$/i,
                    include: /src/,
                    use: [
                        {
                            loader: "style-loader",
                        },
                        {
                            loader: "css-loader",
                            options: {
                                modules: {
                                    localIdentName:
                                        "[name]__[local]--[hash:base64:5]",
                                },
                            },
                        },
                        {
                            loader: "less-loader",
                            options: {
                                lessOptions: {
                                    javascriptEnabled: true,
                                    modifyVars: customTheme,
                                },
                            },
                        },
                    ],
                },
                {
                    test: /\.(png|jpg|gif)$/i,
                    type: "asset",
                    parser: {
                        dataUrlCondition: {
                            maxSize: 4 /*kb */ * 1024,
                        },
                    },
                    generator: {
                        filename: "images/[name]__[hash:8][ext][query]",
                    },
                },
                {
                    test: /\.svg$/i,
                    oneOf: [
                        // 仅用于地图svg的加载
                        {
                            test: /\.map\.svg$/i,
                            use: [
                                {
                                    loader: path.resolve(
                                        __dirname,
                                        "./map-svg-post-loader/src/index.js"
                                    ),
                                },
                            ],
                        },
                        {
                            test: /\.svg$/i,
                            use: ["@svgr/webpack", "url-loader"],
                        },
                    ],
                },
                {
                    test: /\.(ttf|otf)$/i,
                    type: "asset/resource",
                },
            ],
        },
```
