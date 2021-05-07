#### 1.配置yum源

vim /etc/yum.repos.d/gitlab-ce.repo



#### 2.复制下面内容

[gitlab-ce]

name=Gitlab CE Repository

baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/

gpgcheck=0

enabled=1



#### 3.更新本地缓存

sudo yum makecache



#### 4.安装GitLab

sudo yum install gitlab-ce



#### 5.更改配置文件

vim  /etc/gitlab/gitlab.rb

将extrenal_url 'http://121.4.184.80:9999' 改为自己的ip



#### 6.修改Nginx的监听端口

vim  /etc/gitlab/gitlab.rb （同样的配置文件）

 nginx['listen_port'] = nil----> nginx['listen_port'] = 9999



#### 7.启动

 gitlab-ctl reconfigure------->重新加载配置

gitlab-ctl start------->启动gitlab

启动gitlab ： gitlab-ctl reconfigure && gitlab-ctl start

查看状态: gitlab-ctl status 









#### 