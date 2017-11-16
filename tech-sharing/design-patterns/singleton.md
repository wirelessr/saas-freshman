# Singleton

一般介紹design pattern的書籍一定會提到singleton，這概念很重要，但在Ｃ語言其實沒多少人在意，因為Ｃ語言中只要在source code file獨立宣告一個static的變數，基本上已經保證他是singleton了，但在物件的世界裡，每個變數都是物件，都能夠被獨立引用，因此需要有一個機制來確保宣告並使用的物件是唯一，大家用的都是一樣的。

當然，這也牽扯到thread-safe的問題，如果在多執行緒的環境，那麼即使物件唯一，大家在使用上依然存在synchronize，這無論是在Ｃ語言或python中都是無可避免的問題。

Singleton的做法有很多種，可以用繼承可以用meta class還有一些變化，但我看了坊間的書籍，例如PACKT的Learning Python Design Patterns，都沒有提到使用decorator的做法，decorator是python一個很重要的特性，這邊就來介紹這個做法。

```py
import threading
from functools import wraps
 
global_singleton_lock = threading.Lock()
 
def singleton(cls):
    instances = {}
    @wraps(cls)
    def getinstance(*args, **kw):
        global global_singleton_lock
        with global_singleton_lock:
          if cls not in instances:
              instances[cls] = cls(*args, **kw)
          return instances[cls]
    return getinstance
 
@singleton
class MyClass:
  def __init__(self):
    print 'Hello world'
```



現在MyClass就是一個singleton class，無論宣告幾次，使用的都會是同一個instance。

```py
a = MyClass()
b = MyClass()
print a, b
```

而singleton這個decorator可以管理不只是MyClass，要掛別的class也是可以的，並且保證thread-safe，因為在存取instances已經透過lock保護了。



但這效率還可以再提升，若是大量同樣的class宣告，這樣會進入不必要的lock。

```py
import threading
from functools import wraps
 
global_singleton_lock = threading.Lock()
 
def singleton(cls):
    instances = {}
    @wraps(cls)
    def getinstance(*args, **kw):
        global global_singleton_lock
        if cls in instances: # improve performance
          return instances[cls]
        with global_singleton_lock:
          if cls not in instances:
              instances[cls] = cls(*args, **kw)
          return instances[cls]
    return getinstance
 
@singleton
class MyClass:
  def __init__(self):
    print 'Hello world'
```

在進lock前就先檢查一次，若有找到就直接回傳了，可以避免大量同樣的class在消耗lock的cost。

