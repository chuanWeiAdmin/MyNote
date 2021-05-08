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





#### ***8修改gitlab管理员账号***

1. 切换目录  cd /opt/gitlab/bin
2.  执行 ：sudo gitlab-rails console production 命令 开始初始化密码![修改gitlab管理员密码1](E:\破烂\笔记\MyNote\资源\修改gitlab管理员密码1.png)
3. 在irb(main):001:0> 后面通过 **u=User.where(id:1).first** 来查找与切换账号（User.all 可以查看所有用户）![修改gitlab管理员密码2](E:\破烂\笔记\MyNote\资源\修改gitlab管理员密码2.png)
4. 通过**u.password='12345678'**设置密码为12345678(这里的密码看自己喜欢)：![修改gitlab管理员密码3](E:\破烂\笔记\MyNote\资源\修改gitlab管理员密码3.png)
5. 通过**u.password_confirmation='12345678'** 再次确认密码
6. 通过 **u.save! **  进行保存（切记切记 后面的 !）
7. 如果看到上面截图中的*true* ，恭喜你已经成功了，执行 *exit* 退出当前设置流程即可。
8. 回到gitlab ,可以通过 **root/12345678** 这一超级管理员账号登录了











