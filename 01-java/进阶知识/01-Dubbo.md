### 一.Dubbo简介

#### (一).三大核心功能

1. 面向接口的远程调用
2. 智能容错和负载均衡
3. 服务自动注册和发现

#### (二).基本架构

![](..\..\资源\dubbo架构.jpg)

#### (三).推荐使用的协议

在spring的配置文件中

```xml
<beans ...>
	<dubbo : protocol name="dubbo" port="20880"/>
</beans>
```



### 二.直连的方式

#### 1.在spring中的配置文件(服务提供者)

```xml
<beans ..>
    <!-- 1.服务提供者声明名称  （不要导错包了，Apache的包） -->
	<dubbo:application name="唯一标识（使用模块名）"/>
    <!-- 2.指定协议名称和端口号 -->
    <dubbo:protocol name="dubbo" port="208802"/>
    <!-- 3.暴露服务接口 -->
    <dubbo:service interface="接口的权限定类名" ref = "userService" registry="N/A"/>
    <!-- 4.将接口的实现类加载进spring容器 -->
    <bean id="userService" class = ..servie.impl.xxServiceImpl/>
</beans>
```

**注：还要在web.xml中添加监听器，创建创建唯一的spring容器**

```xml
<context-param>  
	<param-name>contextConfigLocation</param-name>  
	<param-value>classpath:spring.xml,classpath:springmvx.xml</param-value>  
</context-param>
<listener>
	<listener-class>..ContextLoadListener</listener-class>
</listener>
```

**要传输的类要进行序列化**

```java
public class User implement Serializable {}
```



#### 2. spring配置文件(消费者)

```xml
<beans  ...>
    <dubbo:application neme="消费者的唯一标识（模块名）"/>
    <dubbo:reference id="远程服务接口对象的名称 (userService,上面的)" 
                     intrface="com.xx.xxx.service.xxService"
                     url="dubbo://localhost:20880"
                     registory="N/A"/>
</beans>
```

**注：要想让消费者知道提供者提供了那些接口，要将接口大哥jar包放入到消费者中，（在pom文件中引入）**

***要想解决,可以将接口和实现类分离***

### 三.使用注册中心zookeeper

#### 1.其他文件

- 同自连方式一样，有接口，有接口的实现类，有消费工程

#### 2.接口实习类的spring配置文件（服务提供者）

```xml
<beans ...>
    <!-- 1.声明唯一标识（一定不要导错包了） -->
	<dubbo:application name="唯一标识（模块名）"/>
    <!-- 2.声明dubbo 的协议和端口号 -->
    <dubbo:protocol name = "dubbpo" port="20880"/>
    <!-- 3.使用zookeeper注册中心指定注册的地址和端口号 -->
    <dubbo:registry address="zookeeper://localhost:2181"/>
    <!-- 4.暴露接口 -->
    <dubbo:service interface="..service.xxService" ref="userService"/>
    <!-- 5.加载接口的实现类 -->
    <bean id = "userService" class = "com..service.impl.xxServiceImpl"/>
</beans>
```

**注：**

- 要加载工程的依赖
- 要加载zookeeper的依赖

#### 3.消费者工程配置文件

```xml
<beans ...>
    <!-- 1.声明消费者唯一标识（一定不要导错包了） -->
    <dubbo:application name="唯一标识（模块名）"/>
    <!-- 2.指定注册中心 -->
    <dubbo:registry address="zookeeper://localhost:2181"/>
    <!-- 3.引用远程接口服务 -->
    <dubbo:reference id ="远程接口实习类的ID名（上面的userService）" 
                     interface ="com..service.xxService"/>    
</beans>
```



#### 4.一个接口多个版本

- **使用 version 属性解决问题**

- 接口实现工程的spring配置文件

  ```xml
  <beans  ...>  
      <!-- 添加版本号 -->
      <dubbo:service interface="..service.xxService" ref="" version="1.0.1"/>
      <dubbo:service interface="..service.xxService" ref="userService2" version="1.0.2"/>
      <!-- 加载接口的实现类 -->
      <bean id = "userService1" class = "com..service.impl.xxServiceImpl"/>
      <bean id = "userService2" class = "com..service.impl.xxServiceImpl"/>
  </beans>
  ```

  

- 消费工程的spring配置文件

  ```xml
  <beans  ...>
      <!-- 引用远程接口服务 -->
      <dubbo:reference id ="userService1" 
                       interface ="com..service.xxService"
                       version="1.0.1"/> 
      
      <dubbo:reference id ="userService2" 
                       interface ="com..service.xxService"
                       version="1.0.2"/>  
  </beans>
  ```

  

