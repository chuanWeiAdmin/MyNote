### 1 .开启防火墙

```shell
systemctl start firewalld.service
```

### 2.关闭防火墙

```shell
systemctl stop firewalld.service
```

### 3. 重启防火墙

```shell
service firewalld restart
firewall-cmd --reload（切记，添加防火墙端口之后要记得重启防火墙）
```

### 4.开启指定端口

```shell
firewall-cmd --zone=public --add-port=9955/tcp --permanent
```

**开启4400-4600的upd协议端口**

```shell
firewall-cmd --zone=public --add-port=4400-4600/udp --permanen
```

### 5.关闭指定端口

```shell
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

### 6.查看通过的端口

```shell
firewall-cmd --zone=public --list-ports
# 查询是否开启80端口
firewall-cmd --query-port=80/tcp
```

### 7. 查看本机已经启用的监听端口

```shell
netstat -ant 	#CentOS7以下使用
ss -ant	 		#7及以上使用ss
```

### 8.查看防火墙状态 

```shell
firewall-cmd --state
```

