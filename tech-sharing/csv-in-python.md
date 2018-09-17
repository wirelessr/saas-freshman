CSV算是很常使用的格式，但真要用的時候還是有些卡卡的，這邊稍微紀錄一下兩個常見的重點：

1. Read csv from string
2. Read csv as dict



要讀一個csv format的字串而不是檔案需要透過`StringIO`，也就是將string當成檔案。在python2和python3稍微有點差別：

```python
try:
    # for Python 2.x
    from StringIO import StringIO
except ImportError:
    # for Python 3.x
    from io import StringIO
    
import csv

scsv = """text,with,Polish,non-Latin,lettes
1,2,3,4,5,6
a,b,c,d,e,f
gęś,zółty,wąż,idzie,wąską,dróżką,
"""

f = StringIO(scsv)
reader = csv.reader(f, delimiter=',')
for row in reader:
    print('\t'.join(row))
```

另外，csv還是當作dict比較容易使用，這時候可以用`DictReader`：

```python
f = StringIO(scsv)
reader = csv.reader(f, delimiter=',')
for row in reader:
    print(row['text'], row['with'], row['lettes'])
```

### Reference
[StringIO](https://stackoverflow.com/questions/3305926/python-csv-string-to-array)
[DictReader](https://blog.gtwang.org/programming/python-csv-file-reading-and-writing-tutorial/)

