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

# 長條圖
無論是水平的還是直的長條圖，都是使用D3來繪製的。過程中都碰到一個很麻煩的事，要如何決定寬度或高度；在記帳主頁，下方的報表區被分割為三塊，使得能夠繪製的長寬都是有限的，如何指定長寬成為繪製長條圖最重要的關卡。為了繪製長條圖，首先先要訂製長條圖使用的style。這邊展示了兩種不同的方式，#barChart是水平長條圖；而#vchart是垂直的。

```css
.barChart{
	font-family:verdana;
	font-size: 13px;
}
.barChart .item {
	float: left;
	height: 40px;
	line-height: 40px;
	margin-bottom: 5px;
	text-align: left;
	width: 100%;
}
.barChart .bar {
	background-color: DarkRed;
	color: #fff;
	float: left;
	height: 40px;
	text-align: right;
	margin-right: 10px;
	padding-right: 10px;
}
.vchart{
	border-bottom-style:solid;
	border-bottom-width:5px;
	padding-left: 6px;
}
```

當準備好style之後就進入實際餵資料和把每一個長條畫出來的步驟。

## Horizontal
水平長條圖的做法如下，首先要餵進去兩個參數，data是一個JSON格式的資料，長相是：
> data = [{type1, cost1}, {type2, cost2}, ...]

還需要所有cost中的最大值，以便能夠算出水平長條的長度。假設整個要用來繪製長條圖的寬度是100%(定義在css)，那麼實際用來繪製長條的寬度只保留80%，剩下的20%要用來顯示文字。而每一個長條所佔比例的公式即是：
> cost/max * 80%

```js
function draw(data, max) {
  d3.select('.barChart') //選擇放在barChart這個div容器裡面
  .selectAll('div') //選取".barChart"範圍內的所有的div
  .data(data) //將資料加入div
  .enter() //傳入資料
  .append('div') //放到畫面上
  .attr('class','item') //將剛剛放到畫面上的div，加上class "item"
  .text(function(d){return d.type}) //加上文字描述，使用json檔案裡面的 "type" 欄位
  .append('div') //加入包含資料的div，這個div是用來畫圖用的
  .text(function (data) {
      return data.cost; //畫圖用div加上文字描述，使用json檔案裡面的 "cost" 欄位
  })
  .attr('class','bar') //畫圖用div加上class "bar"
  .style('width', function(d){
      return (d.cost)*80/max  + '%'
  });
};
```

## Vertical
至於垂直長條圖就複雜了些，因為要決定每個長條的起始位置和寬度，例如：兩筆資料時，第一個長條從0開始，寬度是50%，而文字要從25%開始，而第二根則是起始在50%，文字從75%開始。所以有三個值是需要根據資料數量做計算：

1. 長條起始位置
2. 長條寬度
3. 文字起始位置

此外，垂直長條圖的高度有最高限制，不然若是碰到數據比例差距過大，某個長條會佔據過多版面，因此還需要計算每一根長條所能使用的高度；另一點需要注意，對於長條圖來說最底端是0，因此繪製高度是`height - (d/max * height*0.8)`，此處0.8的目的與水平長條圖相同，需要有20%的高度用來放文字。至於dataset，則相對單純，因為只需要標註數字，不需要文字註記，所以不需要是JSON，只要一個數字陣列即可。

```js
function vBarChart(dataset) {
const max = Math.max(...dataset);
const height = 300;

var chart = d3.select('.vchart')
	.attr("width", '100%')
	.attr('height', height)
	

var bar = chart.selectAll("g")
	.data(dataset)
	.enter().append("g")

bar.append("rect")
    .attr("y",function(d,i){return height-(d/max*(height*0.8));})
    .attr("x",function(d,i){
         return i * 100/dataset.length+'%'; //position
    })
    .attr("height",function(d){
         return (d*3);
    })
    .attr("width",100/dataset.length+'%') //width
    .attr("fill","#5F4B8B")
	.attr("stroke-width",2)
	.attr("stroke","black");

bar.append('text')
	.attr('y',function(d){return height-(d/max*(height*0.8))+21;})
	.attr("x",function(d,i){
         return (i * 100/dataset.length + 50/dataset.length)+'%'; //position
    })
	.style('fill', '#F00')
	.style('font-size', '18px')
	.style('font-weight', 'bold')
	.style("text-anchor", 'middle')
	.text(function(d){
	return d;}
		 );

bar.append('text')
	.attr('y',function(d){return height*0.1})
	.attr("x",function(d,i){
         return (i * 100/dataset.length + 50/dataset.length)+'%'; //position
    })
	.style('fill', '#000')
	.style('font-size', '18px')
	.style('font-weight', 'bold')
	.style("text-anchor", 'middle')
	.text(function(d,i){
	return i+1;}
		 );
}
```