# MVC
現在主流的網站框架都會將Model, View, Controller拆開，也就是所謂的MVC架構。Model負責所有的資料存取，也就是存取資料庫的抽象層，本站是用低成本、快速架站的方式，因此並沒有使用任何線上的資料庫服務，取而代之的是使用google試算表作為資料存放的地方。

# 準備
1. 必須要有一個google的帳號
2. 進入[Google API控制台](https://console.developers.google.com/)，開啟一個新的專案
3. 在`Credential`的地方創建一個`Service account key`
4. 此時會自動下載一個json file，找到裡面的`client_email`
5. 將client_email加入試算表的共用作者![](/assets/Selection_002.png)
6. 上方塗黑的部份是試算表的id，一併紀錄
7. 將`client_email`、`private_key`和`id`存進本機和heroku的環境變數

```bash
export GOOGLE_ACCT_EMAIL="$(client_email)"
export GOOGLE_PRIVATE_KEY="$(cat ./private.key)"
export GOOGLE_SHEET_ID="$(id)"
```

上面提到的`./private.key`的產生過程是將xxx.json內的`priavte_key`寫到一個檔案內，接著把所有的`\n`變成換行。看是要手動還是自動：
> sed -i 's/\r/\n/g' ./private.key

要設定heroku的環境變數，首先要先安裝heroku api，安裝說明在[heroku doc](https://devcenter.heroku.com/articles/heroku-ci):
接著`heroku config:add GOOGLE_PRIVATE_KEY="$(cat ./private.key)" -a $(app_name)`，依此類推。

# Database Schema
任何database都有schema，即便是NoSQL，也需要描述資料長相。使用google試算表作為資料庫有個好處，schema非常容易定義，只需要將試算表的第一行的每個欄位設定名字即可。
在本例中，請將試算表的的tab命名為users，這意義相當於SQL的table name；接著將第一行分別設為：
> username	password	email	uid

也就是SQL的column name。

# 開始實做user models
首先先安裝存取google試算表的邏輯層和async，在後面的章節*Asynchronous*會仔細介紹async：
> npm install google-spreadsheet async --save

開一個新檔的資料夾`models`，本站所有的model都會放在這個資料夾內。以下是`models/users.js`的實做。

```js
var GoogleSpreadsheet = require('google-spreadsheet');
var doc = new GoogleSpreadsheet(process.env.GOOGLE_SHEET_ID);
var async = require('async');
```
我們需要引入剛安裝的google-spreadsheet和async，並且透過剛設定的環境變數產生一個doc的實體。接著新增兩個function，使用doc進行認證，並且讀取users這張table。

```js
function setAuth(step) {
    var creds_json = {
        client_email: process.env.GOOGLE_ACCT_EMAIL,
        private_key: process.env.GOOGLE_PRIVATE_KEY
    }

    doc.useServiceAccountAuth(creds_json, step);
}

function getInfoAndWorksheets(next) {
    doc.getInfo(function(err, info) {
        users = info.worksheets.find(function(element) {
            return element.title == 'users';
        });
        next(err, users);
    });
}
```

上面使用到的`client_email`和`private_key`就是我們剛寫進環境變數內的值，在`getInfoAndWorksheets`讀取名為users的那張表。在各種user model中有四個最常見的API：
1. create user
2. authenticate
3. find user
4. delete user
因此我們的users.js要實做這四個API。

```js
module.exports.create = function(userdata, callback) {
    async.waterfall([
        setAuth,
        getInfoAndWorksheets,
        function _create(users, step) {
            users.getRows({
                query: 'username == '+userdata.username
            }, function( err, rows ){
                if(rows.length) {
                    err = new Error('Already existed');
                    err.status = 403;
                    callback(err);
                } else {
                    userdata.uid = _uuid();
                    users.addRow(userdata, function(){});
                    callback(err, userdata);
                }
            });
        },
    ], function(err){
        if( err ) {
            throw(err);
        }
    });
}
```

這邊列出create，剩下三個基本上大同小異，都是以setAuth開始和google試算表認證，接著取出users這張表，從表內查詢是否有username存在，接著做相對應的處理。傳進來的userdate是一個object，若是需要更多屬性，只要新增試算表內的DB schema，就可以擴充，非常方便：
```js
userdata = {
    username: '',
    password: '',
    email: '',
};
```
傳回去時會增加一個新的屬性`uid`，在本例中是使用UUID v4。
```js
function _uuid() {
    var d = Date.now();
    if (typeof performance !== 'undefined' && typeof performance.now === 'function'){
        d += performance.now(); //use high-precision timer if available
    }
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
            var r = (d + Math.random() * 16) % 16 | 0;
            d = Math.floor(d / 16);
            return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
            });
}
```

這邊特別講解一下`_uuid`這個function。在javascript中，並沒有能夠產生UUID的API，因此必須自己實做，根據RFC4122的定義，UUID是一個用dash隔成五組的字串，分別是8、4、4、4、12個字元，其中第三組的第一碼定義了UUID的版本，目前通用的是4。

剩下三個API請參考[github](https://github.com/wirelessr/accounting-apps/blob/master/models/users.js)