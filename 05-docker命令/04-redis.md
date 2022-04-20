# redis



```dockerfile
docker run -d --name redis -p 6379:6379 redis --requirepass "password"
```

- -d表示后台启动 
- --name表示自定义容器名
-  --requirepass表示设置密码