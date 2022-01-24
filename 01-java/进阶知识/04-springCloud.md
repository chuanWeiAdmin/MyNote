#### 一.RestTemplate的使用

1. 创建配置类

   ```java
   @Configuration
   public class ApplicationContextConfig
   {
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate()
       {
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
   public class OrderZKController
   {
       public static final String INVOKE_URL = "http://cloud-provider-payment";
       
       //注入对象
       @Resource
       private RestTemplate restTemplate;
   
       @GetMapping(value = "/consumer/payment/zk")
       public String paymentInfo()
       {
           String result = restTemplate.getForObject(INVOKE_URL+"/payment/zk",String.class);
           return result;
       }
   }
   
   ```

#### 二.服务注册与发现

##### 1.Eureka服务注册与发现

- **服务治理**

  Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理，在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。 

-   **服务注册与发现**：

  Eureka采用了CS的设计架构，

  Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。在服务注册与发现中，有一个注册中心。**当服务器启动的时候，会把当前自己服务器的信息 比如 服务地址通讯地址等以别名方式注册到注册中心上**。**另一方（消费者|服务提供者），*以该别名的方式*去注册中心上获取到实际的服务通讯地址**，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))

- pom文件

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

- yaml配置

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

  

