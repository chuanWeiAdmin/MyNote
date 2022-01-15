#### QueryWrapper 函数

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



#### Mybatis Sql中一些替换写法

- 第一种写法（1）：

  | 原符号   | <     | <=     | >     | >=     | <>         | &      | '       | "       |
  | -------- | ----- | ------ | ----- | ------ | ---------- | ------ | ------- | ------- |
  | 对应函数 | \&lt; | \&lt;= | \&gt; | \&gt;= | \&lt;\&gt; | \&amp; | \&apos; | \&quot; |

- 第二种写法（2）

  - 大于等于  <![CDATA[ >= ]]>
  -  小于等于   <![CDATA[ <= ]]>

