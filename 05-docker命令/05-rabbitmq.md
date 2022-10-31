## rabbitmq



### 开启镜像

```
docker run -d \
-e RABBITMQ_DEFAULT_USER=user \
-e RABBITMQ_DEFAULT_PASS=123456 \
--name mq \
-p 15672:15672 \
-p 5672:5672 \
rabbitmq
```



### 开启web界面管理插件

```
rabbitmq-plugins enable rabbitmq_management
```

