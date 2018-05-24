我剛好有一個需求，要把一個字串根據固定長度換行。之前都只知道split去切特殊字元，要根據長度切字串只會比較基本的做法：

```py
s = s[:n] + '\n' + s[n:]
```

若是要多次換行，就必須跑個迴圈：

```py
def split(s, n):
  """
  Split string every nth character

  Parameters
  ----------
  s: string
  n: value of nth
  """
  new_list = []
  for i in range(0, len(s), n):
    new_list.append(s[i:i+n])
  return new_list

s = '\n'.join(split(s, n))
```

但我想知道，有沒有比較python的作法，因為python的built-in博大精深，一定有類似的API可以做到同樣的事，不然python這麼肥就只是虛胖。果然，的確有built-in API：

```py
from textwrap import wrap
s = '\n'.join(wrap(s, n))
```

這就是最python的做法，簡單、漂亮。



[[1] StackOverflow](https://stackoverflow.com/questions/9475241/split-string-every-nth-character)

