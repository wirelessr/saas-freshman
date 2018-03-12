sscanf在C語言中是一個解析字串的利器，但在python中並沒有完全相等的作法，有人做了一個sscanf的python module，但要額外安裝package並不是我的風格，我喜歡利用語言native能做到的來解決事情。因此必須使用regular expression。

但是python的regular expression每次都會忘記到底要怎麼用，乾脆打成一篇記錄下來。

```python
import re
s = '/usr/sbin/sendmail - 0 errors, 4 warnings'
m = re.match(r'(\S+) - (\d+) errors, (\d+) warnings', s)
print m.groups()
```

若是要取值只需要`m.groups()[n]`即可抓到token。**Reference 1**記錄了與sscanf等價的regular expression的表示法。

有一個懶人作法能夠適用空白分隔且基本type。

```python
input = '1 3.0 false hello'
strtobool = lambda s: {'true': True, 'false': False}[s]
(a, b, c, d) = [t(s) for t,s in zip((int,float,strtobool,str),input.split())]
print (a, b, c, d)
```

Reference:
1. [Python Org](https://docs.python.org/2/library/re.html#simulating-scanf)
2. [StackOverflow](https://stackoverflow.com/questions/2175080/sscanf-in-python)