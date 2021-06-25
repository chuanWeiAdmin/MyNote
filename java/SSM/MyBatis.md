### 一.MyBatis的创建

#### 1.基本步骤

1. 创建maven项目
2. 引入jar包  （mybatis和jdbc驱动）
3. 创建主配置文件
4. 配置mapper映射文件
5. 在方法中创建mybatis的对象

#### 1.引入三个jar包

1. MyBatis的jar包

2. mysql的jar包 （jdbc的驱动jar)

3. log4j的jar包（***在控制台中显示log文件，方便后期的调试***）

   **要在resources下加入properties文件（*和MyBatis同级目录*）**

   log4j的配置文件信息

   ```properties
   # Global logging configuration
   log4j.rootLogger=DEBUG, stdout
   # MyBatis logging configuration...
   log4j.logger.org.mybatis.example.BlogMapper=TRACE
   # Console output...
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
   ```

#### 2.创建主配置文件

**在resources中创建主配置文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://121.4.184.80:3306/MyTestDB"/>
                <property name="username" value="wei"/>
                <property name="password" value="Aa1234567890."/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--<mapper resource="org/mybatis/example/BlogMapper.xml"/>-->
        <!--添加完的XXXmapper文件要在这里注册-->
        <mapper resource="mytest\mapper\MyTestMapper.xml"/>
    </mappers>
</configuration>
```

#### 3.配置mapper映射文件

**这里的映射文件是用来写sql语句的，一般在dao包下创建这个文件，文件名叫xxxdao.xml。**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--这里的namespace是用来区分多个mapper文件的-->
<mapper namespace="test01">
	<!-这里的id是用来区分多个sql语句的，调用的时候通过namespace.id来调用--->
    <!--parameterType:是传入参数的类型-->
    <!--resultType:是最后生成的类型 MyBatis会自动将查询出的数据打包成相应的类型，完整的类名-->
    <select id="getById" parameterType="java.lang.String" resultType="mytest.domain.Student">
        SELECT  * from tbl_Student where id=#{id}
    </select>
</mapper>
```

#### 4.在方法中创建MyBatis

```java
public class Test{
    //这里的主配置文件的路径，因为是放在resources下，所以只用添加文件名（后期可能要改成全类型路径）
    String resource = "mybatis-config.xml";
    InputStream inputStream = null;
    try {
        inputStream = Resources.getResourceAsStream(resource);
    } catch (IOException e) {
        e.printStackTrace();
    }
    SqlSessionFactory sqlSessionFactory = 
        new SqlSessionFactoryBuilder().build(inputStream);

    //通过sqlSessionFactory创建SQLSession对象，对数据库的操作都是通过这个对象来进行的
    SqlSession session= sqlSessionFactory.openSession();

    //因为进行的根据ID查出数据所以只有一条数据，
    //在xml中设置好了所以回返回相应的类型，只有一条用session.selectOne就可以
    //第一个参数 namespace.id  就可以调用到相应的sql语句
    //第二个参数 想要传入的参数
    Student s = session.selectOne("Student.getById","A001");
    System.out.println(s);

    //使用完要将会话关闭
    session.close();
}
```



### 二.创建一个SqlSession工具类

```java
public class sqlSessionUtil{    
    
    private static SqlSessionFactory sqlSessionFactory;
    
    static{
        String resource = "mybatis-config.xml";
        //Resource.getResourcesAsStream-->myBatis中的一个内置的工具类
        InputStream input =Resource.getResourcesAsStream(resource);
        //获取sqlSessionFactory对象
        sqlSessionFactory=new SqlSessionFactoryBuild().build(input);
    }
    
    //多个类使用sqlsession对象保证是同一个
    private static ThreadLocal <SqlSession> t =new ThreadLocal<>();
    
    //获得session对象   注：为static是工具类通过类名去调
    public static SqlSession getSession(){
        //首先从ThreadLocal 获取 如果没有在通过sqlSessionFactory创建
        SqlSession session = t.get();
        if(session==null){
            session=sqlSessionFactory.openSession();
			//创建完了要放回去
            t.set();
        }
        
        return session;
    }
    
    //关闭SqlSession对象
    public static myClose(SqlSession session){
        if(session!=null){
            session.close();
            
            //极其重要，让ThreadLocal清空要不然后面会有问题
            t.remove();
        }
        
    }
    
}
```

**注：**

- 提交事务的操作在service层（M）完成
- 工具类中如果不想让人实例化对象（会产生垃圾代码）可以将构造方法私有化

```java
private sqlSessionUtil() {}
```



### 
