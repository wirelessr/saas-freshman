# Date
在進入前端開發之前先介紹這次使用到與Date相關的作法。本站需要許多與時間有關的操作，為了方便使用google試算表的*structured query*，所有與時間相關的操作都會使用timestamp而不是Date，因為在query資料庫時必須給予時間區間，才有辦法把對應的資料撈回前端運算。取得本節會介紹：

1. 取得當日起始和結束時間
2. 取得當月起始和結束時間
3. 取得當月每週起始和結束日期
4. 取得當月每週起始和結束時間

# 當日起始和結束時間

先取得當下的Date，接著把小時、分鐘和秒清空，這邊採取稍微偷懶的作法，直接轉成ISO時間字串yyyy-mm-dd，這樣再轉回Date就會沒有小時、分鐘和秒了。接著把day+1就會是下一天的時間，如此`start_ts <= ts < end_ts`就會是當日的起始到結束。

```js
var todayDate = new Date().toISOString().slice(0,10);
var date = new Date(todayDate);
var start_ts = date.getTime();
date.setDate(date.getDate() + 1)
var end_ts = date.getTime();
```

# 當月起始和結束時間

當月起始和結束與當日差不多，也是先取出ISO字串，轉成Date後將月份+1即可得到結果。

```js
var todayMonth = new Date().toISOString().slice(0,7);
var date = new Date(todayMonth);
var start_ts = date.getTime();
date.setMonth(date.getMonth() + 1)
var end_ts = date.getTime();
```

# 當月每週的起始和結束日期

這個稍微麻煩一點，就直接上code了。

```js
function getWeeksInMonth(month, year){
   var weeks=[],
       firstDate=new Date(year, month, 1),
       lastDate=new Date(year, month+1, 0),
       numDays= lastDate.getDate();

   var start=1;
   var end=7-firstDate.getDay();
   while(start<=numDays){
       weeks.push({start:start,end:end});
       start = end + 1;
       end = end + 7;
       if(end>numDays)
           end=numDays;
   }
    return weeks;
}
```

# 當月每週起始和結束時間

取得當週的時間必須透過getWeeksInMonth()轉成timestamp。還有一點要注意，這邊採用yyyy-mm來串接整數，若是沒有將整數補零，會使字串不符合ISO規範，因此還需要實做padding。

```js
Number.prototype.pad = function(size) {
  var s = String(this);
  while (s.length < (size || 2)) {s = "0" + s;}
  return s;
}
```

```js
var var isoy_m = isodate.split('-');
var weeks = getWeeksInMonth(isoy_m[1]-1, isoy_m[0]);
var weeks_ts = weeks.map(function(w) {
    var week_ts = {};
    week_ts.start = new Date(isodate+'-'+w.start.pad(2)).getTime();
    endday = new Date(isodate+'-'+w.end.pad(2));
    endday.setHours(24,0,0,0);
    week_ts.end = endday.getTime();
    return week_ts;
});
```

# 後話
這邊所有的時間處理都是timestamp，也就是UTC時間，並沒有考慮到localtime的情況，也就是若使用者處在有timezone的時間區段，這邊計算出來的時間都會有誤差，其實這是可以補救的，因為這些時間的運算都跑在前端，也就是說，是可以補正時區的，但這邊並沒有加入時區的運算。

# Updated at 2018-06-08
意外的發現一個很好用的Date prototype，他用replace和regexp來處理時間的format，表現能力意外的好，可以選擇要用**/**或者**-**來分隔，也可以給定不同精度的時間格式。

```js
// 对Date的扩展，将 Date 转化为指定格式的String
// 月(M)、日(d)、小时(h)、分(m)、秒(s)、季度(q) 可以用 1-2 个占位符， 
// 年(y)可以用 1-4 个占位符，毫秒(S)只能用 1 个占位符(是 1-3 位的数字) 
// 例子： 
// (new Date()).Format("yyyy-MM-dd hh:mm:ss.S") ==> 2006-07-02 08:09:04.423 
// (new Date()).Format("yyyy-M-d h:m:s.S")      ==> 2006-7-2 8:9:4.18 
Date.prototype.Format = function (fmt) { //author: meizz 
    var o = {
        "M+": this.getMonth() + 1, //月份 
        "d+": this.getDate(), //日 
        "h+": this.getHours(), //小时 
        "m+": this.getMinutes(), //分 
        "s+": this.getSeconds(), //秒 
        "q+": Math.floor((this.getMonth() + 3) / 3), //季度 
        "S": this.getMilliseconds() //毫秒 
    };
    if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
    for (var k in o)
    if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
    return fmt;
}
```

用法也很單純，只需要直接在時間物件上執行Format即可：

```js
var time1 = new Date().Format("yyyy-MM-dd");
var time2 = new Date().Format("yyyy/MM/dd HH:mm:ss");  
```
