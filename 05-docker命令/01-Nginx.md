# Nginx



## 命令启动

```dockerfile
docker run -d -p 8888:80 --name nginx_test \
--privileged=true \
-v /root/nginx/html:/usr/share/nginx/html \
-v /root/nginx/config/nginx.conf:/etc/nginx/nginx.conf \
-v /root/nginx/config/conf.d:/etc/nginx/conf.d \
-v /root/nginx/logs:/var/log/nginx \
nginx
```





```shell
#启动一个容器
 docker run -d --name nginx nginx
# 查看 容器 获取容器ID 或直接使用名字
 docker container ls
# 在当前目录下创建目录：conf 
 mkdir conf

# 拷贝容器内 Nginx 默认配置文件到本地当前目录下的 conf 目录（$PWD 当前全路径）
docker cp nginx:/etc/nginx/nginx.conf $PWD/conf
docker cp nginx:/etc/nginx/conf.d $PWD/conf

# 停止容器
 docker container stop nginx
# 删除容器
 docker container rm nginx

# 在当前目录下创建目录：html 放静态文件
 mkdir html
```

