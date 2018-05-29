# 快速上線

在本節我們來快速把一個站架起來上線，首先說明一下整個快速架站的流程：

1. 開一個空的github repo
2. 開一個新的heroku app
3. 設定heroku綁定github做自動deploy![](/assets/5.png)
4. 假設你已經選好目錄了，使用express產生器快速生成一個prototype。註1
5. 進入產生器建立的目錄，`npm install`把相依套件安裝完成
6. (optional) `npm start`然後就可以去`localhost:3000`看看有沒有正常架起來
7. `git remote add origin $(github_path)`
8. git push origin master
9. 可以去xxx.herokuapp.com看結果了

總共九個步驟，就可以讓一個網站快速上線。

[1] 要使用express產生器需要先安裝再使用：

> npm install express-generator -g
> express --view=pug myapp

這邊我們使用pug這個模板框架，但這其實不重要，因為我們不需要自己寫HTML，本站的HTML大部分都來自W3C提供的template，只需要做一些小修改即可使用。而pug這個模板框架算是現在很常使用的mockup language，只需要敲幾個鍵就可以快速生成HTML，例如：

```
html
  body
    p Hello world
```

這樣就有一個Hello world的HTML網頁了。