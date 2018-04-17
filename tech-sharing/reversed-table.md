想像一種情境，資料庫裏面存了各種車輛資料，包括廠牌。有一個query進來，想知道資料庫裏面所屬福斯集團的車有哪些？這時候需要有一張對應表把福斯集團底下所有的廠牌都列出來，例如：福斯、保時捷、奧迪之類的，那麼對應表就會長下面這個樣子：

```py
CARS = {
	'vw': ('vw', 'skoda', 'audi', 'porsche',),
	'toyota': ('toyota', 'lexus', 'hino',),
	'subaru': ('subaru',),
}
```

那麼只需要把對應表列出來的所有的廠牌撈出來即可：

```py
cars = []
for vendor in CARS[query_type]:
	cars += query(vendor)
```

但反過來說，也有可能需要反過來查找的情況，例如我想知道保時捷所屬哪個汽車集團。最直覺的作法是攤開來找：

```py
for k, v in CARS.items():
	if query_vendor in v:
		return k
```

但這樣的在大量的query下，會需要O(n)。這時最直覺的作法是做一個反向表，讓他一樣以查表來找：

```py
VENDORS = {
	'vw': 'vw',
	'skoda': 'vw', 
	'audi': 'vw', 
	'porsche': 'vw',
	'toyota': 'toyota', 
	'lexus': 'toyota', 
	'hino': 'toyota',
	'subaru': 'subaru',
}
```

這樣當更新了CARS，也要同時更新VENDORS，很麻煩，最好能夠自動化解決，所以要透過list comprehension：

```py
VENDORS = dict([(t, k) for k, v in CARS.items() for t in v])
```

無論CARS被怎麼改動，VENDORS都會被自動建立，而且只建立一次。
