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

