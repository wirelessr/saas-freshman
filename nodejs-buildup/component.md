# 元件
在本節會介紹要將本站架起來的基本需求，並且列出所有使用的套件。

## 網路空間
heroku。本站使用heroku這個PaaS，且只使用免費帳號；免費帳號有些限制，例如：只有500MB的使用空間，並且每天只保證上線16小時，但對於記帳來說，很夠用了，畢竟不會一天到晚在花錢。

## 資料庫
Google spreadsheet。因為需要一個空間作為使用者資料與會計資料的存放地點，且要好查詢好輸入，一個資料庫是在所難免的。但這畢竟是一個小專案，不需要花錢買資料庫，雖然heroku有免費的資料庫方案，但一方面是空間有限，一方面是需要信用卡登記。重申，這是一個簡易網站，沒有太多安全性設計，萬一網站被駭，有人大量創建使用者帳號並且輸入資料，資料庫的免費額度會很快用光，而自動轉為計費模式。基於上述理由，我使用google的試算表作為資料庫空間，後續章節會告訴你如何將試算表當成資料庫。首先你需要一個google帳號。

## 安裝Node.js
這在Node.js的官網很容易可以找到SOP，若是用mac，直接`brew install node`或者ubuntu，直接`apt install node`即可。接著在想要的位置`npm init`就可以迅速產生一個專案開始開發了。

## nodemon
有開發網站經驗的人會知道，在開發過程中，會需要不斷restart server然後reload page，這是一個很煩的過程。所幸，Node.js有一個配套方案，稱為`nodemon`，他可以自動幫你監測網站是否修改，並且自動重啟server，開發Node.js必備。安裝方法如下：
> npm install nodemon -g

`-g`的意思是安裝到本機，而不只是在這個專案內。裝好後若是專案內的`npm start`已經正確設定了，那麼只要在專案目錄下`npx nodemon`就會開啟自動監測。

## mocha
mocha是一個Node.js用的單元測試框架，算是現在主流的套件，必裝，安裝方法一樣是：
> npm install mocha -g

## 網站框架
Express。開發後端有些很繁瑣的工作，例如：API routing、解析query string/header/body、session token等，框架可以將這些常見問題做好封裝，以便於快速進行開發。而Node.js最常使用的網站框架就是Express.js，安裝方式是：
> npm install express --save

這邊不再是使用`-g`而是使用`--save`，因為不一定每個專案都需要express，所以不需要裝在全域，只需要裝在專案內即可。

## Github (optional)
有一個github帳號並不是標準配備，只是github搭配heroku非常方便，只要在heroku app內的deploy開關設定與github連動，那麼被推到github上的code會被自動deploy到heroku上，就不需要使用heroku cli重複登入重複push。
 
以上是這次快速架站會用到的相關套件，還有些未提及的會在之後章節詳述。



