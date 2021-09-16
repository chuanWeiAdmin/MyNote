### Tomcat 的配置

1. File-->Settings

2. Build,Excution Deployment -->Application Servers

3. 点击“+”号，添加TomCat服务器

4. 添加TomCat的开关

   1. Run-->Edit Configaration
   2. 点击“+”号，选择Tomcat服务器（可以选择本地或者远程服务器）
   3. 设置基本信息

5. 创建网站

   new modle -->Java Edterprise --> web Application 

6. 把项目交给TomCat管理

   Run-->Edit Configaration -->Deployment -->"+"-->Artifact-->Application context 起别名



### 创建model一定要有的目录 

1. src：放Java文件

2. web：静态资源文件，jar包，配置文件

   ​        web--->WEB-INF--->Lib     ***自己创建用来存放   jar  包***

注：将   jar  包添加到    lib   下的步骤

File--> Project Structare --->modules(***找到网站项目***)--->Dependencies-->"+"-->Libary

