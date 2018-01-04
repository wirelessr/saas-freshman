# Docker Metadata & Labels
當管理大量的container或image時，依賴名字是一個很沒效率的做法，所以docker提供label的機制來為container和image標註，這樣在使用`inspect`時可以根據label進行快篩。

## Docker Containers
要標註一個container很容易，只要在執行時加入`-l <key>=<value>`即可，若是要複用同樣的label也可以透過指定檔案的方式，`--label-file=<filename>`，將不斷使用的label存在一個檔案，就可以容易的被許多container使用。

```bash
docker run -l user=12345 -d redis
echo 'user=123461' >> labels && echo 'role=cache' >> labels
docker run --label-file=labels -d redis
```

## Docker Images
若是要標註image，那要將`LABEL`這個keyword放入`Dockerfile`內。

```dockerfile
FROM ubuntu
LABEL user=1234
LABEL group=admin
```

## Inspect
要快速查找label則是使用`docker inspect`並且帶入參數，container使用的參數是`Config`而image使用的是`ContainerConfig`

```bash
docker inspect -f "{{json .Config.Labels }}" xxx
docker inspect -f "{{json .ContainerConfig.Labels }}" xxx-image
```

## Query By Label
使用label的目的就是在查看狀態時能快速分群，因此無論是`docker ps`或`docker images`都有一個參數`--filter label=<key>=<value>`。

```bash
docker ps --filter "label=user=12345"
docker images --filter "label=group=admin"
```

# Orchestration using Docker Compose
當要管理多個container時光是使用label是不夠的，label充其量只是提供一個方便查詢方便分群的管道，若要做到auto configure，還是必須要有額外的工具幫忙。就像是使用VM的時候，若是要管理大量VM或使VM可程式化，通常我們會借助`vagrant`，透過編寫`Vagrantfile`將要使用的VM設定放入，例如要用哪個base image、網路設定是bridge/NAT/internal、port forwarding怎麼安排，這些都可以透過`vagrant`自動化的進行。欸，等下，這些似曾相似的東西不是docker也有嗎？是的，所以docker也有一套自己的管理工具，稱為`docker-compose`。
[Docker Compose](https://docs.docker.com/compose/compose-file/)
與`Vagrantfile`的自訂格式不同，`docker-compose`透過一個公定的格式`YAML`來做container管理。

## Defining Settings
下面是一個範例，最頂層的設定是container的名字，若是選擇`build`就是透過`Dockerfile`來建立container；若是image則是透過DockerHub。下面設定`links`, `ports`, `volumes`其實就是之前介紹過的功能。

```yaml
web:
  build: .

  links:
    - redis

  ports:
    - "3000"
    - "8000"

redis:
  image: redis:alpine
  volumes:
    - /var/redis/data:/data
```

## Docker Up
如同`vagrant`一般，`docker-compose`也是透過up將container帶起來，和`docker run` 一樣，也可以用`-d`做背景執行。其他還有常用的`stop`和`rm`也都可以一次套用在所有的container上。

```bash
docker-compose up -d
docker-compose stop
docker-compose rm

docker-compose ps
docker-compose logs
```

> 若是要對單獨的container操作，只需要在動作後面加上名字即可。
> docker-compose up web -d

## Docker Scale
當今天一個container不敷使用，可以透過`docker-compose scale`來改變相同設定的container數量，例如：

```bash
docker-compose scale web=3
```

若是現有的container不夠，就會自動增長；設定的個數小於現有的個數，則會回收掉多餘的container。