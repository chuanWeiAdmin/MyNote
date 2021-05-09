##### 1.mybatis 错误提示：is not known to the MapperRegistry.

没有找到映射文件，

解决方法

1. 查看mapper.xml的**namespace**是不是对应的DAO全路径。
2. 查看是不是这个mapper没有添加到mybatis-config.xml文件中