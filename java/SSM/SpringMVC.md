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

#### (3).返回 void 类型

#### (4).返回对象 Object

#### (5).返回  List 集合

#### (6).返回String 类型的另一种用法



****

### 四.DispatcherServlet处理静态资源

****

### 五.Spring中核心技术

