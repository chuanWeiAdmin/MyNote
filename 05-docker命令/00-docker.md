# docker 开启远程

## 1.编辑文件

```sh
vim /lib/systemd/system/docker.service
```

```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

## 2 重新加载配置

```sh
systemctl daemon-reload
```

## 3 重启docker

```sh
service docker restart
```

## 4 测试

```sh
curl http://localhost:2375/version
curl http://localhost:2375/info
```





# docker容器内安装ps命令



```sh
apt-get update
apt-get install procps
```

