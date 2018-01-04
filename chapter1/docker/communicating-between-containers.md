# Communicating Between Containers
container之間是可以彼此互通的，並不是甚麼特別的技術，而是可以透過docker建立的內部網路互相連線，就像VirtualBox或VMware建立的internal networking一樣，docker也會建立自己的內部網路。
在本章我們使用redis來做示範，會產生一個執行redis-server的container和一個當作redis client的container。

## Start Redis
和之前的做法一樣，我們透過redis這個已經在DockerHub上的base image來產生一個名為redis-server的container。

```bash
docker run -d --name redis-server redis
```

此時redis-server已經在**名為redis-server的container**執行了

## Create Link
要對已知的container建立連線可以透過`--link <container-name|id>:<alias>`將對象的環境變數設定到新建的container上。`--link`只做兩件事，設定環境變數和建立DNS，網路是docker早就準備好的。
透過`env`可以看到，container內部會有以剛剛alias開頭的環境變數，例如*XXX_PORT*。

```bash
$ docker run --link redis-server:xxx alpine env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=e197095c4bfb
XXX_PORT=tcp://172.18.0.2:6379
XXX_PORT_6379_TCP=tcp://172.18.0.2:6379
XXX_PORT_6379_TCP_ADDR=172.18.0.2
XXX_PORT_6379_TCP_PORT=6379
XXX_PORT_6379_TCP_PROTO=tcp
XXX_NAME=/kind_noyce/xxx
XXX_ENV_GOSU_VERSION=1.10
XXX_ENV_REDIS_VERSION=4.0.6
XXX_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-4.0.6.tar.gz
XXX_ENV_REDIS_DOWNLOAD_SHA=769b5d69ec237c3e0481a262ff5306ce30db9b5c8ceb14d1023491ca7be5f6fa
HOME=/root
```

除此之外也可以檢查DNS，會看到有以alias命名的record被映射到172.18.0.2，和上方*XXX_PORT*內的IP一致。

```bash
$ docker run --link redis-server:xxx alpine cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.18.0.2      xxx 8e35ddbd7596 redis-server
172.18.0.4      330243875f8d
```

不信的話也可以用ping試試看，網路是通的。

```bash
docker run --link redis-server:xxx alpine ping -c 1 xxx
```

## Connect to Redis CLI
最後我們就可以在container內實際使用redis-cli來和redis-server連線，透過之前介紹的`-it`可以開啟互動模式，就像在host OS上使用redis-cli一樣，只是實際上是在container內執行。

```bash
docker run -it --link redis-server:xxx redis redis-cli -h xxx
```

> docker run -it --link redis-server:redis redis redis-cli -h redis
> 之前在看範例的時候實際command長這樣，一堆redis，看到都昏了，所以我用xxx來替換掉alias，但還有三處：
> redis-server, 名為redis-server的container
> redis, 將DockerHub上的redis當成base image
> redis-cli, 在container內實際執行的command

# Docker Networks
在上一章，透過`--link`的方法將container之間彼此串聯，但`--link`有一個致命的缺陷，若是container彼此之間是網狀網路，那透過`--link`設定彼此連結會很想死，尤其當container數量增加，每個既存的container都要能夠更新環境變數和DNS，這在實務上不太可能做到。
因此在**docker 1.10**之後加入一個新功能，`Embeded DNS`，可以有個內部使用的DNS server，每個container都有自己對應的record，這樣彼此之間透過query DNS的正常流程就可以知道彼此存在。

## Create Network
首先透過`docker network create`產生一個內部網路，接著container用`--net`連上線。在本章沿用之前的redis當作例子。

```bash
docker network create backend-network
docker run -d --name=xxx --net=backend-network redis
```

這次透過`--net`連上的container和`--link`不同，已經沒有環境變數和DNS的設定了，取而代之的是多了一筆DNS server。這筆127.0.0.11就是docker的`Embeded DNS`。

```bash
$ docker run --net=backend-network alpine cat /etc/resolv.conf
nameserver 127.0.0.11
```

至於連上的container則是使用自己的名字做為DNS的record，因此只要連上的container就可以透過名字找到對方。

```bash
docker run --net=backend-network alpine ping -c1 xxx
```

## Connect Two Containers
當然，要連結數個網路也是可行的。另外，除了`--net`的連線方式之外，也可以透過`docker network connect <networking> <name>`，使用動態指定的方式設定已經運行的container。

```bash
docker network create frontend-network
docker network connect frontend-network xxx
```

## Create Aliases
若是同一個container連結了數個網路，它是可以使用alias的，也就是不只是自己的名字能夠被DNS對應，自己的別名也具有同樣的效果。以下的例子示範了同一個container **xxx**在不同的網路下擁有不同的名字。

```bash
docker network create frontend-network2
docker network connect --alias db frontend-network2 xxx
docker run --net=frontend-network alpine ping -c1 xxx
docker run --net=frontend-network2 alpine ping -c1 db
```

## Disconnect Containers
當網路使用完了，可以透過`docker network disconnect`離開現有的網路。要知道現在有哪些網路可以使用`docker network ls`列出已經建立的網路，接著透過`docker network inspect <networking>`查詢哪些container連上這個網路，最後使用`docker network disconnect <networking> <name>`離線。

```bash
docker network ls
docker network inspect frontend-network
docker network disconnect frontend-network xxx
```
