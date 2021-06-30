### 一.Spring

#### 1.IOC技术实现

DI ：（Dependency Injection ） 依赖注入

只需要在程序中提供要使用的对象名称，至于对象如何在容器中创建，都由容器内部实现



#### 2.通过Spring创建对象的步骤

1. 创建maven项目
2. 加入依赖   juint   Spring  .....
3. 创建类（接口和它的实现类），同普通类一样
4. 创建Spring 的配置文件

#### 3.Spring 的配置文件

​		注：在idea中加入Spring的依赖之后  ， 打xml就可以出来

在Spring中使用配置文件创建对象，使用的是<bean> 标签

```xml
<bean id="" class=""/>
<!-- id: 对象的自定义名称（在文件中要唯一）-->
<!-- class : 类的全限定名称（成classpath下面的位置）例com.test.Student -->
```



#### 4.使用Spring创建对象

1. 指定配置文件的路径
2. 创建容器
3. 通过容器的getBean方法得到对象

```java
//1.指定配置文件的全路径  (在resources下的文件路径)
String config="beans.xml";

//2.创建容器（有一个对应的方法是根据磁盘中的位置创建容器）
ApplicationContext as=new ClassPathXmlApplicationContext(config);

//3.调用getBean会得到一个object对象要强转
Student s=(Student) as.getBesn("id名");

```

#### 5.Spring中提供的常用方法

1. as.getBeanDefinitionCount();  返回int 有几个对象

2. as.getBeanDefinitionName();  返回String[]  Spring中全部对象的名字

   ```java
   String config="beans.xml";
   ApplicationContext ac =new ClassPathXmlApplicationContext(config);
   
   //1.返回对象的个数
   int count= ac.getBeanDefinitionCount();
   
   //2.返回全部名字
   String[]  names=ac.getBeanDefitionName();
   ```

****



### 二.Spring 通过DI对属性赋值

#### 1.注入赋值

set注入，通过xml标签对进行赋值

​	注：通过set注入一定要有set方法，且set后就是属性名

​			如果只有set方法没有属性名，也可以完成set注入  

​					如只有setName()方法，没有属性 只要在<property> 标签中写name 就可以成功编译

```xml
<bean id="xx" class="yyy.yyy.yy">
	<property name="Java中属性名" value="属性值"/>
 	<property name="Java中属性名" value="属性值"/>
</bean>
```



#### 2.引用类型的set注入

```xml
<bean id ="xx" class ="yyy.yyyy.yy">
	<property name="java中的属性值" ref="bean标签中的id"/>
</bean>
```



#### 3.构造注入

使用<constructor-arg> 标签完成构造注入，构造注入一定要有构造方法

```xml
<bean id = "xx" class ="yyy.yy.yyy" >
	<constructor-arg name="java中构造方法的形参名" value="实参（实际要传的值）"/>
</bean>
```

**<constructor-arg> 中可用的参数**

1. name：java中的形参名
2. index：构造方法中参数的位置  从0,1,2开始
3. value：简单的类型，值
4. ref：引用类型的值



#### 4.引用类型的自动注入

- **byName**  （按名称注入）

  java类中的属性名和Spring 容器中<bean> 中的id名一样，且数据类型相同可以实现自动注入

  ```xml
  <!-- 使用自动注入只要加一个属性 autowire  -->
  <!-- 根据名称自动注入只需在autowire的属性值设置为byName -->
  <bean id ="xxx" class="yy.yyy.yyy" autowire="byName">
  	<property name="简单类型" value ="简单类型值"/>
  </bean>
  
  <!--  这里的ID 名字一定要和上面类的属性名相同  -->
  <bean id="zz" class="xxx.xx.xxx">
      <!--假设这个类是上面类的其中一个类型-->
      <!--简单类型的数据-->
  </bean>
  ```

- **byType** （按类型注入）

  java类中的数据类型和<bean> 中class属性是相同的（同源的）

  同源有三种情况：

  1. java中的引用类型和<bean> 中的class的值一致
  2. java中的引用类型是<bean> 中的class值的**父类**
  3. <bean> 中的class值 是 java中引用类型的**实现**

  **注：使用自动注入一定要保证Spring识别出的类是唯一的**

****



### 三.Spring的多配置文件

#### 1.主配置文件

**主配置文件包含其他的配置文件，一般不会定义对象**

```xml
<beans  ....>
	<import resource="classpath:xx/xx.xml"/>
</beans>
```

classpath表示类路径位置（从class文件下开始找，编译前从resources下开始找）



#### 2.在包含文件的位置可以使用通配符

```xml
<beans ....>
	<inport resource="classpath:xx/*.xml"/>
</beans>
```

**注：**

1. 在使用通配符的时候一定要注意不能将通配符包含的范围包含到主配置文件中
2. 使用通配符一定在同一个文件下（同在resources下不可以）



****

### 四.Spring中使用注解

#### 1.基本步骤

1. 加入依赖
2. 创建类，在类中加入注解
3. 创建Spring 的配置文件，声明组件扫描器的标签，指明注解所在的位置
4. 使用注解创建对象，创建容器ApplicationContext

#### 2.常用的注解

- @Component
- @Repository   **（用于持久层）表示创建dao对象**
- @Service  **用于业务层，可以添加事务管理**
- @Controller  **用于控制器，接收数据显示处理结果**
- @value  **为简单类型赋值**
- @Autowired  **为引用类型赋值，Spring中带的注解**
- @Resource  **为引用类型赋值，JDK中带的注解**
- ......

#### 3.Component注解的使用

**Component用来创建对象等同于<bean>** 

- 属性：value 对象的名称，也就是bean中的id值（唯一的）
- 位置：在类的上面

```java
@Component(value="myStudent")
public class Student{
    private String name;
    private String age;    
}
```

**在类中定义了注解之后，一定要在Spring 的配置文件中添加组件扫描器，不然识别不到注解**

```xml
<beans .....>
    <!-- 一般打一个com就全部出来了 -->
    <context:component-scan base-package="xxx.xxx.xx"/>
</beans>
```



#### 4.使用注解进行属性赋值

##### （1）简单类型  @value

*位置*：

1. 属性上，无需set方法  （**推荐使用**）
2. set方法上

```java
public class Student{
    
    @value(value="张三")
    private String name;
    
    @value(value="12")
    private int age;

    //******************************************************
    
    @value(value="张三") //不推荐
    public void setName(String name){
        this.name=name;
    }
    @value(value="12") //不推荐
    public void setAge(int age){
        this.age=age;
    }
    
}
```



##### （2）引用类型通过注解的方式进行赋值

1. @Autowired  自动注入 **默认的规则是byType**

   - **位置**：

     1. 属性上，**无需set方法**
     2. set方法上

   - byType：**默认类型**

     ```java
     public class Student{
         @value(value="张三")
         private String name;
         
         @value(value="12")
         private int age;
         
         @Autowired
         private School school;
         //Spring 会自动检索容器中国类型为School的类型
     }
     ```

   - byName: 要加上@Qualifier(value="bean中的ID")

     ```java
     public class Student{
         @value(value="张三")
         private String name;
         
         @value(value="12")
         private int age;
         
         //一下两个没有先后顺序
         @Autowired
         @Qualifier(name="bean中的id")
         private School school;
     }
     ```

   - Autowired 的属性

     - required=true ：引用类型赋值失败时，程序报错，并终止程序
     - required=false：引用类型赋值失败时，程序正常执行，引用类型数据为null

   - **注:** byType 默认一个注解，byName 两个注解加Qualifier

2. @Resource   来自JDK中的注解

   支持   byName(***默认***)   byType 

   **位置：**

   - 在属性定义上面，无需set     ***推荐使用***
   - 在set方法上面

   注：Resource   是**先使用byName再使用byType**

   ```java
   //只通过byName方法赋值
   @Resources(name="bean中的id")
   private School school;
   ```

****

### 五.AOP 面向对象编程

#### （1）.怎么理解面向切面编程

1. 需要在分析项目功能时，找出切面
2. 合理安排执行时间（**在方法前，在方法后**）
3. 合理安排切面位置，在哪个类，哪个方法增强

#### （2）术语

1. Aspect  切面：表示增强的功能，为业务功能
2. JoinPoint  连接点：连接业务和切面的位置，（**某类中的业务方法**）
3. PointCut  切入点  : 指连接点方法的集合，多个方法
4. 目标对象：各哪个类的方法增加功能
5. Advice  通知： 通知表示切面功能执行的时机

***一个切面的三要素***

- 切面的功能代码<------>切面要干什么
- 切面的执行位置<------>使用PointCut表示，切面的执行位置
- 切面的执行时间<------>使用Advice 表示执行时间，***在目标方法之前，在目标方法之后***

#### （3）aop的实现

**注：aop是一个规范，动态代理的规范化，是一个标准**

1. Spring 的实现，主要用在事务中使用了aop （**很少使用，比较笨重**）

2. aspectJ：一个开源的专门做aop的框架（最牛逼的框架）

   ***Spring 中集成了aspectJ，通过Spring 来使用aspectJ***

   aspectJ框架实现aop的两种方式

   - 使用xml文件：**配置全局事务**
   - 使用注解（**一般使用注解**）aspectJ有五个常用注解

#### （4）aspectJ的使用

1. 切面的执行时间，规范中叫做Advice（通知，增强）
   1. @Before   前置通知
   2. @AfterReturning
   3. @Around
   4. @AfterThrowing
   5. @After
   
2. 切面执行位置：切入点表达式 execution( *..*.*(..)) 

   表达式中的位置 （访问修饰符<可省>     返回值     包名.类名.方法名(参数列表)     抛出异常<可省>  )

   全通配写法：\*..\*.\*(..) 

   通配符含义：

   -   \*   任意个字符
   -   . .  放到参数中就是任意多个参数，或任意多个包

#### （5）aspectJ 实现aop的基本步骤

1. 新建Maven项目

2. 加入依赖

   - Spring 依赖
   - aspectJ 依赖
   - Junit  单元测试依赖

3. 创建目标类：要有接口和他的实现类，（要做的是就行写，业务逻辑）

4. 创建切面类，普通的一个类（**但是要加注解**）

   - 在类上面加  @Aspect
   - 在类中定义方法，方法就是要执行的切面（**要在目标中扩展的方法**），在方法上加入@Before（前置通知）并指定切入点表达式

5. 创建Spring的配置文件，声明对象，将对象统一交给容器管理

   - 声明目标对象

   - 声明切面类对象

   - 声明aspectJ框架中的自动代理生成器标签，用来完成代理对象的自动创建

     ```xml
     <beans   ...>
         <!--声明aspectJ 框架中的自动代理生成器-->
         <aop:aspectJ-autoproxy/>
         <!--  一般打一个aspect 就出来了-->
     </beans>
     ```

     

6. 创建测试类，从Spring 容器中获取对象，通过geBean之后就是代理类，通过代理类生成代理对象实现aop的功能

#### （6）@Before 前置通知

- 属性  value  值是切入点表达式，表示切面能执行 的位置

- 位置： 方法的上面

- 特点

  1. 在目标执行之前执行
  2. 不改变目标方法执行的结果
  3. 不影响目标方法的执行

  ```java
  @Aspect
  public class myAspect{    
      //这里的方法一定要是实现类中的方法
      @Before(value ="execution(public void xx.xx.类名。方法名(String,..类型))")
      public voic myTestAspect01(){
          System.out.println("这里是要执行的切面方法逻辑")；
      }
  }
  ```

- 注：在@Aspect修饰的类中定义的方法要求

  1. 公共方法 public
  2. 方法没有返回值
  3. 方法的名称自定义
  4. 方法可以有参数，可以没有参数，参数不是自定义的，有几个可以使用
  5. 切入点表达式一定要是实现类中的方法

- ***在测试方法中使用Spring 容器+Aspect事项AOP功能***

  ```java
  public class Test{
      @Test
      public void myTest01(){
          String config="applicationContext.xml";//在resource下的Spring 配置文件
          ApplicationContext cxt=new ClassPathXmlApplicationContext(config);
  
          //这个地方一定要写增强方法的接口类型
          SomeService ss=(SomeService) cxt.getBean("目标方法的ID");
          
          //这个时候使用的方法就是增强过的方法了
          ss.doSome();
      }
  }
  
  ```

  

#### （7）@Before 中可以添加的参数

JoinPoint:业务方法，要加入业务功能的业务方法

作用：可以在通知方法中获取目标方法执行时的信息

​	例：获得方法的实参...

```java
@Aspect
public class {
    @Befor(value="execution(* ..(..))")
    public void myBefore(JoinPoint jp){
        //一定要注意 JoinPoint 两个开头都是大写的
        jp.getSignature();//获得方法的完全定义
        jp.getSignature().getName();//获得方法名称
         Object[] args = jp.getArgs();
    }
}
```



#### （8）@AfterReturning()后置通知

通知方法定义的前提要求：

- 公共方法 public
- 方法没有返回值
- 方法名称自定义
- 方法有参数，推荐使用Object ，参数名称自定义

@AfterReturning

- 属性

  - value：切面表达式
  - returning: 自定义的变量，表示目标方法的返回值，**自定义的名必须和通知方法的形参名一样**

- 位置：方法定义之上

- 特点：

  - 在目标方法之后执行
  - 能获得目标方法的返回值，可以根据值做不同的处理
  - 可以修改返回值

  ```java
  @Aspect
  public class{
  	@AfterReturning(value="execution(void doSome(..))",returning="obj")  
      public void muAfterReturning(Object obj){
          //obj就是方法的返回值，可以对返回值作一些判断
  		System.out.println("方法的返回值是"+obj);
          //更改还要验证
      }
  }
  ```

  

#### （9）@Around环绕通知

通知方法的定义格式

- 公共的方法
- 必须有一个返回值，推荐使用object
- 方法名称自定义
- 方法有参数，固定参数 ProcedingJoinPoint

环绕通知

- 属性value 切入点表达式

- 位置：在方法的上面

- 特点：

  - 功能最强的通知
  - 在目标方法前后都能增强
  - 控制目标方法是否被调用
  - 修改原来目标方法的执行结果，影响调用结果

- 环绕通知等同于JDK动态代理中的InvocationHandler接口

- 参数ProceedingJoinPoint等同于Method方法，作用的执行目标方法

- 返回值：就是目标方法的执行结果，可以被修改

  ```java
  @Aspect
  public class{
  	@Around(value="execution(void doSome(..))")
      public Object myAround(ProceedingJoinPoint pjp){
          //ProceedingJoinPoint 继承了 JoinPoint 
          Object [] args = pjp.getArgs();  //获得全部参数
          //如果目标方法有参数，也是这样写
          Object result =pjp.proceed();
          //可以改变返回的值
          return result;
      }
  }
  ```

  

#### （10）@Pointcut定义和管理切入点

**注：如果项目中有多个切入点是重复的可以使用Pointcut管理切入点**

- 属性：value 切入点表达式

- 位置：在自定义的方法上

- 特点：使用@Pointcut在一个方法上，在个方法名就是切入点表达式的别名，在其他通知中可以使用这个别名代替表达式

  ```java
  @Aspect
  public class{
      @Pointcut(value="execution(void doSome(..))")
      public void mypc(){}
      
      //这个名字一定要有“（）”
      @Before(value="mypc()")
      public void myBefore(){}
  }
  ```

#### （11）AspectJ的默认实现方式

- 如果有接口使用的是JDK

- 如果么有接口使用的是CGLIB

- 有接口使用CGLIB的方式

  

```xml
<aop:aspectj-autopoxy proxy-target-alsss="true"/>
```



### fsfssf





































