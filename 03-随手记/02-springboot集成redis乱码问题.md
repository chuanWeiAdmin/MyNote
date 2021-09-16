### 第一种方式

**设置key和value的**StringRedisSerializer

```java
public void insertResisData() {
    //redisTemplate.setKeySerializer(new StringRedisSerializer());  //key使用StringRedisSerializer
    //redisTemplate.setValueSerializer(new StringRedisSerializer());  //value使用StringRedisSerializer
    redisTemplate.opsForValue().set("name","zs");
    return;
}
```



#### 第二种方式

**添加配置文件**

```java
package com.mytest.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;


@Configuration
public class RedisConfig  {
    @Bean(name="redisTemplate")//名字要和自动注入的名字一样
    //RedisTemplate<String, String> 里面的类型要和自动注入里的类型一样
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, String> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        template.setConnectionFactory(factory);
        //key序列化方式
        template.setKeySerializer(redisSerializer);
        //value序列化
        template.setValueSerializer(redisSerializer);
        //value hashmap序列化
        template.setHashValueSerializer(redisSerializer);
        //key haspmap序列化
        template.setHashKeySerializer(redisSerializer);
        //
        return template;
    }

}
```

