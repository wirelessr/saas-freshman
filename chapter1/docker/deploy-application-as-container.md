# Deploying Your First Docker Container
## Running A Container
1. 建一個redis容器
2. Find existed image at [DockerHub](registry.hub.docker.com), `docker search <name>`
3. 載入一個image, `docker run <options> <image-name>`
: 背景執行`-d` 
: 預設會執行`latest`，若要指定版本`<image-name>:<version>`

```bash
docker search redis
docker run -d redis:3.2
```

## Finding Running Containers
1. 查看正在跑的container, `docker ps`
2. 看詳細狀態, `docker inspect <friendly-name|container-id>`
3. 看log, `docker logs <friendly-name|container-id>`

## Accessing Redis
- 為container取一個名字
- 開放port mapping
- `-p <host-port>:<container-port>`

```bash
docker run -d --name redisHostPort -p 6379:6379 redis:latest
```

上述的port mapping是1-1 mapping，當container變多了，就會port conflict。最好的辦法是讓docker自己決定要把map甚麼port，因此只需要告訴container打開6379，讓docker去管理。

```bash
docker run -d --name redisDynamic -p 6379 redis:latest
docker port redisDynamic 6379
```

透過`docker port`可以知道該container被mapping到哪個port
> 0.0.0.0:32768

也可以透過`docker ps`看更進步的訊息，例如ip mapping。

## Persisting Data
除了port mapping之外，實體檔案/目錄的mapping也是一個很重要的功能，在docker可以透過`-v <host-dir>:<container-dir>`來達成。

```bash
docker run -d --name redisMapped -v /opt/docker/data/redis:/data redis
```

除此之外，還可以賦予mapping的目錄存取權限，例如：redis使用的data其他container只能讀取`ro`。若是進行刪除，就會回報`Read-only file system`。

```bash
docker run -d -v /opt/docker/data/redis:/data:ro ubuntu rm -rf /data
```

## Running A Container In The Foreground
如果不加`-d`就可以使container在前景執行，若是需要跟container有交互則需要`-it`，例如使用bash的時候。

```bash
docker run ubuntu ps
docker run -it ubuntu bash
```

container一次只會運行一個application，換句話說，前景執行就是執行完一次，這個container就結束了。`bash`模式也是同樣的，當用`exit`退出時container就結束了。

# Deploy Application as Container
在本章將要透過`nginx`來架設一個靜態網頁伺服器。
## Create Dockerfile
基本上，透過`docker run`我們已經有一個可以執行的image，但實際上這個image要能夠正常工作，可能會需要些額外的環境設置，這時候寫一個`Dockerfile`來告訴image該如何部屬執行環境。

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

- `FROM`定義了要使用甚麼image
- `COPY`是container將檔案複製到指定位置
- `EXPOSE <port>`, e.g., `EXPOSE 80 443` or `EXPOSE 7000-8000`
: `EXPOSE`只有描述的用途，事實上要真的做到開port還是得要`-p`
- `RUN <command>`
- `CMD ["cmd", "-a", "arga value", "-b", "argb-value"]`, 用array給要執行的command和參數
- `ENTRYPOINT`
> RUN、CMD、ENTRYPOINT這三個我還沒辦法分清楚它們的使用場合
- `WORKDIR <dir>`, 設定container的工作目錄
- `ADD <dir> <c-dir>`, 將某個目錄映射到container內的指定位置

當這些關鍵字前面加了一個`ONBUILD`的修飾詞，那這些關鍵字就可以被之後引用這個base image的`Dockerfile`直接套用，例如：

```dockerfile
FROM apache:1-onbuild
EXPOSE 8080
```

假設`apache:1-onbuild`這個base image已經把環境都設定好了，只差port還沒決定要開在哪，那麼引用的`Dockerfile`只需要指定port就好，其他事情都由base image包辦。

## Build Docker Image
利用`docker build`引用`Dockerfile`，指令格式`docker build -t <friendly-name>:<version-tag> <build-directory>`，可以透過`-t`給一個名字和打一個tag以標註版本。

```bash
docker build -t webserver-image:v1 .
```

這樣就將一個base image加工成我們自己的，可以根據這個加工的image來長container。另外，可以透過`docker images`查看在host OS上建立了甚麼image。

## Run
當image已經建立完成，就可以透過`docker run`把container帶起來。

```bash
docker run -d -p 80:80 webserver-image:v1
```

這麼一來就有一個web server在背景執行並且把80 port給開出來。
若是要設定環境變數，可以透過`-e ENV=env`，如此container帶起來時，就會有指定的環境變數。

## Ignore file
若是在使用`ADD`時有一些想要跳過的檔案，可以建立一個`.dockerignore`，然後將想要排除的檔案放進去，類似`.gitignore`。例如：`echo privatekey.pem >> .dockerignore`。但有些場合是真的需要那把private key，這時候建議透過`RUN`把用完的檔案給刪掉。

## Execute commands
在之前介紹了帶起container時可以選擇要前景執行並且執行command，或者進入bash交互模式來下指令。當container在背景執行時，還是可以在host OS對container操作，這時要透過`docker exec`，或者pipe模式的`docker exec -i`。

```bash
docker run -d --name r2 redis
docker exec r2 ps aux
echo 'ps aux' | docker exec -i r2 bash
```

## Logging
Docker提供了內建的機制能夠將container內的`stdout`和`stderr`倒入檔案，讓使用者可以在host OS透過`docker logs`查看；或者將輸出導入syslog；也可以直接將所有的輸出關閉。

```bash
docker run -d --name r2 redis
docker logs r2

docker run -d --name redis-syslog --log-driver=syslog redis
docker run -d --name redis-none --log-driver=none redis
```

要查看container使用哪個log driver則可以透過`docker inspect`查看，有一個簡單的參數可以直接把log driver調出來。

```bash
docker inspect --format '{{ .HostConfig.LogConfig }}' r2
```

## Ensuring Uptime
當container因為某些原因掛掉了，這個container就會離開執行狀態，但依然存在，透過`docker ps -a`還是可以察覺它的存在。Docker可以設定restart policy，`--restart`，當container內的application結束，會被自動重帶。想要無論甚麼原因都重帶可以設為`always`，只有在回傳值為非0結束時才重帶則是`on-failure`。這些策略後面還可以加入重帶次數，例如`on-failure:3`，就是最多重帶3次。

```bash
docker run -d --name restart-always --restart=always scrapbook/docker-restart-example
docker run -d --name restart-always --restart=on-failure scrapbook/docker-restart-example
```

## Docker Stats
Docker提供一套簡易的工具，類似top，稱為`docker stats`，可以即時監控每個執行的container狀態，例如CPU/memory用量。

```bash
docker stats r2
docker ps -q | xargs docker stats
```

透過`docker ps -q`列出所有執行中的container的ID，然後丟入`docker stats`中，因為`docker stats`讀取的是`stdin`所以必須使用`xargs`。

# Data Containers
每一個container建立時都是從全新的base image做出來，若是container本身想要保存自身的狀態，那就必須透過data container。據我的理解，data container可以說是一個當作volume的存在，它不實際執行，所以在`docker ps`看不到它的存在，但其餘執行的container可以儲存資料在它內部，或者利用它來當作載具。

## Create Container
要產生一個data container要透過`-v <dir>`來指定開在container內部的路徑。若是要將現有的檔案放入，可以透過`docker cp`，就可以將host OS的檔案放入container內部。

```bash
docker create -v /config --name dataContainer busybox
docker cp config.conf dataContainer:/config/
```

## Mount Volumes From
當data container已經準備好，就可以將它當作volume給其他的container掛載：

```bash
docker run --volumes-from dataContainer ubuntu ls /config
```

有掛載volume的container就可以直接看到原先就存在內部的檔案，當然也可以將新的檔案放入，新放進去的檔案不會因為container結束執行就消失，下次掛載時依然可以看到。

## Export / Import Containers
之所以要用data container來保存檔案而不是採用映射本地目錄的原因在於，若是將檔案放進container內部，那麼就可以用`docker export`的方式將container匯出，並且搬移到另外一台機器上使用，這樣container的狀態不會被host OS綁住，而是可以被移轉。另外一台機器透過`docker import`的方式就可以導入。

```bash
docker export dataContainer > dataContainer.tar
docker import dataContainer.tar
```

導入的container具有和匯出的container一樣的狀態：

```bash
$ docker import dataContainer.tar
sha256:eca83b8f5f5c05c0e69b6810e0fbb7e9119f5b41860d0f4d2a306d311efed0d0
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              eca83b8f5f5c        14 seconds ago      1.13MB
busybox             latest              6ad733544a63        2 months ago        1.13MB
```
