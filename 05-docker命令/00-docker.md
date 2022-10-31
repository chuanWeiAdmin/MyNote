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
	"registry-mirrors": [
		"https://knt8jedy.mirror.aliyuncs.com",
		"https://mirror.ccs.tencentyun.com",
		"http://hub-mirror.c.163.com",
		"https://docker.mirrors.ustc.edu.cn"
	]
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



## 使用普通用户操作docker

- 添加docker用户组：sudo groupadd docker
- #将登陆用户加入到docker用户组中：sudo gpasswd -a $USER docker
- #更新用户组，当前终端生效，新开终端需要重新输入指令：newgrp docker
- sudo chmod a+rw /var/run/docker.sock



## docker 笔记

### 1.导入和导出容器

export 导出容器的内容留作为一个tar归档文件[对应import命令]

import 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]

命令：

- 导出命令：docker export 容器ID > 文件名.tar
- 导入命令：cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号



### 2. 挂载数据卷  --privileged=true

 Docker挂载主机目录访问如果出现cannot open directory .: Permission denied
解决办法：在挂载目录后多加一个--privileged=true参数即可



### 3. cp命令

#### 3.1 从容器里面拷文件到宿主机

 docke cp 容器名：要拷贝的文件在容器里面的路径    要拷贝到宿主机的相应路径 

#### 3.2 从宿主机拷文件到容器里面

 docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径



### 4.查看容器的基本信息

docker inspect 容器ID

### 5.将容器打成镜像

```dockerfile
docker commit -a="作者"   -m="提交的描述信息"    容器ID           要创建的目标镜像名:[标签名]
docker commit -a "nathan" -m "create new img"   997ce151bc11    镜像名:v0
```

### 6.镜像打包为 tar 文件

```
docker save [OPTIONS] IMAGE [IMAGE...]
docker save -o consul:v0.tar consul:v0
```

### 7. 从 tar 文件载入镜像

```
OPTIONS 选项可选
-i 用于指定载入的镜像文件
-q 精简输出信息
docker load [OPTIONS]

示例
docker load -i consul:v0.tar
```

### 8. 镜像重命名

```dockerfile
docker tag [镜像id] [新镜像名称]:[新镜像标签]
```

