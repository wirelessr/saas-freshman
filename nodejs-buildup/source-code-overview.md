在開始修改之前我們先快速的掃過express產生器建出來的東西，了解概況對於之後開發會更有效率。

```
myapp
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```

* app.js用來設定middleware和各個子模組啟始的routing。
* bin/www是一個執行用的js，在`npm start`的時候其實就是呼叫`node bin/www`
* package.json所有使用的套件都紀錄在這，也就是`npm install --save`的結果
* public用來放所有前端要用的東西，例如：HTML/CSS/JS
* routes跟routing相關的code都放這
* views用來放pug的模板檔，看不習慣可以刪掉，之後用不到

接著我們攤開`app.js`來稍微了解一下Node.js的基礎運作。首先是各個使用到的模組宣告，自己寫的routing模組：index和users，也是這邊宣告。

```js
var createError = require('http-errors');
var express = require('express');
var cookieParser = require('cookie-parser');
var path = require('path');
var logger = require('morgan');

var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');
```

接著是用`app.use`定義各種要使用的middleware。`express.static`定義了哪一個目錄會被使用者存取到，以本例來說就是public這個資料夾，他會落在根目錄下的public內。

```js
var app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
```

若是要設定routing則是像下面這樣，根目錄使用`index.js`而根目錄後接著/users開頭的url則是進入`user.js`。

```js
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

列出來的這幾處都會改動到，在下一節正式進入記帳網站的開發。

