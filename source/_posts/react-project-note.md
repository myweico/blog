---
title: react 项目的基本搭建
date: 2020-02-29 23:52:52
tags:
  - react
  - webpack
---
# react 项目配置
自己搭了一套 react 的的后台，下面记录了我一步步搭建的过程，希望对大家有所帮助。

文章主要描述了下面几个部分
- 项目的结构
- 项目的搭建过程
  - webpack 的多环境配置、loader配置、webserver配置等
  - babel 的基本配置
  - eslint 的基本配置
<!-- more -->
## 项目结构
```
|
|- config               // webpack 相关配置
|   |- webpack.common.js    // 通用的配置
|   |- webpack.dev.js       // 发展的配置 
|   |- webpack.prod.js      // 生产的配置
|
|- dist                 // 放置打包好的文件
|- public               // 放置html模板以及图标等
|- src
|   |- apis             // apis 接口文件
|   |   |- brand.js
|   |   |- channel.js
|   |   |- ...
|   |
|   |- assets           // 放置通用的资源
|   |   |- images
|   |   |- icons
|   |
|   |- components       // 通用的组件
|   |   |- PageLoading
|   |
|   |- config
|   |   |- menu.js      // 菜单配置
|   |   |- routes.js    // 路由配置
|   |
|   |- layouts          // 布局组件
|   |   |- BasicLayout  // 基本布局
|   |   |   |- components   // 组成布局的组件
|   |   |   |- index.jsx    // 布局的主要结构文件
|   |   |   |- index.module.scss    // 布局对应的scss
|   |   |-...
|   |
|   |- pages            // 页面文件
|   |   |- OrderManage
|   |   |- ...
|   |
|   |- stores
|   |   |- actions
|   |   |   |- index.js // 触发 actions 的常量名列表
|   |   |   |- menu.js  // 对应的 actions
|   |   |
|   |   |- reducers
|   |   |   |- index.js // 合并各个模块的 reducer，输入根的 reducer
|   |   |   |- menu.js  // 对应模块的 reducer
|   |   |
|   |   |- index.js     // 输出根据 rootReducer 生成 store
|   |
|   |- utils // 各种工具
|   |   |- date.js      // 日期格式化相关的工具
|   |   |- request.js   // 封装好的 axios 组件
|   |
|   |- index.js         // 入口文件
|   |- router.jsx       // 生成路由组件
|
|- test                 // 测试文件
|   |- unit             // 单元测试
|
|- .babelrc             // babel 相关配置
|- .editourconfig       // 编辑器的相关配置，编辑器会根据此格式化文件
|- .eslintrc.js         // eslint 的配置文件
|- .gitignore           // git 忽略追踪的配置文件
|- jsconfig.josn        // js的配置文件，可以让别名 alias 得到识别
|- package.json         // 配置文件
|- postcss.config.js    // postcss 配置文件
|- README.md            // Readme.md 文件
```

## 项目搭建
初始化项目
```
npm init
```

安装webpack
```shell
npm install --save-dev webpack
```
webpack 4+ 需要安装 CLI
```shell
npm install --save-dev webpack-cli
```
创建一个 config 文件夹，专门用于存放 webpack 的配置文件
```
|- config
    |- webpack.common.js    存放开发和生产环境共用的配置
    |- webpack.dev.js       开发环境的配置
    |- webpack.prod.js      生产环境的配置
```

这里使用到`webpack-merge`合并通用配置
```
npm i webpack-merge -D
```
```js
// webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

const rootPath = path.resolve(__dirname, '../');

module.exports = {
  entry: {
    index: path.resolve(rootPath, 'src/index')    // 入口文件
  },
  output: {
    filename: 'static/js/[name].[hash:8].bundle.js',       // 将文件放置到 static/js目录下，名字格式为[name].[hash:8].bundle.js
    chunkFilename: 'static/js/[name].[hash:8].bundle.js',  // 将 bunndle 文件也放置在 static/js 目录下
    path: path.resolve(rootPath, 'dist'),         // 打包文件存放的根目录
    publicPath: '/'                               // 引入文件的根目录
  },
  module: { ...
  },
  plugins: [ ... ],
  resolve: {
    alias: {
      '@': path.resolve(rootPath, 'src')                      // 设置 @ 为 src 的别名
    },
    extensions: ['.js', '.jsx', '.json']                      // 没有扩展名的时候，尝试的扩展名
  }
};

// webpack.dev.js
const merge = require("webpack-merge");
const path = require("path");
const webpack = require("webpack");

const common = require("./webpack.common");

const rootPath = path.resolve(__dirname, "../");

module.exports = merge(common, {
  mode: "development", // 设置为开发模式，使用 webpack 内置专为开发配置的设置
  devtool: "cheap-source-map",
  module: {
    rules: [
      ...
    ]
  },

  devServer: {
    ...
  },
  plugins: [ ... ]
});

// prod.webpack.js
const merge = require("webpack-merge");
const webpack = require("webpack");
const path = require("path");
const common = require("./webpack.common");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin"); // 压缩 css, 好像有没有也可以~

const rootPath = path.resolve(__dirname, "../");

module.exports = merge(common, {
  mode: "production", // 启动生产模式，使用webpack默认设置的生产模式的设定
  devtool: "none",
  module: {
    rules: [ ... ]
  },
  plugins: [ ... ]
});

```
在`package.json`中调用对应的配置文件
```json
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --config ./config/webpack.dev.js",
    "dev": "webpack --config ./config/webpack.prod.js"
  },Ï
}
```

#### HtmlWebpackPlugin
> HtmlWebpackPlugin插件自动生成html文件，引入对应的包文件

安装
```
npm install --save-dev html-webpack-plugin
```
在`common.js`中添加以下配置
```
const HtmlWebpackPlugin = require('html-webpack-plugin');

const rootPath = path.resolve(__dirname, '../');
module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
          // 自动生成引入 bundle 文件的 html
          template: path.resolve(rootPath, 'public/index.html'),  // 使用 public 下的 index 文件作为 html 模板
          favicon: path.resolve(rootPath, 'public/favicon.ico')   // 引入 图标
    })
    ]
}
```

#### webpack-dev-server
安装
```
npm install --save-dev webpack-dev-server
```
在`dev.js`配置
```
module.exports = {
  devServer: {
  contentBase: path.resolve(rootPath, "dist"), // 服务器的根目录
    compress: true,     // 启用 gzip 压缩
    overlay: true,      // 错误在浮层显示
    open: true,         // 服务器开启后，自动在浏览器打开
    hot: true,          // 启用热加载
    host: "0.0.0.0",    // 服务启动的ip，可以让在局域网内的用户都可以访问
    port: 9000,         // 开启的端口
    proxy: [
      // 代理配置
      {
        context: "/api/geo/",
        target: "http://0.0.0.0:1131",  // 转发的地址
        changeOrigin: true,             // 是否改变转发的域名
        secure: false                   // 关闭安全的设定
      },
      {
        context: [
          "/api/v2",
          "/api/orderproduct",
          "/api/member",
          "/api/user",
          "/api/channel",
          "/api/store2",
          "/api/white",
          "/api/taobao",
          "/api/tool",
          "/api/brand",
          "/api/order"
        ],                              
        target: "http://127.0.0.1:9502"     // 批量转发，上面的接口都转发到本地的9502接口
      },
      {
        context: "/api",
        target: "http://127.0.0.1:11299",   // api前缀的转发到11299端口
        changeOrigin: true,                 // 是否改变转发的域名
        secure: false                       // 关闭安全的设定
      }
    ],
    historyApiFallback: true                // 404的时候重定向回 index.html
  }
}
```
某些路由使用的history模式的路由，需要404时候，重定向回 index.html，则可以开启下面的选项
```
historyApiFallback: true // 404的时候重定向回 index.html
```

#### clean-webpack-plugin
安装
```
npm install --save-dev clean-webpack-plugin
```
在common中配置文件
```
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports {
    plugins: [
        new CleanWebpackPlugin() // 清除 dist 文件
    ]
}
```

#### 图片的loader
安装url-loader, url-loader可以将小于一定大小的图片，转换成DataURL格式然后引入，对于大于等于指定大小的图片，将会默认使用 file-loader 将图片引入，所以需要安装file-loader
```shell
npm i url-loader file-loader -D
```
loader配置
```
...
    module: {
        rules: [
              {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'url-loader',
                options: {
                  limit: 10240,                           // 小于 10kb 的图片转化为 url 数据引入，否则调用 file-loader 引入文件
                  name: 'static/img/[path][name].[ext]'   // 图片放置在 static/img 目录下, 生产模式可以使用内容哈希的文件名，来缓存
                }
              }
        ]
    }

```

#### 字体文件的loader
字体文件loader与图片文件的loader类似,同样需要url-loader以及file-loader
```
npm i url-loader file-loader -D
```
option配置
```
    ...
      {
        test: /\.(svg|eot|ttf|woff|woff2)$/i,
        loader: 'url-loader',
        options: {
          limit: 4096,                            // 与图片loader类似
          name: 'static/fonts/[path][name].[ext]' // 字体文件放置在 static/fonts 文件下
        }
      },
    ...
```

#### css以及scss样式loader的配置
先安装css的相关loader
```
npm i style-loader css-loader -D
```
安装scss的相关loader
```
npm i node-sass sass-loader --save-dev
```
为了让自动加上css前缀，需要安装postcss
```
npm install postcss-loader autoprefixer --save-dev
```
在根目录创建一个 postcss.config.js 配置文件,
```js
module.exports = {
  loader: "postcss-loader",
  plugins: {
    "postcss-preset-env": {
    }
  }
};
```
安装`postcss-preset-env`，默认包含了autoprefixer
```
npm i -D postcss-preset-env
```
对于module后缀的启动css模块化
```
  {
    // 对于module。scss或者 module.css后缀的文件，开启 css 模块化，并设定指定生成class的格式
    test: /\.module\.s?css/,
    use: [
      'style-loader',
      {
        loader: 'css-loader',
        options: {
          importLoaders: 2,
          modules: {
            mode: 'local',
            localIdentName: '[path][name]__[local]--[hash:base64:5]' 
          }
        }
      },
      'postcss-loader',
      'sass-loader'
    ]
  },
```
其余的样式文件关闭css模块化，否则会造成引入的antd样式失效
```
{
    // 除了带有module结尾的scss文件或者css文件，不使用css模块化，否则将会导致antd的样式失效
    test: /\.s?css$/,
    exclude: /\.module\.s?css$/,
    use: [
      'style-loader',
      {
        loader: 'css-loader',
        options: {
          importLoaders: 2
        }
      },
      'postcss-loader',
      'sass-loader'
    ]
}
```
对于生产环境，若想将css提取出来，需要安装`mini-css-extract-plugin`
```
npm i mini-css-extract-plugin --save-dev
```
更改`webpack.prod.js`,将style-loader换成 css module 即可
```
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module: {
    rules: [
    {
        // module.scss 或者 module.css 结尾的文件启动css module，并提取到单独的文件夹
        test: /\.module\.s?css/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader // 提取到单独的文件
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: {
                mode: 'local',
                localIdentName: '[hash:base64:8]'
              }
            }
          },
          'postcss-loader',
          'sass-loader'
        ]
      },
      {
        // 除了module的scss或css文件，都不启动 css module 并且提取到单独的文件
        test: /\.s?css$/,
        exclude: /\.module\.s?css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2
            }
          },
          'postcss-loader',
          'sass-loader'
        ]
      }
    ]
}

plugins: [
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[hash:8].css',         // 将css文件都提取到 static/css 目录下
      chunkFilename: 'static/css/[id].[hash:8].css'
    })
]
```

#### babel配置
react中大多都是使用es6语法编写，为了让es6语法以及react的jsx语法能够正确在浏览器上运行，需要安装babel，
- @babel-core
- @babel/preset-env 将es6语法转换成es5语法
- @babel/preset-react 将jsx语法转换为Javascript
- @babel/polyfill: ES6 内置方法和函数转化垫片
- @babel/plugin-transform-runtime: 避免 polyfill 污染全局变量，减小打包体积
```
npm i @babel/core babel-loader @babel/preset-env @babel/preset-react --save-dev
```
在根目录创建`.babelrc`文件
```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```
若想转换es6新的语法，则需要安装`@babel/polyfill`
```
npm i @babel/polyfill @babel/runtime
```
若想重复利用babel注入的工具代码，可以安装以下插件
```
@babel/plugin-transform-runtime
```
若想要直接在 class 里面声明静态类变量以及直接声明箭头函数的实例方法
```
class Dog {
    // 声明静态变量
    static type = 'animal';
    
    // 箭头函数的实例方法
    bark = () => {
        console.log('Wang Wang ~')
    }
}
```
需要安装[ @babel/plugin-proposal-class-properties ](https://www.npmjs.com/package/@babel/plugin-proposal-class-properties)插件
```
npm install --save-dev @babel/plugin-proposal-class-properties
```
babel 整体配置
```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": [
    "react-hot-loader/babel",
    "@babel/plugin-proposal-class-properties",
    ["import", {
      "libraryName": "antd",
      "libraryDirectory": "es",
      "style": "css"
    }]
  ]
}

```

#### eslint配置
```javascript .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es6: true
  },
  extends: ['eslint:recommended', 'plugin:react/recommended'],
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly'
  },
  parser: 'babel-eslint',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 2018,
    sourceType: 'module'
  },
  plugins: ['react'],
  rules: {
    'react/jsx-filename-extension': ['warn', { extensions: ['.js', '.jsx'] }],
    'react/prop-types': ['error', { skipUndeclared: true }],
    'no-unused-vars': ['warn']
  },
  settings: {
    'import/resolver': {
      alias: {
        map: [['@', './src']],
        extensions: ['.js', '.jsx', '.json']
      }
    }
  }
};

```
#### 整体webpack的配置

项目 webpack 文件如下：
```js webpack.common.js
// webpack.commmon.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

const rootPath = path.resolve(__dirname, '../');

module.exports = {
  entry: {
    index: path.resolve(rootPath, 'src/index')    // 入口文件
  },
  output: {
    filename: 'static/js/[name].[hash:8].bundle.js',       // 将文件放置到 static/js目录下，名字格式为[name].[hash:8].bundle.js
    chunkFilename: 'static/js/[name].[hash:8].bundle.js',  // 将 bunndle 文件也放置在 static/js 目录下
    path: path.resolve(rootPath, 'dist'),         // 打包文件存放的根目录
    publicPath: '/'                               // 引入文件的根目录
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,                          // js, jsx 文件使用的 loader
        use: ['babel-loader', 'eslint-loader'],
        exclude: /node_modules/                   // 不转化 node_modules 下的文件
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      // 自动生成引入 bundle 文件的 html
      template: path.resolve(rootPath, 'public/index.html'),  // 使用 public 下的 index 文件作为 html 模板
      favicon: path.resolve(rootPath, 'public/favicon.ico')   // 引入 图标
    }),
    new CleanWebpackPlugin() // 清除 dist 文件夹
  ],
  resolve: {
    alias: {
      '@': path.resolve(rootPath, 'src')                      // 设置 @ 为 src 的别名
    },
    extensions: ['.js', '.jsx', '.json']                      // 没有扩展名的时候，尝试的扩展名
  }
};

```
{% codeblock lang:javascript webpack.dev.js%}
const merge = require("webpack-merge");
const path = require("path");
const webpack = require("webpack");

const common = require("./webpack.common");

const rootPath = path.resolve(__dirname, "../");

module.exports = merge(common, {
  mode: "development", // 设置为开发模式，使用 webpack 内置专为开发配置的设置
  devtool: "cheap-source-map",
  module: {
    rules: [
      {
        // 对于module。scss或者 module.css后缀的文件，开启 css 模块化，并设定指定生成class的格式
        test: /\.module\.s?css/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              importLoaders: 2,
              modules: {
                mode: "local",
                localIdentName: "[local]--[hash:base64:5]"
              }
            }
          },
          "postcss-loader",
          "sass-loader"
        ]
      },
      {
        // 除了带有module结尾的scss文件或者css文件，不使用css模块化，否则将会导致antd的样式失效
        test: /\.s?css$/,
        exclude: /\.module\.s?css$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              importLoaders: 2
            }
          },
          "postcss-loader",
          "sass-loader"
        ]
      },
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: "url-loader",
        options: {
          limit: 4096, // 小于 10kb 的图片转化为 url 数据引入，否则调用 file-loader 引入文件
          name: "static/img/[path][name].[ext]" // 图片放置在 static/img 目录下
        }
      },
      {
        test: /\.(svg|eot|ttf|woff|woff2)$/i,
        loader: "url-loader",
        options: {
          limit: 4096, // 与图片loader类似
          name: "static/fonts/[path][name].[ext]" // 字体文件放置在 static/fonts 文件下
        }
      }
    ]
  },

  devServer: {
    contentBase: path.resolve(rootPath, "dist"), // 服务器的根目录
    compress: true,     // 启用 gzip 压缩
    overlay: true,      // 错误在浮层显示
    open: true,         // 服务器开启后，自动在浏览器打开
    hot: true,          // 启用热加载
    host: "0.0.0.0",    // 服务启动的ip，可以让在局域网内的用户都可以访问
    port: 9000,         // 开启的端口
    proxy: [
      // 代理配置
      {
        context: "/api/geo/",
        target: "http://0.0.0.0:1131",  // 转发的地址
        changeOrigin: true,             // 是否改变转发的域名
        secure: false                   // 关闭安全的设定
      },
      {
        context: [
          "/api/v2",
          "/api/orderproduct",
          "/api/member",
          "/api/user",
          "/api/channel",
          "/api/store2",
          "/api/white",
          "/api/taobao",
          "/api/tool",
          "/api/brand",
          "/api/order"
        ],                              
        target: "http://127.0.0.1:9502"     // 批量转发，上面的接口都转发到本地的9502接口
      },
      {
        context: "/api",
        target: "http://127.0.0.1:11299",   // api前缀的转发到11299端口
        changeOrigin: true,                 // 是否改变转发的域名
        secure: false                       // 关闭安全的设定
      }
    ],
    historyApiFallback: true                // 404的时候重定向回 index.html
  },
  plugins: [
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": JSON.stringify("development")
    })
  ]
});

{% endcodeblock %}

{% codeblock lang:javascript webpack.prod.js %}
const merge = require("webpack-merge");
const webpack = require("webpack");
const path = require("path");
const common = require("./webpack.common");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCssAssetsPlugin = require("optimize-css-assets-webpack-plugin"); // 压缩 css, 好像有没有也可以~

const rootPath = path.resolve(__dirname, "../");

module.exports = merge(common, {
  mode: "production", // 启动生产模式，使用webpack默认设置的生产模式的设定
  devtool: "none",
  module: {
    rules: [
      {
        // module.scss 或者 module.css 结尾的文件启动css module，并提取到单独的文件夹
        test: /\.module\.s?css/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader // 提取到单独的文件
          },
          {
            loader: "css-loader",
            options: {
              importLoaders: 2,
              modules: {
                mode: "local",
                localIdentName: "[hash:base64:8]"
              }
            }
          },
          "postcss-loader",
          "sass-loader"
        ]
      },
      {
        // 除了module的scss或css文件，都不启动 css module 并且提取到单独的文件
        test: /\.s?css$/,
        exclude: /\.module\.s?css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          {
            loader: "css-loader",
            options: {
              importLoaders: 2
            }
          },
          "postcss-loader",
          "sass-loader"
        ]
      },
      {
        // 图片的loader，小于10Kb就使用base64编码,否则则调用file-loader引入文件，文件名使用由内容哈希组成的文件
        test: /\.(png|jpe?g|gif)$/i,
        loader: "url-loader",
        options: {
          limit: 4096,
          name: "static/img/[hash:16].[ext]"
        }
      },
      {
        // 字体文件loader，小于4kb的字体使用base64格式引入，否则使用file-loader引入，字体文件名为内容哈希组成的文件
        test: /\.(svg|eot|ttf|woff|woff2)$/i,
        loader: "url-loader",
        options: {
          limit: 4096,
          name: "static/fonts/[hash:16].[ext]"
        }
      }
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": JSON.stringify("production") // 将文件中的process.env.NODE_ENV 设置为 production，以优化某些库中的代码
    }),
    // 提取 css 到单独的文件
    new MiniCssExtractPlugin({
      filename: "static/css/[name].[hash:8].css", // 将css文件都提取到 static/css 目录下
      chunkFilename: "static/css/[id].[hash:8].css"
    }),
    // 压缩 css
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /\.css$/g,
      cssProcessor: require("cssnano"), //用于优化\最小化 CSS 的 CSS处理器，默认为 cssnano
      cssProcessorOptions: { safe: true, discardComments: { removeAll: true } }, //传递给 cssProcessor 的选项，默认为{}
      canPrint: true //布尔值，指示插件是否可以将消息打印到控制台，默认为 true
    })
  ]
});
{% endcodeblock %}
在`webpack.common.js`配置输入和输出
{% codeblock lang:javascript webpack.common.js %}
const rootPath = path.resolve(_dirname, '../');

module.exports = {
  entry: {
    index: path.resolve(rootPath, 'src/index')    // 入口文件
  },
  output: {
    filename: 'static/js/[name].bundle.js',       // 将文件放置到 static/js目录下，名字格式为[name].bundle.js
    chunkFilename: 'static/js/[name].bundle.js',  // 将 bunndle 文件也放置在 static/js 目录下
    path: path.resolve(rootPath, 'dist'),         // 打包文件存放的根目录
    publicPath: '/'                               // 引入文件的根目录
  }
}
{% endcodeblock %}
