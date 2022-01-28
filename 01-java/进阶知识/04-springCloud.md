### 一.RestTemplate的使用

1. 创建配置类

   ```java
   @Configuration
   public class ApplicationContextConfig {
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   }
   ```
   
2. 通过方法调用分布式方法

   - 通过 getForObject (url, requestMap, ResponseBean.class)方法
   - 这三个参数分别代表 **REST请求地址**、**请求参数**、**HTTP响应转换被转换成的对象类型**

   ```java
   @RestController
   @Slf4j
   public class OrderZKController {
       public static final String INVOKE_URL = "http://cloud-provider-payment";
   
       //注入对象
       @Resource
       private RestTemplate restTemplate;
   
       @GetMapping(value = "/consumer/payment/zk")
       public String paymentInfo() {
           String result = 
               restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
           return result;
       }
   }
   
   ```

****

### 二.服务注册与发现

#### 1.Eureka服务注册与发现

- ##### **服务治理**

  Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理，在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。 

- ##### **服务注册与发现**：

  Eureka采用了CS的设计架构，

  Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。在服务注册与发现中，有一个注册中心。**当服务器启动的时候，会把当前自己服务器的信息 比如 服务地址通讯地址等以别名方式注册到注册中心上**。**另一方（消费者|服务提供者），*以该别名的方式*去注册中心上获取到实际的服务通讯地址**，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))

- ##### pom文件

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

- ##### yaml配置

  ```yaml
  server:
    port: 7001
  
  
  eureka:
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client:
      register-with-eureka: false     #false表示不向注册中心注册自己。
      #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      fetch-registry: false     
      
      service-url:
      #集群指向其它eureka
        #defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
        defaultZone: http://eureka7001.com:7001/eureka/
    #server:
      #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
  ```

- ##### 将生产者注册到Eureka

  1. 添加pom依赖

     ```xml
     <!--eureka-client-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
     </dependency>
     ```

  2. 更改yaml文件

     ```yaml
     eureka:
       client:
         #表示是否将自己注册进EurekaServer默认为true。
         register-with-eureka: true
         #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
           # 集群版
           #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
       instance:
           instance-id: payment8001
           #访问路径可以显示IP地址
           prefer-ip-address: true
           #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
           #lease-renewal-interval-in-seconds: 1
           #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
           #lease-expiration-duration-in-seconds: 2
     
     ```

  3. **在主启动类中添加注解**

     ```java
     @SpringBootApplication
     @EnableEurekaClient
     public class PaymentMain8001
     {
         public static void main(String[] args) {
             SpringApplication.run(PaymentMain8001.class, args);
         }
     }
     ```

- ##### 将消费者注册到 Eureka 中

  **同服务者相同**

- ##### **Eureka 搭建服务端集群**

  相互调用相互守望

  ```yaml
  eureka:
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client:
      register-with-eureka: false     #false表示不向注册中心注册自己。
      #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      fetch-registry: false    
      
      service-url:
      #集群指向其它eureka
        #defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
        defaultZone: http://eureka7001.com:7001/eureka/
  ```

- ##### 使用生产者集群的时候出现的问题

  使用 @Bean 标签的时候RestTemplate不具备负载均衡的能力，所以会报错

  要在*配置类*中使用**@LoadBalanced注解赋予RestTemplate负载均衡的能力**

  ```java
  @Configuration
  public class ApplicationContextConfig {
      //@Bean
      @LoadBalanced //使用注解使 RestTemplate 具有负载均衡的能力
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

- ##### actuator 微服务信息完善

  1. 主机名称:服务名称的修改，让Eureka上不显示主机名称

     ```yaml
     eureka:
       client:
         register-with-eureka: true
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
       instance: #让Eureka现在服务名
           instance-id: payment800
     ```

  2. 访问路径有ip提示

     ```yaml
     eureka:
       client:
         register-with-eureka: true
         fetchRegistry: true
         service-url:
           #单机版
           defaultZone: http://localhost:7001/eureka
       instance:
           instance-id: payment8001
           #访问路径可以显示IP地址
           prefer-ip-address: true
     ```

- ##### 服务发现 Discovery 

  1. 修改controller

     ```java
     @RestController
     @Slf4j
     public class PaymentController {
         @Resource
         private DiscoveryClient discoveryClient;
     
         @GetMapping(value = "/payment/discovery")
         public Object discovery() {
             //获得 全部的服务
             List<String> services = discoveryClient.getServices();
             for (String element : services) {
                 log.info("*****element: " + element);
             }
     
             //获取 其中一个服务的全部实例
             List<ServiceInstance> instances = 
                 discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
             for (ServiceInstance instance : instances) {
                 log.info(instance.getServiceId()
                         + "\t" + instance.getHost()
                         + "\t" + instance.getPort()
                         + "\t" + instance.getUri());
             }
     
             return this.discoveryClient;
         }
     }
     ```

  2. 修改启动类

     ```java
     @SpringBootApplication
     @EnableEurekaClient		//这个注解之后可能不常用了
     @EnableDiscoveryClient	//这个注解之后还会用到
     public class PaymentMain8001
     {
         public static void main(String[] args) {
             SpringApplication.run(PaymentMain8001.class, args);
         }
     }
     ```

#### 2.Zookeeper服务注册与发现

- 服务提供者添加进注册中心

  1. 修改pom文件

     **注意：如果zookeeper的版本和服务器上的版本不同会报错，并且springboot会自带一个zookeeper，要先排除这个自带的**

     ```xml
     <!-- SpringBoot整合zookeeper客户端 -->
     <dependency>
     	<!--springBoot中会带一个zookeeper如果同注册中心的版本不符会报错-->
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
         <!--所以先排除自带的zookeeper3.5.3-->
         <exclusions>
             <exclusion>
                 <groupId>org.apache.zookeeper</groupId>
                 <artifactId>zookeeper</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     <!--添加同服务器版本相同的zookeeper3.4.9版本-->
     <dependency>
         <groupId>org.apache.zookeeper</groupId>
         <artifactId>zookeeper</artifactId>
         <version>3.4.9</version>
     </dependency>
     ```

  2. 修改yaml文件

     ```yaml
     #8004表示注册到zookeeper服务器的支付服务提供者端口号
     server:
       port: 8004
     #服务别名----注册zookeeper到注册中心名称
     spring:
       application:
         name: cloud-provider-payment  #会在zookeeper中显示的名字
       cloud:
         zookeeper:
           connect-string: 192.168.111.144:2181  #2181 zookeeper的默认端口
     ```

  3. 设置主启动类

     ```java
     @SpringBootApplication
     //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
     @EnableDiscoveryClient
     public class PaymentMain8004
     {
         public static void main(String[] args) {
                 SpringApplication.run(PaymentMain8004.class, args);
         }
     }
     ```

- 服务消费者注册近zookeeper

  1. 修改 yaml 文件

     **同上**

  2. 修改pom文件

     **同上**

  3. 设置主启动类

     **同上**

  4. 设置RestTemplate的配置文件

     ```java
     @Configuration
     public class ApplicationContextConfig{
         @Bean
         @LoadBalanced
         public RestTemplate getRestTemplate(){
             return new RestTemplate();
         }
     }
     ```

#### 3.consul 的注册和发现

- 微服务注册进consul

  1.  加pom

     ```xml
     <!--SpringCloud consul-server -->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-consul-discovery</artifactId>
     </dependency>
     ```

  2. 改yaml

     ```yaml
     ###微服务的端口号
     server:
       port: 8006
     
     spring:
       application:
         name: consul-provider-payment
     ####consul注册中心地址
       cloud:
         consul:
           host: localhost
           port: 8500
           discovery:
             #hostname: 127.0.0.1
             service-name: ${spring.application.name}
     ```

  3. 主启动类

     **同上**

- 将服务消费者注册进 consul

  **同上**

****

### 三.负载均衡

#### 1.Ribbon 实现负载均衡

- **Ribbon 是在客户端的一种负载均衡，软负载均衡**
- *Ribbon 就是（负载均衡 + RestTemplate 调用）*
- 

#### 2.Ribbon 核心组件 IRule

- 可以根据特定的算法从服务列表中选取一个要访问的服务（7种模式）

  |      | 模式                                    | 简介                                                         |
  | ---- | --------------------------------------- | :----------------------------------------------------------- |
  | 1    | com.netflix.loadbalancer.RoundRobinRule | 轮询                                                         |
  | 2    | com.netflix.loadbalancer.RandomRule     | 随机                                                         |
  | 3    | com.netflix.loadbalancer.RetryRule      | 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务 |
  | 4    | WeightedResponseTimeRule                | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择 |
  | 5    | BestAvailableRule                       | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务 |
  | 6    | AvailabilityFilteringRule               | 先过滤掉故障实例，再选择并发较小的实例                       |
  | 7    | ZoneAvoidanceRule                       | 默认规则,复合判断server所在区域的性能和server的可用性选择服务器 |

- 实现方法

  1. 配置类不能同 @ComponentScan 即 @SpringBootApplication 同包获子包下

  2. 在同包上一层新建一个包存放 IRule 的配置类

     ```java
     @Configuration
     public class MySelfRule {
         @Bean
         public IRule myRule() {
             //定义为随机，一共有 7 种类型
             return new RandomRule();
         }
     }
     ```

  3. RestTemplate 的配置类

     同上 第一 节 的RestTemplate 配置类

     ```java
     @Configuration
     public class ApplicationContextConfig {
         @Bean
         //@LoadBalanced
         public RestTemplate getRestTemplate() {
             return new RestTemplate();
         }
     }
     ```

  4. 主启动类添加注解

     @RibbonClient(name = "要访问的服务名",configuration=自定义的配置类.class)

     -  name  要访问的服务名
     -  configuration 自定义的配置类

     ```java
     @SpringBootApplication
     @EnableEurekaClient
     @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
     public class OrderMain80
     {
         public static void main(String[] args) {
                 SpringApplication.run(OrderMain80.class, args);
         }
     }
     ```

     
