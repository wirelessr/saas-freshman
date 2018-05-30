# Record models
有了user model，接下來就是記帳軟體的核心，會計。但根據MVC的分工原理，model只負責資料的存取，因此會計模組與使用者模組很相像，所有做統計的運算都歸類在Controller內，之後的章節會仔細介紹。

# 插入一筆資料
使用者的表是預先建立的，但每個使用者要儲存的數據資料不會放入使用者的總表，而是會放在每個使用者專屬的獨立空間，一般稱為user profile，也就是隨著使用者建立而產生；使用者消失而摧毀。刪掉profile的程式在使用者模組內，這邊介紹建立profile的流程，同時也會介紹設定DB schema的作法：

```js
var HEADER = ['date', 'cost', 'note', 'type'];

module.exports.create = function(uid, data, callback) {
    async.series([
        setAuth,
        function _getWorksheet(step) {
            doc.getInfo(function(err, info) {
                records = info.worksheets.find(function(element) {
                    return element.title == uid;
                });
                step();
            });
        },
        function _checkWorksheet(step) {
            if(!records) {
                doc.addWorksheet({
                    title: uid,
                    headers: HEADER
                }, function(err, sheet){
                    records = sheet;
                    step();
                });
            } else {
                step();
            }
        },
        function _create(step) {
            data.date = (new Date()).getTime();
            records.addRow(data, callback);
            step();
        }
    ], function(err) {
        if(err) {
            console.log(err);
        }
    });
}
```

當使用者要插入一筆數據的時候，會循著這個流程：
1. 認證試算表
2. profile是否存在
3. 不存在則建立profile。title是table name而headers則是DB schema
4. 插入數據

# 讀取資料
在實做讀取資料時有一個很重要的地方，google試算表具有基本的關聯資料庫功能，例如排序、鎖定範圍等，因此讀取資料的API有一個參數query_params用來設定讀取範圍。

```js
module.exports.retrieve = function(uid, query_params, callback) {
    async.waterfall([
        setAuth,
        function _getWorksheet(next) {
            doc.getInfo(function(err, info) {
                var record = info.worksheets.find(function(element) {
                    return element.title == uid;
                });
                next(err, record);
            });
        },
        function _retrieve(record, step) {
            if(record) {
                record.getRows(query_params, function(err, rows){
                    rowdata = [];
                    for(i=0; i<rows.length; i++) {
                        rowdata.push({
                            date: rows[i].date,
                            cost: rows[i].cost,
                            type: rows[i].type,
                            note: rows[i].note,
                            income: rows[i].income,
                            });
                    }
                    callback(err, rowdata);
                });
            }
            else {
                err = new Error('Not existed');
                err.status = 404;
                throw err;
            }
        }
    ], function(err) {
        if(err) {
            console.log(err);
        }
    });
}
```

輸入的query_params是一個object：
```js
query_params = {
    offset: Int, //從第幾筆開始，通常為0
    limit: Int,  //最多讀取幾筆資料，不設
    orderby: '', //給定column name可以排序
    reverse: Boolean, //降冪排序
    query: '',   //非常強大，例如：'a > 100 and b < 100 or c == 100'
};
```

而回傳的rowdata則是一個列表，放入讀取到的資料，在restful API來取時，會被轉成JSON。