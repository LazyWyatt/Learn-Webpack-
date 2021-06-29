# Webpack
這裡使用Webpack3


## Loader
打包非js檔案的時候，webpack原本沒辦法處理，需要載入其他的Loader，用到哪些功能再去下載。

---
### 讀取Css檔案並且掛載到DOM上面
首先拿css舉例，載入css到網頁上呈現需要兩個。一個是css-loader，一個是style-loader。
- css-loader
    - 主要是將css文件加載到bundle.js裡面
- style-loader
    - 加載之後必須將樣式掛載到DOM上面，使用style-loader來達成效果

---
### Scss/Sass 轉換成 css
- sass-loader
    - 可以將sass/scss檔案轉換成css檔案，如果要使用sass-loader，需要連sass和node-sass一起安裝

---
### 圖片的加載
- url loader
- file loader
```javascript=
//webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: 'dist/',
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|jpeg)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: 'img/[name].[hash:8].[ext]',
            },
          },
        ],
      },
    ],
  },
 }
```

---
### ES6轉ES5
```bash=
npm install babel-loader@7 babel-core babel-preset-es2015 --save-dev
```
- babel-core babel 主要程式碼
- babel-loader@7 webpack使用babel-loader
- bebel-preset-es2015 es6的
- bebel-preset-env 需透過.babelrc配置


1.runtime-only -> 代碼中不能有任何template
2.runtime-compiler -> 代碼中，可以有template,因為有compiler可以用於編譯template

---
### Webpack配置Vue環境
```bash=
npm install vue --save
npm install vue-loader vue-template-compiler
```
#### Vue 中 el 與 template 的關係
template 會替換掉 el(原本我們寫的el="#app")的地方，所以我們可以引入一個名字為<App/>的組件
#### 抽取template
當我們發現抽取出來的template也是一個Object的時候，根據模組化開發的方式，可以新建一個js檔使用commonjs導出和引入，但這樣還是相當雜亂，因此Vue分別把他們裝進<template></template>和<script></script>和<style></style>的標籤裡面，並使用附檔名為vue的文件
#### 解析副檔名.vue文件
需要使用vue-loader來載入vue文件，並且使用vue-template-compiler才能將vue中有使用mustache的地方顯示出來
- vue-loader 
- vue-template-compiler

---


## Webpack Plugin
Loader像是轉化器，Plugin像是擴展器
#### BannerPlugin
- 以Webpack原本就有的版權宣告插件為例
```
//webpack.config.js
module.exports = {
    entry: ...,
    output: {},
    module: {},
    resolve: {},
    plugins: [new webpack.BannerPlugin('測試版權')],
}
```
#### HtmlWebpackPlugin
- 自動生成一個index.html文件
- 將打包過的js文件，自動通過script標籤插入body
```bash=
npm install html-webpack-plugin --save-dev
```
```javascript=
//webpack.config.js
module.exports = {
    entry: ...,
    output: {},
    module: {},
    resolve: {},
    plugins: [
      new webpack.BannerPlugin('測試版權'),
      new HtmlWebpackPlugin({
        template: 'index.html',
      })
    ],
}
```

#### UglifyJsPlugin
- 壓縮js文件
```bash=
npm install uglifyjs-webpack-plugin@1.1.1 --save-dev
```

```javascript=
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
    entry: ...,
    output: {...},
    module: {...},
    resolve: {...},
    plugins: [
        new UglifyjsWebpackPlugin(),
    ],
}
```

#### webpack-dev-server
搭建本地伺服器，使用node來搭建
```bash=
npm install webpack-dev-server@2.9.3 --save-dev
```

```javascript=
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
    entry: ...,
    output: {...},
    module: {...},
    resolve: {...},
    plugins: [...],
    devServer: {
      contentBase: './dist',
      inline: true,
      port: 8999, //port號
      // historyApiFallback: false // SPA頁面中，依賴HTML5的History模式
    }
}
```
```json=
//package.json
{
  "scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --open"
  },
}
```

## Webpack配置分離
#### webpack-merge
把開發環境、跟最後生產環境配置分開來
```cmd
npm install webpack-merge@4.1.5 --save-dev
```
```javascript=
//base.config.js
//node裡的模組
const path = require('path')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'bundle.js',
    // publicPath: 'dist/',
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        //css loader 只負責將css加載
        //style loader 將樣式添加到DOM上面
        //webpack使用多個loader時，是從右向左
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.s[ac]ss$/i,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
      {
        test: /\.(png|jpg|gif|jpeg)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: 'img/[name].[hash:8].[ext]',
            },
          },
        ],
      },
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015'],
          },
        },
      },
      {
        test: /\.vue$/,
        use: ['vue-loader'],
      },
    ],
  },
  resolve: {
    // alias別名
    extensions: ['.js', '.css', '.vue'],
    alias: {
      vue$: 'vue/dist/vue.esm.js',
    },
  },
  plugins: [
    new webpack.BannerPlugin('測試版權'),
    new HtmlWebpackPlugin({
      template: 'index.html',
    }),
  ],
}

```

```javascript=
//dev.config.js
const webpackMerge = require('webpack-merge')
const baseConfig = require('./base.config')

module.exports = webpackMerge(baseConfig, {
  devServer: {
    contentBase: './dist',
    inline: true,
  },
})
```

```javascript=
//prod.config.js
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
const webpackMerge = require('webpack-merge')
const baseConfig = require('./base.config')

module.exports = webpackMerge(baseConfig, {
  plugins: [new UglifyjsWebpackPlugin()],
})

```