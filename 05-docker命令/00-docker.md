## docker 开启远程

### 1.编辑文件

```sh
vim /lib/systemd/system/docker.service
```

```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

### 2 重新加载配置

```sh
systemctl daemon-reload
```

### 3 重启docker

```sh
service docker restart
```

### 4 测试

```sh
curl http://localhost:2375/version
curl http://localhost:2375/info
```





## docker容器内安装ps命令



```sh
apt-get update
apt-get install procps
```







## 安装Docker

### 1.卸载旧版本的docker

```
yum remove docker \
	docker-client \
	docker-client-latest \
	docker-common \
	docker-latest \
	docker-latest-logrotate \
	docker-logrotate \
	docker-engine
```

### 2.安装依赖

```
yum install -y yum-utils
```

### 3.配置阿里云镜像安装docker

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 4.安装最新社区版docker

```
yum install docker-ce docker-ce-cli containerd.io
```

### 5.启动docker

```
systemctl start docker
```

#### 6.确认docker是否安装成功，同时确认docker安装版本

```
docker version
```

### 7.运行hello-world

```
docker run hello-world
```

### 8.查看镜像docker images

```
docker images
```

### 删除docker

```
# 第一步
yum remove docker-ce docker-ce-cli containerd.io
# 第二步
rm -rf /var/lib/docker
```





## 配置镜像加速

### 1. 配置命令

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://knt8jedy.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 2.其他镜像

```
https://mirror.ccs.tencentyun.com
http://hub-mirror.c.163.com
https://docker.mirrors.ustc.edu.cn
```

