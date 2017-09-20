Celery

Preparation

Erlang

```
# In /etc/yum.repos.d/rabbitmq-erlang.repo
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/20/el/6
gpgcheck=1
gpgkey=https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```

```
yum install erlang
```

SOCAT

```
wget –no-cache http://www.convirture.com/repos/definitions/rhel/6.x/convirt.repo -O /etc/yum.repos.d/convirt.repo
yum makecache
yum install socat
```

RabbitMQ

```
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
yum install rabbitmq-server
```



