今天我在實作一個功能，想要在每個class function內透過自己的function name做些事情，然後google找了老半天也找不到如何取得自己的名字，只看到一些很沒用的回答，例如：`my_function.__name__`這樣會得到`my_function`。看到這種答案我白眼都要翻到頭頂了，我如果有`my_function`我還需要`__name__`幹嘛？

後來不想找了，直接用decorator硬上。

```py
def get_name(func): 
    def with_name(*args, **kwargs): 
        kwargs['function'] = func.__name__
        return func(*args, **kwargs) 
    return with_name

class foo(object):
    @get_name
    def bar(self, function=''):
        print 'function =', function

```

既然可以`__name__`拿到名字，那我透過decorator塞回來總行吧！但這樣有一個很明顯的缺點，function會多一個沒意義的參數，只是為了取得名字，不僅影響API的美觀，也很容易誤動作。

後來找到sys底下可以直接拿call stack，那我就從call stack裡面去把上一層的名字抓出來就好。

```py
class foo(object):
    def get_name(self):
        return sys._getframe(1).f_code.co_name

    def bar(self):
        name = self.get_name()
        print name

```

`_getframe()`有用過pdb的人一定很熟悉frame，這個`_getframe`就是在拿那個frame，`1`則是要往上拿一層，然後後面就是把名字抓出來。這樣就可以很漂亮的拿到名字。






