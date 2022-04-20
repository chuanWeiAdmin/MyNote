# Nginx基础使用

## 一.目录结构

进入Nginx的主目录我们可以看到这些文件夹

```shell
client_body_temp conf fastcgi_temp html logs proxy_temp sbin scgi_temp uwsgi_temp
```

**其中这几个文件夹在刚安装后是没有的，主要用来存放运行过程中的临时文件**

```
client_body_temp fastcgi_temp proxy_temp scgi_temp
```

### 1 conf

用来存放配置文件相关

### 2 html

用来存放静态文件的默认目录 html、css等

### 3 sbin

nginx的主程序

### 4 nginx 启动

```txt
1、验证配置文件
./nginx -tc 配置文件(nginx.conf)
或者
./nginx -t -c  配置文件(nginx.conf)

2、指定配置文件启动
./nginx -c  配置文件(nginx.conf)

3、指定配置文件重启
./nginx -s reload -c  配置文件(nginx.conf)
```



## 二.基本运行原理

![](.\..\99-资源\nginx运行原理.jpg)

## 三.Nginx配置与应用场景

### 1 最小配置

#### worker_processes

worker_processes 1; 默认为1，表示开启一个业务进程

#### worker_connections

worker_connections 1024; 单个业务进程可接受连接数

#### include 

include mime.types; 引入http mime类型

#### default_type 

default_type application/octet-stream; 如果mime类型没匹配上，默认使用二进制流的方式传输。

#### sendfile 

sendfile on; 使用linux的sendfile(socket, file, len) 高效网络传输，也就是数据0拷贝。

- 未开启sendfile

  ![](.\..\99-资源\未开启sendfile.jpg)

- 开启后

  ![](.\..\99-资源\开启后.jpg)



#### keepalive_timeout 

keepalive_timeout    timeout   [header_timeout];

keepalive_timeout 65;

1. 第一个参数：设置keep-alive客户端连接在服务器端保持开启的超时值（默认75s）；值为0会禁用keep-alive客户端连接；
2. 第二个参数：可选、在响应的header域中设置一个值“Keep-Alive: timeout=time”；通常可以不用设置；

**注**：keepalive_timeout默认75s，一般情况下也够用，对于一些请求比较大的内部服务器通讯的场景，适当加大为120s或者300s；

### 2 虚拟主机

**原本一台服务器只能对应一个站点，通过虚拟主机技术可以虚拟化成多个站点同时对外提供服务**

#### 2.1 servername匹配规则

我们需要注意的是servername匹配分先后顺序，写在前面的匹配上就不会继续往下匹配了。

#### 2.2 完整匹配

我们可以在同一servername中匹配多个域名

```nginx
server {
	listen 88;
	server_name vod.mmban.com www1.mmban.com;
	location / {
		root /usr/share/nginx/html/test;
		index index.html;
	}
}
```

#### 2.3 通配符匹配

```nginx
server_name *.mmban.com
```

#### 2.4 通配符结束匹配

```nginx
server_name vod.*;
```

#### 2.5 正则匹配

```nginx
server_name ~^[0-9]+\.mmban\.com$;
```

### 3 反向代理

```nginx
server {
	listen 81;
	server_name localhost;
	location / {
		proxy_pass http://baidu.com;
	}
}
```



## 四.基于反向代理的负载均衡

```nginx
upstream hs {
	server 121.4.184.80:8891;
	server 121.4.184.80:8892;
}

server {
	listen 81;
	server_name localhost;
	location / {
		proxy_pass http://hs/t/;
	}
}
```

### 1 负载均衡策略

#### 1.1 轮询

默认情况下使用轮询方式，逐一转发，这种方式适用于无状态请求。

#### 1.2 weight(权重)

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```nginx
upstream httpd {
	server 127.0.0.1:8050 weight=10 down;
	server 127.0.0.1:8060 weight=1;
	server 127.0.0.1:8060 weight=1 backup;
}
```

- down：表示当前的server暂时不参与负载
- weight：默认为1.weight越大，负载的权重就越大。
- backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。

#### 1.3 ip_hash

根据客户端的ip地址转发同一台服务器，可以保持回话。

#### 1.4 least_conn

最少连接访问

#### 1.5 url_hash

根据用户访问的url定向转发请求

#### 1.6 fair

根据后端服务器响应时间转发请求

### 2 动静分离

#### 2.1 配置反向代理

```nginx
location / {
	proxy_pass http://127.0.0.1:8080;
    # 我不大理解下面的两行一定要写吗？？？？？？？？？？？？？？？？？
	root html;
	index index.html index.htm;
}
```



#### 2.2 增加一个location

```nginx
server {
	listen 81;
	server_name localhost;
	
    location / {
		proxy_pass http://127.0.0.1:8080;
		root html;
		index index.html index.htm;
	}

    #当反向代理的数据遇到静态文件后到下面的路径寻找 
	location /css {
		root /usr/local/nginx/static;
		index index.html index.htm;
	}
	location /images {
		root /usr/local/nginx/static;
		index index.html index.htm;
	}
	location /js {
		root /usr/local/nginx/static;
		index index.html index.htm;
	}
}
```



#### 2.3 location 匹配方法

=    精准匹配，不是以指定模式开头

~    正则匹配，区分大小写

~*   正则匹配，不区分大小写

^~   非正则匹配，匹配以指定模式开头的location



注：上节静态文件匹配可以写成如下方式

```nginx
location ~*/(css|img|js) {
	root /usr/local/nginx/static;
	index index.html index.htm;
}
```



#### 2.4 location匹配顺序

- 多个正则location直接按书写顺序匹配，成功后就不会继续往后面匹配
- 普通（非正则）location会一直往下，直到找到匹配度最高的（最大前缀匹配）
- 当普通location与正则location同时存在，如果正则匹配成功,则不会再执行普通匹配
- 所有类型location存在时，“=”匹配 > “^~”匹配 > 正则匹配 > 普通（最大前缀匹配）

#### 2.5 alias与root

**root用来设置根目录，而alias在接受请求的时候在路径上不会加上location。**

```nginx
location /css {
	alias /usr/local/nginx/static/;
	index index.html index.htm;
}
location /js {
	root /usr/local/nginx/static/;
	index index.html index.htm;
}
```

**我的理解的**

- 如上使用 root 配置的最后路径 ：/usr/local/nginx/static/js;
- 如上使用 alias 配置的最后路径： /usr/local/nginx/static;
- 使用 alias 想当于给路径起了一个别名，而 root 是拼接上的



**注：**

1. alias指定的目录是准确的，即location匹配访问的path目录下的文件直接是在alias目录下查找的； 
2. root指定的目录是location匹配访问的path目录的上一级目录,这个path目录一定要是真实存在root指定目录下的；
3. 使用alias标签的目录块中不能使用rewrite的break（具体原因不明）；另外，**alias指定的目录后面必须要加上"/"符号！！** 
4. alias虚拟目录配置中，location匹配的path目录如果后面不带"/"，那么访问的url地址中这个path目录后面加不加"/"不影响访问，访问时它会自动加上"/"； 但是如果location匹配的path目录后面加上"/"，那么访问的url地址中这个path目录必须要加上"/"，访问时它不会自动加上"/"。如果不加上"/"，访问就会失败！ 
5. root目录配置中，location匹配的path目录后面带不带"/"，都不会影响访问。

### 3 UrlRewrite

**rewrite语法格式及参数语法:**

```nginx
#rewrite 是实现URL重写的关键指令，根据regex (正则表达式)部分内容，重定向到replacement，结尾是flag标记。

rewrite   <regex>   <replacement>   [flag];
关键字       正则      替代内容       flag标记
```

#### 3.1 关键字

就是关键字

#### 3.2 正则

- perl兼容正则表达式语句进行规则匹配
-   ^ .*$      ^开始   $结束

#### 3.3 替代内容

将正则匹配的内容替换成想要替代的内容

#### 3.4 flag标记

- **last** 匹配完成后，继续向下匹配新的location URI规则
- **break** 匹配完成即终止，不再匹配后面的任何规则
- **redirect** 返回302临时重定向，浏览器地址会显示跳转后的URL地址
- **permanent** 返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

#### 3.5 rewrite参数的标签段位置：

- server
- location
- if

#### 3.6 实例

```nginx
rewrite ^/([0-9]+).html$ /index.jsp?pageNum=$1 break;
# 在 () 内的规则表示一个参数，使用  $xx使用参数
# $1 表示使用第一个括号内的参数
```



### 4 防盗链配置

```nginx
valid_referers none | blocked | server_names | strings ....;
```

- none， 检测 Referer 头域不存在的情况。
- blocked，检测 Referer 头域的值被防火墙或者代理服务器删除或伪装的情况。这种情况该头域的值不以 “http://” 或 “https://” 开头。
- server_names ，设置一个或多个 URL ，检测 Referer 头域的值是否是这些 URL 中的某一个。

#### **在需要防盗链的location中配置**

```nginx
server {
	listen 81;
	server_name localhost;
    
	location /css {
        # 这里判断 请求头中的 referers 是不是从 配置的地址从过来的
		valid_referers 192.168.44.101;
		if ($invalid_referer) {
			return 403;
		}
		root /usr/local/nginx/static;
		index index.html index.htm;
	}
}
```



#### curl测试

**curl 是一个软件 如果没有的话就安装一下 **

```sh
yum install curl
```

```sh
curl -I http://192.168.44.101/img/logo.png
```

**剩下的用法直接百度**

### 5 应用服务器防火墙配置

#### 5.1 开启防火墙

```shell
systemctl start firewalld
```

#### 5.2 重启防火墙

```sh
systemctl restart firewalld
```

#### 5.3 重载规则

```sh
firewall-cmd --reload
```

-  **firewall-cmd   是一个防火墙工具**

#### 5.4 查看已配置规则

```sh
firewall-cmd --list-all
```

#### 5.5 指定端口和ip访问

```sh
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.44.101" port protocol="tcp" port="8080" accept"
```

#### 5.6 移除规则

```sh
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.44.101" port port="8080" protocol="tcp" accept"
```

## 五.高可用配置

## 六.Https证书配置