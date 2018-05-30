# Session
至此已經完成了整個data model並且也經過單元測試驗證，接下來要將restful API與資料庫連結在一起，並且加入session的機制。本站使用者的session機制是cookie，單純只有cookie有一個很大的缺點，後端無法更深入控制使用者的行為，因為cookie在瀏覽器端，是誰都可以更改的。目前主流的作法是將session轉換為一種token的形式，存於後端的資料庫，這樣後端就有足夠多元的方式可以管理使用者行為，例如限制token的存在時間、限制token的存在數量等。在express的框架下，要使用session token可以透過express-session這個套件來做到，但資料庫必須實做相關的API，這不符合快速架站的宗旨，因此本站只使用cookie-session。
> npm install cookie-session --save

在app.js中加入：
```js
var cookieParser = require('cookie-parser');
app.use(cookieSession({
  name: 'session',
  keys: ['key1', 'key2'],
    
  // Cookie Options
  maxAge: 24 * 60 * 60 * 1000 // 24 hours                              
}));
```

# Index
將cookie-session設定完成後，開始編輯index.js進行routing的設定。本站需要首頁、使用者登入和註冊、登出、記帳首頁、紀錄支出、紀錄收入、列出紀錄。

1. get('/)
2. post('/')
3. get('/logout')
4. get('/accounting')
5. post('/accounting')
6. post('/incoming')
7. get('/list/:type/:range*?')