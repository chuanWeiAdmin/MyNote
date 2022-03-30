# MySql

## 一.不进行挂载

### 1 运行docker命令

```dockerfile
docker run -p 3306:3306 --name mysqltest -e MYSQL_ROOT_PASSWORD=root -d mysql
```

### 2 进入 mysql 容器

```dockerfile
docker exec -it eea /bin/bash
```

### 3 修改远程访问的权限

```sh
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
```

### 4 刷新命令

```sh
flush privileges;
```

