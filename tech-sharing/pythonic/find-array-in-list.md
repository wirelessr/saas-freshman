最近常常在做資料分析，最頭痛的點是numpy的array如果在list中是無法用`in`來判斷的，例如：

```python
from numpy import array
l = [array([1,1]), array([2,2]), array([3,3])]
array([1,1]) in l
```

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()
```

這裡我們知道，因為array的比較，回傳值依然是個array，而不是單純的boolean，所以`in`沒作用。因此我們需要一個能夠設定比較方法的API，其實python已經內建了，`filter`。

```python
len(list(filter(lambda x: numpy.all(x == array([2,2])), l))) > 0
```

這樣就直接知道是`True`還是`False`。

另外filter在python2和python3的回傳結果不太相同，在python2直接就是傳回一個list，不需要自己casting，但python3為了效能，要自己多轉一次才會變回list。

[Reference](https://medium.com/@happymishra66/lambda-map-and-filter-in-python-4935f248593)