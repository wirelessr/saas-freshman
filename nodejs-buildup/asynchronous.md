# Asynchronous
有許多人無法理解同步和非同步的差別，這邊以去餐廳吃飯來舉個例子。

## 同步
排隊點餐，點完餐，站在櫃檯等餐來，領完餐才離開櫃檯。

## 非同步
排隊點餐，點完餐拿著號碼牌走掉，櫃檯繼續服務下個客人，餐點送來依號碼牌取餐。

從上述例子來看，同步處理是線性的，一件事做完才會做下一件事；但非同步是服務生在點單，而後面的廚房同時在備餐。非同步處理與平行處理不同，平行處理指的是櫃檯有很多個服務生，可以同時點一堆單，但是怎麼取餐還是回到同步或非同步的處理。

在Node.js，所有的I/O都是非同步的，也就是I/O call出去，並不會等到I/O回來才接著做，而是會接著做下面的task。要寫Node.js一定要先有這個認知，例如：

```js
var fs = require("fs");
var file_str = fs.readFile("./foo.txt");
console.log(file_str); // undefined
```

以上的file_str是未定義，因為readFile是非同步，所以檔案還沒讀完就已經執行到`console.log`。正確的做法是：

```js
var fs = require("fs");
var file_str = fs.readFile("./foo.txt", function(err,result){
    if(err) console.log(err);
    else console.log(result); // foo.txt content
});
```

但若是一連串的非同步操作，例如：我要讀的檔案A被記錄在檔案B內，而檔案B又必須從網路上取得，網路的URL又被記錄在設定檔案。這樣會一層接著一層的callback function。有一個專有名詞叫做回呼地獄(callback hell)。在javascript的ES2016出了一個功能是Promise，用來解決回呼地獄，而ES2017出了async/await來使Promise更容易調用。

但即便是async/await我還是覺得那個使用方式很繁瑣，更重要的是error handle不是很直覺，必須要try/catch才有辦法。因此我個人更偏好使用外部的async module：
> npm install async

在async模組中提供了許多流程控制的API，例如waterfall/series/parallel等，這邊舉waterfall作為例子。

```js
const Async = require('async');

function asyncio(msg, next, err, res) {
    setTimeout(function(){
        console.log('asyncio: '+msg);
        next(err, res);
    }, 1000);
}

function done(err, final_result) {
    console.log('!DONE!');
    console.log('!ERROR! '+err);
    console.log('!RESULT! '+final_result);
}

function cb(err, final_result) {
    console.log('!CALLBACK! '+final_result+', err: '+err);
}
```

這邊先準備一個偽裝的非同步操作`asyncio`，雖然只是setTimout，但其實非同步API做的事也大同小異。`done`則是最後的error handler。首先說明waterfall的運作原理。
> async.waterfall([function, function, ...], function);

原型是如上所述，waterfall會將列表內的function按照順序執行，而每一個function會有一個控制流程用的API，用來傳遞錯誤和結果。
> function(err, result)

若是任何一個流程控制的API有收到error，那個就會執行waterfall最後的的error handler。

```js
function wf_step1_stop(callback) {
    Async.waterfall([
        function _step1(next) {
            err1 = new Error('Err step1');
            asyncio('step1', next, err1, 'result step1');
        },
        function _step2(result1, next) {
            err2 = new Error('Err step2');
            asyncio(result1+' in step2', next, err2, 'result step2');
        },
        function _step3(result2, next) {
            asyncio(result2+' in step3', callback, null, 'result in step3');
        }
    ], done);
}
wf_step1_stop(cb);
```

上面的`wf_step1_stop`執行到step1就因為錯誤而結束了，所以在step3的callback並不會被執行到。想要執行callback就必須確保step1和step2能夠正確執行。

```js
function wf_callback_without_done(callback) {
    Async.waterfall([
        function _step1(next) {
            asyncio('step1', next, null, 'result step1');
        },
        function _step2(result1, next) {
            asyncio(result1+' in step2', next, null, 'result step2');
        },
        function _step3(result2, next) {
            asyncio(result2+' in step3', callback, null, 'result in step3');
        }
    ], done);
}
```

但這樣的做法並不好，在async的文件中提到，若是callback在列表內的function就被執行到，必須要在前面加上return才可以避免在一些corner case發生不可預期的行為。也因此，正確的practice應該是把callback放在done內執行，才可以確保既有error handle又能夠處理callback的邏輯。

```js
function wf_done(callback) {
    Async.waterfall([
        function _step1(next) {
            asyncio('step1', next, null, 'result step1');
        },
        function _step2(result1, next) {
            asyncio(result1+' in step2', next, null, 'result step2');
        },
        function _step3(result2, next) {
            asyncio(result2+' in step3', next, null, 'result in step3');
        }
    ], (err, res) => {console.log('!DONE! err: '+err+', result: '+res); callback(err, res); });
}
```

在async的文件中還有介紹許多關於流程控制的API，請看註1。另外可以透過註2更加理解非同步的流程控制基礎。

[1] [async document](https://caolan.github.io/async/docs.html#controlflow)
[2] [control flow](http://book.mixu.net/node/ch7.html)