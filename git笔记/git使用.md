## Git环境配置

#### 常用的Linux命令

1）、cd : 改变目录。

2）、cd . . 回退到上一个目录，直接cd进入默认目录

3）、pwd : 显示当前所在的目录路径。

4）、ls(ll):  都是列出当前目录中的所有文件，只不过ll(两个ll)列出的内容更为详细。

5）、touch : 新建一个文件 如 touch index.js 就会在当前目录下新建一个index.js文件。

6）、rm:  删除一个文件, rm index.js 就会把index.js文件删除。

7）、mkdir:  新建一个目录,就是新建一个文件夹。

8）、rm -r :  删除一个文件夹, rm -r src 删除src目录

```
rm -rf / 切勿在Linux中尝试！删除电脑中全部文件！
```

9）、mv 移动文件, mv index.html src index.html 是我们要移动的文件, src 是目标文件夹,当然, 这样写,必须保证文件和目标文件夹在同一目录下。

10）、reset 重新初始化终端/清屏。

11）、clear 清屏。

12）、history 查看命令历史。

13）、help 帮助。

14）、exit 退出。

15）、#表示注释



#### Git配置

所有的配置文件，其实都保存在本地！

1. ###### 查看配置

   ```git
   git config -l
   ```

2. ###### 查看系统级别配置

   ```git
   #查看系统config
   git config --system --list
   ```

3. ###### 查看当前用户的配置

   ```git
   #查看当前用户（global）配置
   git config --global  --list
   ```

4. git的配置文件

   - Git\etc\gitconfig  ：Git 安装目录下的 gitconfig   --system 系统级
   - C:\Users\Administrator\ .gitconfig   只适用于当前登录用户的配置  --global 全局
   - *注：也可以通过修改文件直接进行配置*



#### 设置用户名与邮箱（用户标识，必要）

**使用git之前要先设置用户名称和e-mail地址**

```git
git config --global user.name "kuangshen"  #名称
git config --global user.email 24736743@qq.com   #邮箱
```



## Git基本理论（重要）

#### 三个区域

- Workspace：**工作区**，就是你平时存放项目代码的地方
- Index / Stage：**暂存区**，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- Repository：**仓库区（或本地仓库）**，就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
- Remote：**远程仓库**，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

![git三个工作区](..\资源\git三个工作区.PNG)

#### 工作流程

git的工作流程一般是这样的：

1. 在工作目录中添加、修改文件；
2. 将需要进行版本管理的文件放入暂存区域；
3. 将暂存区域的文件提交到git仓库。

**注：git管理的文件有三种状态：*已修改（modified）*,*已暂存（staged）*,*已提交(committed)***



#### 常用语句   

##### 本地操作 

| **git代码**                      | **含义**                                        |
| :------------------------------- | ----------------------------------------------- |
| git  init                        | 在当前目录新建一个Git代码库(初始化一个仓库)     |
| git  add  <filr>                 | 将文件添加到暂存区                              |
| git commit -m "提交信息"         | 将文件提交到本地仓库                            |
| git status                       | 查看仓库当前的状态显示有变更的文件              |
| git diff                         | 比较文件的不同，（有一些非文本文件对比不了）    |
| git reset  --hard  版本号        | 回退版本                                        |
|                                  |                                                 |
| git restore                      | 撤销更改                                        |
| 撤销更改的两种形式：             |                                                 |
| 1.还未提交暂存区，返回到工作区   | git restore 文件/路径                           |
| 2.已经提交暂存区的数据返回工作区 | git restore  --staged  文件    （从暂存区撤回） |
|                                  | git  restore  文件                              |
|                                  |                                                 |
| git log                          | 查看提交log                                     |
| git config --list                | 以列表的形式查看配置                            |
|                                  |                                                 |
|                                  |                                                 |
|                                  |                                                 |
|                                  |                                                 |
|                                  |                                                 |



##### 远程仓库操作

| **git代码**                              | **含义**                      |
| ---------------------------------------- | ----------------------------- |
| ssh-keygen -t rsa -C "1696255001@qq.com" | 生成ssh秘钥，添加到远程仓库中 |
| git remote add origin 远程仓库的ssh      | 关联远程仓库                  |
|                                          |                               |
|                                          |                               |
|                                          |                               |



##### 分支操作

| git代码                                 | 含义                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| git   switch  -c  "分支名"              | 创建分支并切换分支                                           |
| git   switch   "分支名"                 | 切换分支                                                     |
| git   branch                            | 查看分支   *是当前分支                                       |
| git   branch   -d     "分支名"          | 删除分支                                                     |
| git   merge   --no-ff    "分支名"       | 合并分支（将分支拉取到当前分支）                             |
| git  merge  --no-ff -m" 消息"  "分支名" | 合并分支，并添加消息                                         |
| git    tag   v1.0                       | 设置版本号                                                   |
| git  lg                                 | 查看日志（设置完别名）                                       |
|                                         |                                                              |
| git stash                               | 出现bug要更改但是不想提交的时候通过这个将分支***将当前工作现场“储藏”起来*** |
| git stash list                          | 查看储藏起来的工作区                                         |
| git stash apply                         | 恢复工作区但不删除储藏的内容                                 |
| git stash drop                          | 删除储藏起来的内容                                           |
| git stash pop                           | 恢复并删除储藏起来的内容                                     |
| git stash apply stash@{0}               | 恢复指定储藏内容（后面的东西通过git stash list查看）         |
| git cherry-pick 4c805e2                 | 复制一个特定的提交到当前分支                                 |
|                                         |                                                              |
| git reset --hard test                   | 在主分支时，将主分支重置为test分支                           |



