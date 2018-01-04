# Study Notes
這是我自學docker的過程和筆記，因為不是把整本書啃完，所以可能有些是我自己的理解，或者沒跟上最新版的地方，歡迎指證。

# Docker concept
我的理解是必須有一個docker image跑在guest OS上，由這個`image`產生`container`，也就是container是執行實體`instance`，但其依附在image上。所有當下的鏡像`snapshot`都記錄在image，而container則會繼續演進，必要時，可以將container當下的狀態快取下來成為新的image。