### 第一部分：springBoot入门

#### 一.springBoot基本知识

##### 1.springBoot的四大核心

- 自动配置
- 起步依赖
- Actuator
- 命令行界面

##### 2.创建一个springBoot工程

点击 new modle--> Spring Initializr(创建springBoot项目使用模板需要网络)--->添加基本信息-->Dependencies(选择依赖)

##### 3.springBoot最重要的注解

@SpringBootApplication  :主要用来开开启spring的自动配置，放在main方法只是，是程序的入口

##### 4.resources下的包

- static  :  静态文件  css ,js  ,.....
- templates  :  存放模板
- application.properties  ：spring的核心配置文件

**注：springBoot项目所写的代码都要放在Application的同级或子级目录下**

#### 二.springBoot的核心配置文件

##### 1.设置内嵌TomCat端口号

- server.port=8080

##### 2.设置TomCat上下文根

- 访问地址  url  后面的东西
- server.servlet.context-path=/springBoot
- 一定要以“/”开头，如果设置为空 一定要设置“/”

##### 3.yml文件或yaml文件

- 值与前面的  ： 之前一定要有空格

##### 4.多种类型的配置文件共存

- 当多种格式的配置文件都存在时 以 .properties为准

##### 5.springBoot的多配置文件

- application-dev.properties
- application-test.properties
- ........配置文件都要以application 开头
- 在主配置文件application.properties中设置属性  **spring.profiles.active=dev**  (上面  -  后面的名字)

##### 6.使用自定义的配置（String）

- 使用注解  @value("${key}")

- 注解可以在任何地方使用

- ```java
  public class TestController{
      @Value("${name}")  //这样就可以通过容器注入属性
      private String name;
  }
  ```

##### 7.获取自定义的配置（对象）

- applcation.properties中的配置

  ```properties
  schoole.name="myName"
  school.websit="myweb"
  ```

- 自定义一个类

  ```java
  @Component  //将对象交给容器来管理
  @ConfigurationProperties(prefix="school")//使用这种方法一定要有一个前缀
  public class School{
      private String name;
      private String websit;
      //setAndGet方法
  }
  ```

- 在其他类中注入容器中的对象

  ```java
  public class Test{
  	@Autowired  //自动注入
      private School school;
  }
  ```

##### 8.解决@ConfigurationProperties警告问题

```xml
<!-- 在pom文件中 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



#### 三.springBoot前端使用jsp

##### 1.添加对对jsp解析依赖

```xml
<!-- 在pom文件中 -->
<!-- tomcat的支持-->
<dependency>
       <groupId>org.apache.tomcat.embed</groupId>
       <artifactId>tomcat-embed-jasper</artifactId>
</dependency>     
```

##### 2.jsp的编译

- springboot推荐使用的是thymeleaf 现在要使用Jsp要手动的指定Jsp的编译路径

- 集成好的路径在META-INF/resources

- 在  buid  标签下添加

- ```xml
  <resources>
  	<resource>
          <!-- 源文件 -->
          <directory>src/main/java</directory>
          <!-- 指定编译到 META-INF/resources -->
          <targetpath>META-INF/resources</targetpath>
          <!-- 那些文件需要编译 -->
          <includes>
              <include>*.*</include>
          </includes>
      </resource>
  </resources>
  ```

  

##### 3.application中配置视图解析器

- spring.mvc.view.prefix=/WEB-INF/

- spring.mvc.view.suffix=.jsp

- ```properties
  #在application文件中设置
  
  #Spring boot视图配置
  spring.mvc.view.prefix=/WEB-INF/
  spring.mvc.view.suffix=.jsp
  ```

  

****

### 第二部分：springBoot Web开发

#### 一.springBoot集成mybatis

##### 1.添加依赖

- mybatis  

- MySQL驱动

- ```xml
  <!-- MySQL驱动 -->
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.xx</version><!-- 默认的版本是8.0的可以自己改 -->
  </dependency>
  
  
  <!-- mybatis整合SpringBoot框架 -->
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.0.0</version>
  </dependency>
  ```

##### 2.mybatis逆向工程

- 用来生成数据库中向对应的类，生成一些文件（完全记不住）

1. 添加   GeneratorMapper.xml   文件及内容

2. 添加反向工程的依赖(是个插件放在plugin标签中)

3. 在  GeneratorMapper.xml   中jdbc的驱动一定要对

4. 点击maven-->plugins-->mybatis-generator-->mybatis-generator(第一个)

   **注：数据库中的字段多个单词一定要用  _  来分割，mybatis逆向工程只能从  _  转换成驼峰**

##### 3.Dao文件加入到spring容器中

- 使用  @Mapper  注解 将dao接口加入到容器中

- ```java
  @Mapper
  public interface StudentDao{
      //方法
  }
  ```

  

##### 4.连接数据库的配置

```properties
# DB Configuration
#指定数据库驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#数据库jdbc连接url地址,serverTimezone设置数据库时区东八区
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/user
#数据库账号
spring.datasource.username=root
spring.datasource.password=root
```



##### 5.让spring容器识别到mapper的两种方式

- 在dao接口上添加  @Mapper注解

- 在入口(main方法上)加入@MapperScan注解

  ```java
  @SpringBootApplication
  @MapperScan(basePackage="要扫描的包")  //添加之后能全部扫描进去
  public class Application{
      public static void main(String[] args)
      {
      }
  }
  ```

  

##### 6.mapper映射文件的存放的第二种方式

- 将Mapper文件放到resources/mapper下
- 在application配置文件中添加mapper映射文件的位置
- **mybatis.mapper-location=classpath:mapper/*.xml**

##### 7.springBoot集成事务

- 在service的方法上添加注解  @Transaction

  ```java
  @Service
  public class TestService{
      @Transaction
      public int updataTest(){
          //要执行的方法
      }
  }
  ```

  

#### 二.springBoot下web

##### 1.@RestController

- 如果返回值全为Json可以在类上添加  @RestController 
- 相当于  @Controller  +  @ResponseBody

##### 2. @GetMapping 只能接收Get请求

- @GetMapping(value="/xx")
- 查询数据

##### 3.@PostMapping 只能接收Post请求

- @PostMapping (value="/xx")
- 新增数据

##### 4.@DeleteMapping  只能接收Delete请求

- @DeleteMapping(value="/xx")
- 删除数据

##### 5.@PutMapping  只能接收put请求

- @PutMapping(value="/xx")
- 更新请求

#### 三.springBoot实现RESTFull

##### 1.RESTFull的请求风格

- http协议：localhost:8080/student?id=1001&name=zs
- RESTFull : locathost:8080/student/1001/zs

##### 2.实现方式

```java
@Controller
public class MyController{    
    @RequestMapping(value="/student/{id}/{age}")
    public Object student(@PathVariable("id") Integer id,
                         @PathVarable("age") String age){
        //要执行的方法
        return "";
    }
}
```



##### 3.RESTFull请求冲突解决方式

- 使用不同的请求方式解决
- 更改请求参数  (url) 位置来解决
  - /student/{id}/detail ..
  - /student/detail/{id}

#### 四.springBoot集成Rides

##### 1.添加依赖

- springboot集成Redis的起步依赖

  ```xml
  <!-- pom文件中 -->
  <!-- redis依赖 -->
  <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

  

##### 2.设置redis的配置信息

```properties
spring.redis.host=127.0.0.1
#Redis服务器连接端口
spring.redis.port=6379

######################下面可不填#######################

#Redis服务器连接密码（默认为空）
spring.redis.password=
#连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
#连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.pool.max-idle=8
#连接池中的最小空闲连接
spring.redis.pool.min-idle=0
#连接超时时间（毫秒）
spring.redis.timeout=30000

```



##### 3.向redis中放值

```java
@Service
public class StudentService{
    @Autowired
    private RedisTemplate<Object,Object> redisTemplate;
    
    public void put(String key,String Value){
        //向redis中存数据
        redisTemplate.OPSForValue().set(key,value);
    }
    
}
```



##### 4.从redis中取值

```java
@Service
public class StudentService{
    @Autowired
    private RedisTemplate<Object,Object> redisTemplate;
    
    public void put(String key,String Value){
        //向redis中获取数据
        String value=(String)redisTemplate.OPSForValue().get(key);
    }
    
}
```

#### 五.spring集成Dubbo

##### 1.服务提供者的依赖

- dubbo集成spring的起步依赖
- 注册中心的依赖（zookeeper）
- 接口工程的依赖

##### 2.服务提供者的application配置

```properties
#下面的配置没有配置
spring.application.name=(唯一标识)一般是模块的包名
#当前工程是一个服务提供者
spring.application.server=true
#设置注册中心
spring.application.registry=zookeeper://127.0.0.1:2181
```



##### 3.消费者工程的依赖

- 同服务者的依赖相同

##### 4.消费者工程的application配置

```properties
spring.appliction.name=(唯一标识)一般是模块的包名
spring.application.registry=zookeeper://127.0.0.1:2181
```



##### 5.服务者提供的方法

```java
@Component  //将对象交给容器管理
//这里的注解一定要是alibaba.dubbo.config.annotation  下的注解
@Service(interfaceClass=StudentService.class,version="1.0.0",timeout=15000)
public class StudentServiceImpl implements StudentService {
    public Integer query(){
        return 1250;
    }
}
```



##### 6.消费者工程使用的方法

```java
@Controller
public class StudentController{
    @Reference(interfaceClass=StudentService.class,version="1.0.0",check=false)
    private StudentService service;
    //这样就能将消费者工程中的接口注入进来
    
    //....... 一些方法
} 
```



##### 7.主入口添加注解

- 使用dubbo要在主入口中开启Dubbo的注解

  ```java
  @SpringBootApplication
  @EnableDubboConfigration  //开启Dubbo注解
  
  ```

  

##### 8.实体类在网络中传输一定要序列化

```java
public class Student implements Serializable{
    //属性
    //set和get方法
}
```



****

### 第三部分：springboot集成其他

#### 一.springBoot非web开发(了解)

##### 1.第一种实现方式

- 创建springBoot时不选择web

- 通过容器拿到对象

  ```java
  @SpringBootApplication
  public class Application{
      public static void main(String[] args){
          //获取容器
          configurableApplicationContext applicationContext=
              SpringApplication.run(Application.class,args);
          //从容器中获取指定对象
          Student stu=(Student)applicationContext.getBean("类名的首字母小写");
          //就可以通过对象调取方法了
      }
  }
  ```

- springBoot启动时返回值也是ConfigurableApplicationContext也是一个容器，想当于spring启动容器ClassPathXmlApplication

##### 2.第二种实现方式

```java
@SpringBootApplication
public class Application implements CommandLinRunner{
    @AutoWired
    private StudentService service;
    
    public static void main(String[] args){       
    }
    
    public void run (String ...args)throws Execption{
        service.sayHello();
    }
    
}
```



##### 3.关闭/改变启动  logo 

- 在resource下放入banner.txt 内容就是logo

#### 二.springBoot使用拦截器

##### 1.实现HandlerInterceptor接口，并重写三个方法

- 同springMVC相同

##### 2.编写配置类

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer{
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new ReqInterceptor())
            .addPathPatterns("/**")//l拦截的路径
            .exdudePathPatterns();//排除的路径
    }
}
```



#### 三.springBoot使用过滤器 Filter (了解)

##### 第一种方式：注解方式

##### 第二种方式：通过配置类（注册组件）

#### 四.springBoot设置字符编码

#### 五.springBoot打包

#### 六.springBoot集成logback

#### 七.springBoot集成Thymeleaf
