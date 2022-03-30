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

