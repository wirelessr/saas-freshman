一般畫統計圖表的選擇是D3，另外還有一個將D3再包裝一次C3。C3的好處在於他的使用較D3更為簡潔，因為封裝過，所以不需要考慮底層的實做，幾乎是只要把資料倒入即可；但C3也不是沒有壞處，因為封裝過，所以要修改樣式遠比D3麻煩。

本站使用了三種不同的統計圖表：

1. 圓餅圖
2. 長條圖
3. 橫向長條圖

本節會分別介紹三種圖表的作法。

# 圓餅圖
本站的圓餅圖既不是C3也不是D3，而是使用google提供的圖表工具，語法比起C3又更為簡便，因此圓餅圖使用[這個](https://www.gstatic.com/charts/loader.js)。使用google chart要先設定loader和callback。

```js
google.charts.load('current', {'packages':['corechart']});
google.charts.setOnLoadCallback(drawChart);
```

接著在callback內部取得資料和畫圖。首先一樣是從利用XHR取得資料，然後填入圓餅圖的資料列表中，形式是[[type1, cost1], [type2, cost2], ...]，接著設定圖形的寬度和標題，最後是render在DOM上。語法很單純，不需要考慮太多，資料整理好、指定寬度、指定DOM就可以畫出來了。省去還要寫CSS、寫style的麻煩。

```js
var list = [['type', 'cost']];
for(i in summary) {
    list.push([types[i], summary[i]]);
}
var data = google.visualization.arrayToDataTable(list);

// Optional; add a title and set the width and height of the chart
var options = {'title':'月結', 'width':'100%'};

// Display the chart inside the <div> element with id="piechart"
var chart = new google.visualization.PieChart(document.getElementById('piechart'));
chart.draw(data, options);
```