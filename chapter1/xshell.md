# Xshell

只要是使用linux的工程師，一定都會有一套自己順手的terminal tool，我一直以來都是putty掛的，我覺得可以儲存連線資訊、能夠做log，偶而改改刺眼或難讀的顏色，這樣就很夠用了。

* 碰到要多開怎麼辦?
  * 開很多個視窗
* 碰到要傳檔案怎麼辦?
  * 多開一個WinSCP

這兩個問題我覺得用最傳統的解決方法就好。但當視窗越開越多，切換點來點去實在很麻煩，這時候工程師病就上身了，會想用熱鍵解決；另一方面，明明眼前已經有一個連線的SSH可以用，還要再多開一個WinSCP總覺得很多餘，還要多一套連線管理機制，這邊改了金鑰另外一邊也要跟著改。

基於這兩個原因，我換到Xshell，有分頁管理連線，所以可以用熱鍵切換，Alt+&lt;n&gt;。能夠支援ZMODEM傳送/接收檔案，也可以直接使用拖拉的方式上傳，這實在太方便了。

要能夠支援ZMODEM要能夠使用`rz`，在CentOS上可以透過yum安裝：

> yum install lrzsz

另外有些使用設定必須要調整才能夠as close as putty：

1. Properties &gt; Terminal &gt; VT Modes：把DECNKM改成Set to normal
2. Options &gt; Keyboard and Mouse：把Right button改成Paste the clipboard contents
3. 把下面的Copy selected texts to the clipboard automatically勾起來

這樣就可以即框即複製，也可以右鍵貼上，還支援數字鍵盤。

最後我還發現一點，putty的default color scheme有夠醜，那個藍色根本看不到，那個紅色/紫羅蘭色完全在挑戰視力，但Xshell的顏色預設就很漂亮了，這點算是意外發現。

當要從guest傳送檔案回host時，並沒有比較方便的UI可以使用，必須透過`sz`來做，只需要：
> sz ${filename}

就會開啟檔案對話窗格，指定路徑後就會將剛輸入的檔案放進去，非常方便。