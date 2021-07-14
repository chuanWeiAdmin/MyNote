### 零.关键内容

重点内容：DispatcherServlet 中央调度器

![](..\..\资源\DispatcherServlet.png)

### 一.SpringMVC实现步骤

#### (1).基本步骤

1. 新建maven项目
2. 加入依赖
   - Spring-webMVC依赖，会间接加入Spring的依赖
   - jsp，servlet依赖
3. **在web.xml中注册SpringMVC核心对象DispatcherServlet**
   - 是一个Servlet 叫中央调度器，父类 HttpServlet
   - 也叫前端控制器   (foront Controller)
   - 负责接收用户的请求，调用其他控制器类对象，显示结果
4. 创建发起请求的页面
5. 创建控制器类
   - 在类上加入@Conterller注解，并添加到SpringMVC容器中
   - 在类中的方法上加入@RequestMapping注解
6. 创建Spring MVC的配置文件**（和Spring 的配置文件一样）**
   - **声明组件扫描器，指定@Controller在的包名**
   - 声明视图解析器，处理视图

#### (2).代码和配置文件

1. 创建中央调度器，在web.xml文件中

   ```xml
   <!-- 同其他控制器类相同 -->
   
   <servlet>
   	<servlet-naem>myWeb</servlet-naem>
       <servlet-class>..DispatcherServlet</servlet-class>
       <!-- 自定义SpringMVC要读取的配置文件 -->
       <init-param>
       	<param-naem>ContextConfigLoaction</param-naem>
           <!-- 自定义的Spring配置文件位置 -->
           <param-value>classpatch:SpringMVC.xm</param-value>
       </init-param>
       <!-- 自定义启动顺序,数字越小启动越早 -->
       <load-on-statop>1</load-on-statop>
   </servlet>
   <servlet-mapper>
   	<servlet-name>myWeb</servlet-name>
       <url-pattern>*.do</url-pattern>
       <!-- 有两种定义方式 -->
       <!-- *.xxx xxx自定义的扩展名，常用 .do  .action -->
       <!-- 使用斜杠  /  -->
   </servlet-mapper>
   ```

2. 创建控制器类

   ```java
   //使用注解一定要在Spring配置文件中添加组件扫描器
   @Controller
   public class MyController{
       @RequestMapping(value="/some.do")
       public ModelAndView doSome(){
           ModelAndView mv = new ModelAndView();
           //添加数据
           mv.addObject("msg","要放到request中的数据");
           //添加视图
           mv.setViewName("show");//要这么写一定要加视图解析器
           return mv;
       }
   }
   ```

   @RequestMapping 请求映射，作用是将地址和一个方法绑定在一起，一个请求指定一个方法

   - 属性：value 表示请求的url地址，必须保证唯一，**推荐使用"/"开头**
   - 位置：
     - 在方法上（常用）
     - 在类的上面（表示全部方法之前的地址）
   - 说明：使用RequestMapping修饰方法叫 **处理器方法** 或 **控制器方法**

3. 视图解析器

   ```xml
   <beans ...>
   	<bean class="..InternalResourceViewResolver">
           <!-- 前缀 ， 视图文件的路径，value一定要前后都有"/" -->
       	<property name="prefix" value="/WEB-INF/view/"/>
           <!-- 后缀，视图文件的扩展名 -->
       	<property name="suffix" value=".jsp"/>
       </bean>
   </beans>
   ```

4. 声明组件扫描器（用来扫描注解）

   ```xml
   <beans ...>
   	<!--声明组件扫描器-->
   	<context:component-scan base-package="com.bjpowernode.controller" />
   </beans>
   ```

****

### 二.接受参数

#### (1).RequestMapping用法

- 位置：
  - 在类的上面，表示路径公共的部分
  - 在方法的上面，表示要访问方法的url路径
- 属性
  - value : 表示要访问此方法的路径
  - method : 表示访问方法是通过 post / get方法  通过RequestMethod 的枚举值来设置值
    - @RequestMapping(value="xx/xx",method=RequestMethod.POST)   //表示只能通过post方法访问
    - @RequestMapping(value="xx/xx",method=RequestMethod.GET)   //表示只能通过get方法访问
  - produces ：设置格式   (produces="text/plain;charset=utf-8")

#### (2).处理器方法获取参数的四类

1. HttpServletRequest
2. HttpServletResponse
3. HttpServletSession
4. 请求中所携带的请求参数（Object）

#### (3).逐个接收

- 在方法中的形参名和传过来的参数名一致

  ```java
  @Controller
  public class MyController{
      @RequestMapping(value="/some.do")
      public ModelAndView doSome(String name , Interage age ){
          ...
      }
  }
  ```

- 通过以上方法接收的参数会存在中文乱码

- 可以通过自定义过滤器解决

- 可以通过框架中写好的过滤器来解决乱码的问题，在web.xml文件中添加过滤器

  ```xml
  <filter>
  	<filer-name>charcterEncodingFilter</filer-name>
      <file-class>..CharacterEncodingFilter</file-class>
      <!-- 设置项目中的字符集 -->
      <init-param>
      	<param-name>encoding</param-name>
          <param-value>utf-8</param-value>
      </init-param>
      <!-- 强制请求使用编码格式 -->
      <init-param>
      	<param-name>forceRequestEncoding</param-name>
          <param-value>true</param-value>
      </init-param>
      <!-- 强制响应使用编码格式 -->
       <init-param>
      	<param-name>forceResponseEncoding</param-name>
          <param-value>true</param-value>
      </init-param>
  </filter>
  <filter-mapping>
  	<filer-name>charcterEncodingFilter</filer-name>
      <url-pattern>/*</url-pattern><!-- 强制使用所用请求先进过滤器 -->
  </filter-mapping>
  ```

  

#### (4).请求中参数名和形参不一致

使用  @RequestParam   解决请求中参数名与形参名不一致的情况

- 属性：

  - value 请求中的参数名
  - required 默认是true，必须包含此参数，false 可以不用包含此参数

- 位置：在处理器方法的形参定义的前面

  ```java
  @Controller
  public class MyController{
      @RequestMapping(value="/some.do")
      public ModelAndView doSome(@Requestparam(value="myName")String name , 
                                 @Requestparam(value="myAge")Interage age ){
          ...
      }
  }
  ```

  

#### (5).通过对象接收参数

```java
@Controller
public class MyController{
    @RequestMapping(value="/some.do")
    public ModelAndView doSome(Student name){
        //Student 自定义的类，类中的属性名就是请求参数名
        ...
    }
}
```



****

### 三.返回数据

#### (1).四种返回值类型

- ModelAndView
- String
- void  (很少用)
- 返回对象   Object

#### (2).返回String类型

注：表示视图，可以是逻辑名称也可以是完整的视图路径

```java
@Controller
public class MyController{
    @RequestMapping(value="/some.do")
    public String doSome(String name , Interage age ){
        //这样写要配置视图解析器
        return "show";//表示转发到show.jsp页面当中
    }
}
```



#### (3).返回 void 类型

- 不能表示数据也不能表示视图，在处理Ajax的时候可以使用void返回值
- 通过HttpServletRequest返回数据
- ajax只有数据没有视图

#### (4).返回对象 Object

- 主要响应ajax

- 在处理ajax请求中主要用到json实现步骤

  - 加入处理json的工具库的依赖（jackson） Spring使用 的就是jackson

  - 在springMVC配置文件中加入注解驱动   一定要主要是mvc结尾的

    ```xml
    <beans ...>
        <mvc:annocation-driven>
    </beans>
    ```

  - 在处理器方法中加入注解  （@ResponseBody）

    ```java
    @Controller
    public class MyController{
        @RequestMapping(value="/some.do")
        @ResponseBody  //返回Object一定要有
        public Student doSome(String name , Interage age ){
            Student st =new Student();
            st...
            return st;//表示转发到show.jsp页面当中
        }
    }
    ```

  - 常用的两个方法

    1. StringHTTPMessageConverter
    2. MappingJackson2HttpMessageConverter

#### (5).返回  List 集合

***同返回对象一样，会将List转换成json数值***

#### (6).返回String 类型的另一种用法

- 另一种用法，返回String类型数据是数据

- 区分String是数据还是视图就看有么有@ResponseBody注解

- String返回的数据使用的是  ISO-8859-1  数据类型，可以在@RequestMapping添加属性produces修改编码格式

- 代码实现

  ```java
  @Controller
  public class MyController{
      @RequestMapping(value="/some.do",produce="text/plain;chartset=utf-8")
      @ResponseBody  //返回String代表数据时一定要有这个
      public String doSome(String name , Interage age ){
          return "此时返回的内容就是数据，且解决了中文乱码";
      }
  }
  ```

  

****

### 四.DispatcherServlet处理静态资源

#### (1).当中央调度器中   url-pattern 为 /时

中央调度器中的  url-pattern 为 "/ "时表示不管是什么路径到要经过中央调度器，

但是中央调度器默认没处理静态资源的能力，有两种处理方法

1. 处理方法一：在Spring 的配置文件中添加标签，**使用默认的servlet处理静态资源**

   ```xml
   <beans  ...>
       <!-- 添加这个标签一定要添加注解驱动 -->
   	<mvc:default-servlet-handler/>
   
       <!-- 注解驱动 -->
       <mvc:annotation-driven/>
   </beans>
   ```

2. 处理方法二：将静态资源位置告诉Spring , 在Spring配置文件中添加静态文件的位置

   ```xml
   <beans ...>
       <!-- mapping：访问静态资源的uri -->
       <!-- location : 表示静态资源在项目中的位置 -->
       <!-- 可以存着多个 -->
   	<mvc:resources mapping="/images/**" loaction ="/images/"
   </beans>
   ```
   
   注：
   
   - 访问动态资源要额外添加注解驱动
   - 可以将文件同一放在一个目录下，这样配置一个同一的就可以了

****

### 五.Spring中核心技术

#### (一).请求的转发重定向

1. ##### 关键字

   - forward：表示转发，相当于  request.getRequestDispatcher("xxx.jsp").forward(req,resp);
   - redirect：表示重定向，response.sendRedirect("xxx.jsp")

2. ##### 处理器方法返回ModelAndView实现  --->  ***转发***

   - 语法  mv.setViewName("forward:视图的完整路径");

   - 特点：不和视图解析器一同使用，**（当做没有视图解析器）**

     ```java
     @Controller
     public class myController{
         @RequestMapping(value="/doSome.do")
         public ModelAndView doSome(){
             ModelAndView mv = new ModelAndView();        
             //设置转发
             mv.setViewName("forward:/hello.jsp");
             return mv;
     	}    
     }
     ```

3. ##### 处理器方法返回ModelAndView实现 ---> ***重定向***

   - 特点 和 语法格式同上

   - 注：

     1. 不能访问WEB-INF 下的内容
     2. 在目标页可以使用 ${param.key}获取值，入 ${param.name}

     ```java
     @Controller
     public class myController{
         @RequestMapping(value="/doSome.do")
         public ModelAndView doSome(){
             ModelAndView mv = new ModelAndView();        
             //设置转发
             mv.setViewName("redirect:/hello.jsp");
             return mv;
     	}    
     }
     ```

#### (二).异常处理

注：使用aop思想，将异常同一处理

##### (1).异常处理步骤

1. 新建maven项目
2. 加入依赖
3. 在controller抛出异常
4. 创建一个普通类，作为全局异常处理类
   - 在类上加入@controllerAdvice
   -  在类中定义方法，方法上加@ExceptionHandler
5. 创建处理异常类的视图
6. 创建SpringMVC的配置文件
   - 组件扫描器，扫描@Controller注解
   - 组件扫描器，扫描@ControllerAdvice所在的包名 
   - 声明注解驱动

##### (2).自定义异常类

- 一定要在类上添加@ControllerAdvice

- 一定要在方法上添加@ExceptionHandler

- **处理器方法能返回声明就能返回什么，处理器能接收什么就能接收什么**

  ```java
  @ControllerAdvice
  public class ClobalException{
     @ExceptionHandler(value=xxxException.class)
      public ModelAndView doSome(Exception ex){
          /*
          在异常处理中要执行的工作：
          1.把需要记录的信息记录下来
          2.发送消息，短信通知
          3.给用户友好的提示
          */
          ModelAndView mv = new ModelAndView();
          mv.sendObject(ex.message());
          mv.setViewName("error");
          return mv;
      }
  }
  ```

  

##### (3).ExceptionHandler中的属性

- @ExceptionHandler(value=NameException.class)  表示处理姓名异常(自定义异常)，**处理特定的异常**
- @ExceptionHandler  表示处理带有属性之外的全部异常

##### (4).Spring 中的配置文件

```xml
<beans ...>
    <!-- 组件扫描器 -->
	<context:component-sacn base-package="异常处理的包名"/>
    <!-- 注解驱动 -->
    <mav:annotation-driven/>
</beans>
```



#### (三).拦截器

##### (1).前情提要

1. 拦截器是SpringMVC中的，需要实现HandlerInterptor接口
2. 和过滤器类似
   - 过滤器：用来过滤请求参数，设置编码字符集
   - 拦截器：拦截用户的请求，在请求的判断
3. 拦截器可以是全局的，可以对多个Controller做拦截，可以有0个或多个，用着，用户登录，权限检查，记log

##### (2).拦截器的使用步骤

1. 定义类实现HandlerIntercepor接口
2. 在SpringMVC的配置文件中，声明拦截器，**让框架知道自定义的拦截器存着**

##### (3).拦截器执行的时间

- 分别对应了三个方法

1. 在请求处理之前执行（首次）---->preHandle  方法
2. 在控制请执行之后会执行----------->postHandle  方法
3. 在请求处理完成也会执行----------->afterCompletion  方法

##### (4).代码实现

1. 实现HandleInterceptor并重写三个方法（5.25之后用谁重写谁）

2. **preHandle**（预处理方法）

   - 参数： Object  handle ： 被拦截的控制器对象
   - 返回值      boolean
     - true：**请求通过验证，可以执行处理器方法**
     - false：**请求没有通过验证，被拦截器截止了（没有之后的方法了）**
   - 特点
     - 在控制器方法之前执行
     - 可以获得请求信息，判断信息

3. **postHandle**（后处理方法）

   - 参数：
     - Object  handle ： 被拦截的控制器对象
     - ModelAndView mv 处理器方法的返回值
   - 特点：在处理器方法之后执行，可以拿到处理器方法的返回值并且修改

4. **afterCompletion** （最后执行的方法)

   - 参数：
     - Object  handle ： 被拦截的控制器对象
     - Exception  ex 程序中的异常
   - 特点：对视图执行forward请求完成之后才执行，一般用作资源回收工作

   ```java
   
   ```

5. 声明拦截器（在Spring 的配置文件中）

   ```xml
   <beans ...>
       <!-- 声明第一的拦截器对象。（可以有多个拦截器对象） -->
   	<mvc:interceptor>
           <!-- 指定拦截器的uri，表示访问这个地址，首先要经过这个拦截器 -->
       	<mvc:mapping path="/user/**/"/>
           <!-- 声明要使用的拦截器对象 -->
           <bean class="包名+类名"/>
       </mvc:interceptor>
   </beans>
   ```

   

##### (5).过滤器和拦截器的区别

1. 过滤器是servlet中的对象，拦截器是框架中的对象

2. 过滤器是实现Filter接口，拦截器是实现HandlerInterceptor接口

3. 过滤器是用来设置request，response参数属性，侧重对数据的过滤

   拦截器是用来验证请求的，能拦截请求

4. 过滤器是在拦截器之前执行的

5. 过滤器的Tomcat创建的对象

   拦截器是Spring MVC创建的对象

6. 过滤器只有一个执行时机

   拦截器有三个执行时机

7. 过滤器可以处理jsp，js，html等

   拦截器侧重对Controller对象的拦截，如果请求不被DispatcherServlet接收，这个请求不会执行拦截器的内容

8. 拦截器拦截普通方法

   过滤器过滤servlet请求响应

#### (四).DispatcherServlet流程图

![](..\..\资源\SpringMVC流程.png)
