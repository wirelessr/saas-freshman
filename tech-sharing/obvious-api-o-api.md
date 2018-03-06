[O API](https://hackernoon.com/o-api-an-alternative-to-rest-apis-e9a2ed53b93c)是一種取代Restful的concept，原作者認為他很obvious，所以直接稱他為O API。到底Restful有甚麼問題，讓許多人都想要把他換掉？主要原因是Restful號稱簡單但卻一點都不單純，一個action他所需要的information被分散在很多地方：

1. HTTP method, POST/GET/PUT/DELETE
2. Query parameters (?abc=123&def=456)
3. Request body

想要新增一本書有可能會是：POST $base/books/1

```json
{
    article: "Hello"
    author: "John"
    date: "2018/03/06"
}
```

> 若是將所有的東西包在request body，不用到處去拿該會有多方便阿，這就是O API的精神。

我贊同這樣的想法，事實上我也一直都是這樣想的，但經過一番討論，這樣的作法在架構變得龐大後會有些難以克服的代價。例如：

1. Authentication, 若是將所有的資訊都包在request body，那前端的負責認證的server就必須要有能夠解析body的know-how，都是文字或json可能還好，若是Google protocol buffer，那還必須安裝相關package來解析serialization後的資料，這不利於decouple也會影響single sign on。
2. 原本從apache或nginx的access log就可以大致看出一些資訊，能夠做流量或者除錯分析，但都包在request body內就必須要由service本身來做log，某種程度來說也增加了effort。

當然這些問題都是可以克服的，但這無法簡單的從Restful咻地變成O API或其他替代方案，不過，現在是micro-service的時代了，也許取代Restful會越來越常發生吧。