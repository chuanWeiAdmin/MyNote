### 1、停止 firewall 服务

```
systemctl stop firewalld 
```

### 2、注销 firewall 服务

```
systemctl mask firewalld
```

### 3、安装 iptables 服务

```
yum install -y iptables 
yum install iptables-services
```

### 4、启动 iptables 服务

```
systemctl start iptables
```

或者

```
service iptables start
```

### 5、设置 iptables 开机自启动

```
systemctl enable iptables
```

### 6、查看 iptables 状态

```
systemctl status iptables
```

或者

```
service iptables status
```

Active是**激活**状态

### 7、查看 iptables 文件

此时可以在 /etc/sysconfig/ 目录下看到 **iptables** 文件

### 8、iptables 服务的停止命令如下：

```
systemctl stop iptables
```

或者

```
service iptables stop
```

### 9、iptables 服务的重启命令如下：

```
systemctl restart iptables
```

或者

```
service iptables restart
```

### 10、保存 iptables 文件修改，命令如下：

```
service iptables save
```

### 11、重载 iptables 文件，命令如下：

```
systemctl reload iptables
```

或者

```
service iptables reload
```

### 12、查看已配置的iptables规则，命令如下：

1. 不带编号

   ```
   iptables -n -L
   ```

2. 带编号

   ```
   iptables -n -L --line-numbers
   ```

   