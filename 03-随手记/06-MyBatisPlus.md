#### MyBatisPlus大于等于、小于等于等等函数

使用 QueryWrapper

| 原符号   | <    | <=   | >    | >=   | <>   |
| -------- | ---- | ---- | ---- | ---- | ---- |
| 对应函数 | lt() | le() | gt() | ge() | ne() |

- Mybatis-plus写法：  queryWrapper.ge("create_time", localDateTime);
- Mybatis写法：       where create_time >= #{localDateTime}



#### Mybatis-plus 开启sql日志

```yaml
mybatis-plus:
  configuration:
    #默认不显示SQL日志
    #    log-impl: org.apache.ibatis.logging.nologging.NoLoggingImpl
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

```

