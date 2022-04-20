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



#### Mybatis 中使用模糊查询

```mysql
--方式1：这种方式，简单，但是无法防止SQL注入，所以不推荐使用
LIKE  '%${name}%'

-- 方式2：#
LIKE "%"#{name}"%"

-- 方式3：字符串拼接
AND name LIKE CONCAT(CONCAT('%',#{name},'%'))
```

