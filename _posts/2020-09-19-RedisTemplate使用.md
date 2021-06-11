---
layout: post
title:  "RedisTemplate使用"
imges: 
date:   2020-09-19 10:02:00 +0800
description: 
tags: java Redis
---

### 在这里笔者新建的是springboot项目测试

> 1.导入依赖

```
 <!--redis的快速开发依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

>2.配置文件application.properties添加IP和端口

```
#配置redis,做win本地测试，根据数据修改地址和端口号
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

> 3.编写RedisConfig.java文件，在项目中直接使用就行了，注意修改包名

```
package com.zeng.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {
    //编写自己的redisTemplate,这个是一个模板
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> getredisTemplate(RedisConnectionFactory factory){
        //为了开发方便 项目使用<string，Object>
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(factory);

        //Json序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //配置具体的序列化方式
        ObjectMapper om  = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        //string的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //key采用的string的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        //hash的key也采用string的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        //value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的value序列化方式也采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }
}
```

> 4.新建pojo

```
@Component
/*@AllArgsConstructor //生成全参数构造函数
@NoArgsConstructor //生成无参构造函数
@Data //生成getter,setter等函数*/
public class User implements Serializable {
    
    private String name;
    private int age;
    public User(){}
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return "{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

> 5.编写测试类

**要注意的是，在使用redistemplate存数据，项目中用的比较多的是<String,Object>，值一般多使用json字符串格式，下面测试的时候，存的value是一个user对象，因为redistemplate默认使用jdk的方式存储数据，如果不编写自定义的redistemplate，则会出现很多问题，还有序列化的问题，在项目中，直接使用上面的redisconfig配置类就行**
```
class Redis02SpringbootApplicationTests {
    @Autowired
    @Qualifier("getredisTemplate")
    private RedisTemplate redisTemplate;
    
    @Test
    public void test()throws Exception{
        //清空数据库
        redisTemplate.getConnectionFactory().getConnection().flushDb();
        User user = new User("yiming",22);
        //String jsonasuser = new ObjectMapper().writeValueAsString(user);
        redisTemplate.opsForValue().set("userinfo:"+"user",user);
        System.out.println(redisTemplate.opsForValue().get("userinfo:"+"user"));
    }
}
```

>6.测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828210310916.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828210321979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjY4MjgwMA==,size_16,color_FFFFFF,t_70#pic_center)
