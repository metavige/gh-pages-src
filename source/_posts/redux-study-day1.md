title: "Redux Study Day1"
date: 2015-10-14 20:25:24
tags: 
- reactjs
- Javascript 
---

準備開始學習使用 [Redux](https://github.com/rackt/redux)

<!--more-->

## 開始準備
* ECMAScript 6: [https://www.gitbook.com/book/wohugb/ecmascript-6/details]()  
* Redux Github: [https://github.com/rackt/redux]()
* Redux 的中文 gitbook: [http://chentsulin.github.io/redux/index.html]()

其中我覺得 ECMAScript 6 一定要先懂，因為，基本上現在你找的到的 Redux 範例，都會是 ES6 的語法了  
所以，如果你不先了解 ES6 的語法，一定會很痛苦.....  

當然，你要用 Redux 前要先了解 [Flux](https://facebook.github.io/flux) 是什麼會比較好   
以下這張圖大概就是 Flux 最著名的一張架構圖了 .....  
![](https://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)

## 參考範例

我個人覺得 [Redux-Hacker-News](https://github.com/tngan/redux-hacker-news) 的確是寫的不錯的～  

## 開始實作

本來我要用 yo 來做開始，後來發現產生出來的東西，大部分我都不懂，乾脆參考 Hacker-News 的範例重新來一次～  
之後遇到要使用到的時候再裝，印象比較深刻，比較知道自己在用什麼  

### 套件安裝

當然，先裝基本的 React/Redux  

```bash
npm install --save redux
npm install --save react
```

接下來是安裝開發環境要用的，比如說 webpack, babel 等

```bash
npm install --save-dev babel-core
npm install --save-dev babel-loader
npm install --save-dev babel-plugin-react-transform
npm install --save-dev webpack-dev-middleware
npm install --save-dev webpack-hot-middleware
```

安裝完之後，我的 package.json 大概是長下面這樣子的   

```
{
  // 其他我就不貼了～
  "dependencies": {
    "react": "^0.14.0",
    "redux": "^3.0.2"
  },
  "devDependencies": {
    "babel-core": "^5.8.25",
    "babel-loader": "^5.3.2",
    "babel-plugin-react-transform": "^1.1.1",
    "redux-devtools": "^2.1.5",
    "webpack": "^1.12.2",
    "webpack-dev-middleware": "^1.2.0",
    "webpack-hot-middleware": "^2.4.1"
  }
}
```


### 準備 webpack.config.js

不得不說，我覺得 redux-hacker-news 的 webpack.config.js 蠻不錯的，所以就直接先複製過來使用了！  

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
  devtool: 'cheap-module-eval-source-map',
  entry: [
    'webpack-hot-middleware/client',
    './index'
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/static/'
  },
  plugins: [
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  ],
  module: {
    loaders: [{
      test: /\.js$/,
      loaders: ['babel'],
      exclude: /node_modules/,
      include: __dirname
    },{
      test: /\.css$/,
      loader: 'style!css'
    },{
      test: /\.gif$/,
      loader: 'url?limit=10000&minetype=image/gif'
    }]
  }
};
```

### 準備一個測試的 server

一樣，hacker-news 的 server.js，很簡單也很實用～    
但是因為會用到 express ，補一下 npm install   

```bash
npm install --save-dev express
```

server.js 程式碼   

```javascript
var webpack = require('webpack');
var webpackDevMiddleware = require('webpack-dev-middleware');
var webpackHotMiddleware = require('webpack-hot-middleware');
var config = require('./webpack.config');

var app = new require('express')();
var port = 3000;

var compiler = webpack(config);
app.use(webpackDevMiddleware(compiler, { noInfo: true, publicPath: config.output.publicPath }));
app.use(webpackHotMiddleware(compiler));
app.use(function(req, res) {
  res.sendFile(__dirname + '/index.html');
});

app.listen(port, function(error) {
  if (error) {
    console.error(error);
  } else {
    console.info("==> 🌎  Listening on port %s. Open up http://localhost:%s/ in your browser.", port, port);
  }
});
```

本來想說測試一下，直接 run 一下 node server.js 看看有沒有問題，結果就出現了下面的錯誤：  
  
```
==> 🌎  Listening on port 3000. Open up http://localhost:3000/ in your browser.
webpack built 9cc6533f9b26bec5e1ce in 190ms
Hash: 9cc6533f9b26bec5e1ce
Version: webpack 1.12.2
Time: 190ms
    Asset   Size  Chunks       Chunk Names
bundle.js  45 kB       0       main
chunk    {0} bundle.js (main) 7.69 kB [rendered]
    [0] multi main 40 bytes {0} [built] [1 error]
    [1] (webpack)-hot-middleware/client-overlay.js 883 bytes {0} [built]
    [2] (webpack)-hot-middleware/client.js 3.27 kB {0} [built]
    [3] (webpack)-hot-middleware/~/strip-ansi/index.js 161 bytes {0} [built]
    [4] (webpack)-hot-middleware/~/strip-ansi/~/ansi-regex/index.js 145 bytes {0} [built]
    [5] (webpack)-hot-middleware/process-update.js 2.93 kB {0} [built]
    [6] (webpack)/buildin/module.js 251 bytes {0} [built]

ERROR in multi main
Module not found: Error: Cannot resolve 'file' or 'directory' ./index in /Volumes/RamDisk/redux-study
 @ multi main
```

最後試著試著發現，本來以為是我漏掉了 index.html，但最後發現，其實 index.js 才是最重要的  
當我做了下面的動作  

```bash
touch index.js
```

再執行 node server.js，感覺上就正常了！   

```
==> 🌎  Listening on port 3000. Open up http://localhost:3000/ in your browser.
webpack built 1e04a7ed2baa7052b68b in 771ms
webpack building...
webpack built 1e04a7ed2baa7052b68b in 8ms
```

### index.html

其實，因為 React 算是走 SPA 路線的，所以其實首頁很簡單，可以像下面這樣   

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Redux Study</title>
  </head>
  <body>
    <div id="root">Hello
    </div>
    <script src="/static/bundle.js"></script>
  </body>
</html>
```

其中，javascript 是使用了一個目前還沒產生的檔案  
不過，透過 [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware)，這個檔案在你執行 server.js 的時候會動態產生  
所有檔案，都會 on-fly 在記憶體中處理，不需要像以前一樣產生一堆暫存檔案  
  
不過，記得這只是在開發階段方便而已，Production 階段還是要產生實體的 bundle.js 檔案  

就如同  [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) 的 README 就說到：  
> THIS MIDDLEWARE SHOULD ONLY USED FOR DEVELOPMENT!
> DO NOT USE IT IN PRODUCTION!

## 第一天結後心得

其實，對不太熟悉 javascript 開發的人來講，像我這樣，準備環境比較不容易  
尤其我記得我有印象的，還是在 grunt/gulp 階段～  
不過才沒幾個月，一堆工具又出來了，雖然是為了 ES6 而產生的工具，但是還是覺得太多東西要學了   
 [webpack](https://webpack.github.io) 去年就聽過了，不過聽不太懂，所以本來是要放棄的。但是現在感覺好像又不是我之前聽過的那樣了～～～   
變化真快     
  
之後可能還要對 [webpack](https://webpack.github.io) 多多了解一下....    

至於 Redux 本身，大概又是另外一個難解的問題，需要好好了解了～  