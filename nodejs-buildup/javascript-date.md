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

取得當週的時間必須透過getWeeksInMonth()轉成timestamp。

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
