# Mock

在python中要做單元測試很簡單，因為已經有內建的好用package: unittest了，也有很方便的工具nosetests，無論是基本的functional testing或者要看覆蓋率，鼻子都能夠做到。

在做單元測試的時候常會碰到一個問題，就是某一個function很難測，這通常是在開API的時候沒設計好，包山包海，這時候就需要有機制能夠讓測試繼續進行下去，在python中提供的機制就是mock \(在python3中mock已經被整合進了unittest中\)，mock能夠複寫掉一個function並指定回傳結果。

例如：

```py
def foo(x):
    # balabala
    return something

def bar(x):
    sth = foo(x)
    # balabala
    return result
```

像這樣的API，bar依賴著foo，正常來說測試應該要兩個都測：

```py
import unittest
from example import foo, bar

class ExampleTest(unittest.TestCase):
    def test_foo(self):
        self.assertEqual(foo(10), 100)
        
    def test_bar(self):
        self.assertEqual(bar(50), 50000)        
```

先確認foo運作正常，接著再來測試bar，但有的時候，foo非常難測 \(不是自己寫的或者裡面行為很複雜\)，這時候使用者只想確保自己的bar是正確的，foo只要給定該給的輸出就好，那麼mock就登場了。

```py
import unittest
import mock
# if python3, from unittest import mock
from example import foo, bar

class ExampleTest(unittest.TestCase):
    @mock.patch('example.foo')
    def test_bar(self, mock_foo):
        mock_foo.return_value = 100
        self.assertEqual(bar(50), 50000) 
```

透過mock.patch就可以將example中的foo轉成mock\_foo這個物件，然後直接給定return\_value即可。

問題來了，若是要測試的API跟時間有關，怎麼辦？原理是一樣的，把時間的相關API都mock掉，變成自己來指定時間，即使是python原生提供的datetime或者calendar都可以透過這種方式來產生想要的時間。

```py
from datetime import datetime
from calendar import timegm
import calendar

def get_now():
    now_dt = datetime.utcnow()
    now_ut = timegm(now_dt.timetuple())
    return now_ut

def get_now_ver2():
    now_dt = datetime.utcnow()
    now_ut = calendar.timegm(now_dt.timetuple())
    return now_ut
```

要取得現在的timestamp，這是很常見的做法，或者直接用time.time\(\)取int，但通常為了一致會取utcnow。這邊提供兩種版本，差別只在於import的方式不一樣。兩個都可以mock，只是mock的對象不同。

```py
import unittest
import mock
from example import get_now, get_now_ver2

class ExampleTest(unittest.TestCase):
    @mock.patch('example.timegm')
    def test_get_now(self, mock_foo):
        mock_foo.return_value = 100
        now = get_now()
        print 'now', now
        self.assertEqual(now, 100)

    @mock.patch('example.calendar')
    def test_get_now(self, mock_foo):
        mock_foo.timegm.return_value=200
        now = get_now_ver2()
        print 'now2', now
        self.assertEqual(now, 200)
```

即使是原生套件，依然可以透過mock覆蓋，有了這好東西，希望大家不要害怕寫測試，往CI/CD之路邁進吧。

