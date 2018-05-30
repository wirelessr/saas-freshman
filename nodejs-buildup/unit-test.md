# 單元測試
整個網站的model就只有兩個：users和records，也就是說，此時已經實做完整個資料存取的邏輯層，這時需要寫幾個test case來驗證功能是否正確。目前在Node.js上使用的主流單元測試框架是`mocha`，因此本站使用mocha寫幾個單元測試的案例，並且搭配一個輕便的assert framework *should*。
> npm install should --save-dev

只在開發環境需要should這個套件，因此使用`--save-dev`這個參數。接著開一個test資料夾，新增test_models.js。

```js
describe('test users', function() {
  var uid;

  it('test create', function(done) {
      User.create({username: 'unittest', password: 'unittest'}, function(err, user) {
          assert.exist(user);
          assert.equal(user.username, 'unittest');
          uid = user.uid;
          done();
      });
  });

  it('test auth', function(done) {
      User.authenticate('unittest', 'unittest', function(err, user) {
          assert.exist(user);
          assert.equal(user.username, 'unittest');
          assert.equal(user.uid, uid);
          done();
      });
  });

  it('test del', function(done) {
      User.delById(uid, function() {
          User.findById(uid, function(err, user) {
              assert.not.exist(user);
              assert.equal(err.status, 404);
              done();
          });
      });
  });
});
```

這邊單元測試只做了一些簡單的case，並沒有將每一個corner case都完整測試。寫單元測試的目的是為了反覆驗證功能是否正常，尤其是這個data model牽扯到google試算表，需要有機制能夠確保相容於之後的試算表版本；或者當要enhance data model時，能夠有一個安心機制來確保不會改壞原本的功能。

有一點很重要，若是直接`mocha`，八成會噴error，因為整個資料庫是從google試算表取得，受限於網路頻寬，因此要延長mocha的timeout：
> mocha --timeout 15000

