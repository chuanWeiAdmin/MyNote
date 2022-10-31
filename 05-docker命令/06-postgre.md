## postgre+postgis

### 拉取镜像

```dockerfile
docker pull mdillon/postgis
```

### 部署

```dockerfile
docker run --name postgresql -d \
-p 65432:5432 \
-v /home/pgdata:/var/lib/postgresql/data \
-e POSTGRES_PASSWORD=1234 \
mdillon/postgis
```



### 导出数据

```
pg_dump -U postgres -d postgres > ssssss.dump;
```

### 导入数据

```
psql -h localhost -p 5432 -U postgres -d postgres -f ssssss.dump
```

