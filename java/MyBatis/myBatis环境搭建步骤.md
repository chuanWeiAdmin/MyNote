### 搭建MyBatis要放三个文件



### 第一步：引入三个jar包

1. MyBatis的jar包

2. mysql的jar包

3. log4j的jar包（***在控制台中显示log文件，方便后期的调试***）

   - 要在sec下加入properties文件（***和MyBatis同级目录***）

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

     



### 第二步：创建主配置文件

- 一般在src下创建文件名为mybatis-config.xml的配置文件

- ```xml
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



### 第三步：配置mapper映射文件

**这里的映射文件是用来写sql语句的，一般在mapper包下创建这个文件，文件名叫xxxMapper.xml。**



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



### 第三步：在方法中调用

```java
//这里的主配置文件的路径，因为是放在src下，所以只用添加文件名（后期可能要改成全类型路径）
String resource = "mybatis-config.xml";
InputStream inputStream = null;
try {
    inputStream = Resources.getResourceAsStream(resource);
} catch (IOException e) {
    e.printStackTrace();
}
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

//通过sqlSessionFactory创建SQLSession对象，对数据库的操作都是通过这个对象来进行的
SqlSession session= sqlSessionFactory.openSession();

//因为进行的根据ID查出数据所以只有一条数据，在xml中设置好了所以回返回相应的类型，只有一条用session.selectOne就可以
//第一个参数 namespace.id  就可以调用到相应的sql语句
//第二个参数 想要传入的参数
Student s = session.selectOne("Student.getById","A001");
System.out.println(s);

//使用完要将会话关闭
session.close();
```

