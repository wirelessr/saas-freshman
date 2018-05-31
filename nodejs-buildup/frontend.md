# 前端
本站的兩個主頁：使用者登入和記帳首頁，分別來自[W3 Layout](https://w3layouts.com/)和[W3.CSS](https://www.w3schools.com/w3css/w3css_templates.asp)；使用者登入頁面是W3 Layout下載後直接放入的。而記帳首頁則是W3.CSS的最後一個範例，將中間不必要的部份刪除並做了少部份修改而成。在本節，會稍微介紹修改的部份，而下一節則是會介紹各種圖表的畫法。

# 開始記帳
首先比較大修改的是開始記帳按下後彈跳出來的視窗，他加入了一個輸入的form和一個可以切換模式的panel。
切換模式的panel構造很單純，一個onclick的event handler，搭配W提供的class。

* w3-border w3-round-xlarge : 加框，而且是圓框
* w3-half : 佔比一半
* w3-border-red : 當下模式標紅框

```html
<h5>
  <div id="outcome" onclick="switchmode(this.id);" class="w3-panel w3-border w3-round-xlarge w3-half w3-hover-border-red w3-border-red">支出</div>
  <div id="income" onclick="switchmode(this.id);" class="w3-panel w3-border w3-round-xlarge w3-half w3-hover-border-red">收入</div>
</h5>
```

至於event handler要做的事情也不複雜，將當下模式設成紅框，非當下模式的紅框拔掉，並且把對應的form給render出來。比較值得留意的是操作class的DOM API是`classList.add`和`classList.remove`。

```js
var items = ['income', 'outcome'];                                 
var inputform;
items.forEach(function(i) {                                        
    if(i == item) {
        document.getElementById(i).classList.add('w3-border-red'); 
    } else {
        document.getElementById(i).classList.remove('w3-border-red');
    }                                                              
});
```

# 日報表

日報表的部份有兩個重點，一個是table的render取決於當下#todayDate上的日期，另一個是日期可以經由左和右兩個按鈕來增減，所以會有一個onclick的event handler。

```html
<div class="w3-third">
  <div class="w3-card w3-container" style="min-height:460px">
    <h3 id="todayDate">今天</h3>
    <br />
    <table class="w3-table w3-striped w3-bordered">
    <thead>
    <tr class="w3-theme">
    <th>項目</th> 
    <th>類別</th> 
    <th>花費</th> 
    </tr>
    </thead>
    <tbody id="todayReport">
    </tbody>
    </table>
    <div class="w3-bar">
      <a onclick="shift_day(-1)" class="w3-bar-item w3-button w3-hover-theme">«</a>
      <a onclick="shift_day(1)" class="w3-bar-item w3-button w3-hover-theme">»</a>
    </div>
  </div>
</div>
```

至於event handler的實做則是先取得今天日期的ISO字串後根據按下左還是右來增減，接著把ISO字串設回#todayDate，最後是render table。要先處理完#todayDate的顯示日期才進行render table，因為render table依然是要看#todayDate的日期來決定呈現哪一天的資料。

```js
function shift_day(offset) {
    var isodate = document.getElementById("todayDate").innerHTML;
    var date = new Date(isodate);
    date.setDate(date.getDate() + offset);

    isodate = date.toISOString().slice(0,10);
    document.getElementById("todayDate").innerHTML = isodate;
    list_daily_report(isodate);
}
```

# Render Table

要把日報表的資料撈出來採用XHR，對後端的`/list/:type/:range*?`下GET，將時間區間內的資料撈回來。撈回來的資料是JSON的格式，會把資料庫中符合時間區間的每筆record倒出來，前端將其塞入table內。而本站使用的icon是[Font Awesome](https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.3.0/css/font-awesome.min.css)提供的。

```js
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
        var rowdata = JSON.parse(this.responseText);
        var table = ''; 
        for(i in rowdata.rows) {
            if(rowdata.rows[i].cost > 0) {
                table += '<tr>';
                table += '<td>'+rowdata.rows[i].note+'</td>';
                table += '<td><i class="fa '+icons[rowdata.rows[i].type]+'"></i></td>';
                table += '<td>'+rowdata.rows[i].cost+'</td>';
                table += '</tr>';
            }
        }
        document.getElementById("todayReport").innerHTML = table;
    }
};
xhttp.open("GET", "/list/time/"+start_ts+'-'+end_ts, true);
xhttp.send();
```

本站的排版皆是使用W3提供的w3-half或w3-third，好處是w3會將各種不同的螢幕尺寸考慮進去，進而呈現適合各種裝置的版面配置。

完整的程式碼在[github](https://github.com/wirelessr/accounting-apps/blob/master/templateLogReg/js/accounting.js)