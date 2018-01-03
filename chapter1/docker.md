# Docker concept
我的理解是必須有一個docker image跑在guest OS上，由這個`image`產生`container`，也就是container是執行實體`instance`，但其依附在image上。所有當下的鏡像`snapshot`都記錄在image，而container則會繼續演進，必要時，可以將container當下的狀態快取下來成為新的image。

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
