---
title: springboot-redis
date: 2018-09-17 11:14:52
tags: [springboot]
---

## 1.redis大概


### 1.2 Jedis
```
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
	<version>2.7.3</version>
</dependency>
<!-- Springboot自带的redis与redis.clients有冲突，为了使用jodis,选redis.clients -->
<!--<dependency>-->
    <!--<groupId>org.springframework.boot</groupId>-->
    <!--<artifactId>spring-boot-starter-data-redis</artifactId>-->
<!--</dependency>-->
```

通过redis.clients.jedis.JedisPool来管理，即通过池来管理，通过池对象获取jedis实例，然后通过jedis实例直接操作redis服务，剔除了与业务无关的冗余代码
<br/>

配置类
```
package com.example.redisdemo.jedis;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

@Configuration
@PropertySource({ "classpath:application.properties" })
public class JedisConfig {

    @Bean
    public JedisPool getJedisPool(){
        JedisPoolConfig config = new JedisPoolConfig();
        //连接设置
        //config.setMaxTotal(Integer.parseInt(maxTotal));
        //config.set（等等）
        //建立连接
        JedisPool jedisPool = new JedisPool(config, "127.0.0.1", 6379);
        return jedisPool;
    }


}

```

使用
```
package com.example.redisdemo.jedis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.List;

@Service
public class JedisServer {
    @Autowired
    JedisPool jedisPool;

    public  void setKey(String key, Object value){
        //try/catch后面添加
        Jedis jedis = jedisPool.getResource();
         //set(String, String) 所以List需要序列化,此处value为list
        //jedis.set(key.getBytes(), ObjectTranscoder.serialize(value));
        //设置过期时间
        jedis.setex(key.getBytes(),600, ObjectTranscoder.serialize(value));
    }

    public Object getKey(String key){
        Jedis jedis = jedisPool.getResource();
        if(jedis == null || !jedis.exists(key.getBytes())){
            return null;
        }
        byte[] in = jedis.get(key.getBytes());
        List list = (List) ObjectTranscoder.deserialize(in);
        return list;
    }
}

```
### 1.3 Jodis
```
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.7.3</version>
        </dependency>
        <dependency>
            <groupId>com.wandoulabs.jodis</groupId>
            <artifactId>jodis</artifactId>
            <version>0.2.2</version>
        </dependency>
```

与codis结合使用对jedispool做了封装。<br/>
通过zookeeper上注册的codis proxy个数创建相应个数的jedispool封装<br/>为RoundRobinJedisPool，并监听节点的变化，proxy的地址会传回来，可以及时增删配置的jedispool<br/>

配置类
```
package com.example.redisdemo.jodis;

import com.wandoulabs.jodis.JedisResourcePool;
import com.wandoulabs.jodis.RoundRobinJedisPool;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import redis.clients.jedis.JedisPoolConfig;

@Configuration
@PropertySource({ "classpath:application.properties" })
public class JodisConfig {
    @Value("${spring.jodis.zkPath}")
    private String zkPath;
    @Value("${spring.jodis.product}")
    private String product;
    @Value("${spring.jodis.zkPassword}")
    private String zkPassword;
    @Value("${spring.jodis.zkTimeout}")
    private int zkTimeout;
    @Value("${spring.jodis.maxTotal}")
    private String maxTotal;
    @Value("${spring.jodis.maxWaitMillis}")
    private String maxWaitMillis;
    @Value("${spring.jodis.poolMinIdle}")
    private int poolMinIdele;
    @Value("${spring.jodis.poolMaxIdle}")
    private int poolMaxIdle;
    @Value("${spring.jodis.testOnBorrow}")
    private boolean testOnBorrow;
    @Value("${spring.jodis.testOnReturn}")
    private boolean testOnReturn;

    @Bean
    public JedisResourcePool zkJedisPool(){
        JedisResourcePool jedisPool = null;

        try {
            JedisPoolConfig config = new JedisPoolConfig();
            config.setMaxTotal(Integer.parseInt(maxTotal));
            config.setMaxWaitMillis(Integer.parseInt(maxWaitMillis));
            config.setMaxIdle(poolMaxIdle);
            config.setMinIdle(poolMinIdele);
            config.setTestOnBorrow(testOnBorrow);
            config.setTestOnReturn(testOnReturn);
            jedisPool = RoundRobinJedisPool.create().curatorClient(zkPath, zkTimeout)
                    .zkProxyDir(product).password(zkPassword)
                    .poolConfig(config).build();
        }catch (Exception e){
            System.out.println("jodis 连接出错。");
        }
        return jedisPool;
    }

}

```

使用
```
package com.example.redisdemo.jodis;

import com.example.redisdemo.jedis.ObjectTranscoder;
import com.wandoulabs.jodis.JedisResourcePool;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;

import java.util.List;

@Service
public class JodisServer {

    @Autowired
    JedisResourcePool jedisResourcePool;

    public void setKey(String key, Object value, int time){
        Jedis jedis = jedisResourcePool.getResource();
        jedis.setex(key.getBytes(),time, ObjectTranscoder.serialize(value));
    }
    public void setKey(String key, Object value){
        Jedis jedis = jedisResourcePool.getResource();
        System.out.println("ttl: ");
        System.out.println(jedis.ttl(key));
        //long转int,向下转换，可能溢出，不能直接转换，所以long -> String -> int
        jedis.setex(key.getBytes(),Integer.parseInt(String.valueOf(jedis.ttl(key))), ObjectTranscoder.serialize(value));
        //jedis.setex(key.getBytes(),20, ObjectTranscoder.serialize(value));
    }

    public Object getKey(String key){
        Jedis jedis = jedisResourcePool.getResource();
        if(jedis == null || !jedis.exists(key.getBytes())){
            return null;
        }
        byte[] in = jedis.get(key.getBytes());
        List list = (List) ObjectTranscoder.deserialize(in);
        return list;
    }
}

```

## 2.一些注意事项
### 2.1 存储数据
在jedis存储数据过程中
```
public String set(String key, String value) {
  this.checkIsInMultiOrPipeline();
  this.client.set(key, value);
  return this.client.getStatusCodeReply();
}
```
类型为String,String 所以我们要存储List数据，需要先进行序列化<br/>
Serialization（序列化）是一种将对象以一连串的字节描述的过程；反序列化deserialization是一种将这些字节重建成一个对象的过程。<br/>

```
//存
public  void setKey(String key, Object value){
  Jedis jedis = jedisPool.getResource();
  jedis.set(key.getBytes(), ObjectTranscoder.serialize(value));
}
//取
public Object getKey(String key){
    Jedis jedis = jedisPool.getResource();
    if(jedis == null || !jedis.exists(key.getBytes())){
        return null;
    }
    byte[] in = jedis.get(key.getBytes());
    List list = (List) ObjectTranscoder.deserialize(in);
    return list;
}
```
序列化工具类
```
package com.example.redisdemo.jedis;

import java.io.*;

//序列化工具类
public class ObjectTranscoder {
    public static byte[] serialize(Object value) {
        if (value == null) {
            throw new NullPointerException("Can't serialize null");
        }
        byte[] rv=null;
        ByteArrayOutputStream bos = null;
        ObjectOutputStream os = null;
        try {
            bos = new ByteArrayOutputStream();
            os = new ObjectOutputStream(bos);
            os.writeObject(value);
            os.close();
            bos.close();
            rv = bos.toByteArray();
        } catch (IOException e) {
            throw new IllegalArgumentException("Non-serializable object", e);
        } finally {
            try {
                if(os!=null)os.close();
                if(bos!=null)bos.close();
            }catch (Exception e2) {
                e2.printStackTrace();
            }
        }
        return rv;
    }

    public static Object deserialize(byte[] in) {
        Object rv=null;
        ByteArrayInputStream bis = null;
        ObjectInputStream is = null;
        try {
            if(in != null) {
                bis=new ByteArrayInputStream(in);
                is=new ObjectInputStream(bis);
                rv=is.readObject();
                is.close();
                bis.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if(is!=null)is.close();
                if(bis!=null)bis.close();
            } catch (Exception e2) {
                e2.printStackTrace();
            }
        }
        return rv;
    }
}
```

### 2.2 maven包冲突
```
      <!-- redis -->
       <!--<dependency>-->
           <!--<groupId>org.springframework.boot</groupId>-->
           <!--<artifactId>spring-boot-starter-data-redis</artifactId>-->
       <!--</dependency>-->
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>2.7.3</version>
       </dependency>
       <dependency>
           <groupId>com.wandoulabs.jodis</groupId>
           <artifactId>jodis</artifactId>
           <version>0.2.2</version>
       </dependency>
```

springboot自己集成的spring-boot-data-redis与redis-clients有冲突<br/>
redis-clients可以与jodis配合使用，所以选择redis-clients.

### 2.3 关于redis过期时间

redis默认不过期，当内存满时，删除先存入的
```
//存短信记录,首次，设置过期时间
   public void setJodisWithTime(String key, Object value, int time){
       Jedis jedis = jedisResourcePool.getResource();
       //set(String, String) 所以List需要序列化，有效时间为time
       jedis.setex(key.getBytes(),time, FrequencyVerifiy.serialize(value));
   }
   //存短信记录，不重新设置过期时间,使用原有的
   public void setJodis(String key, Object value){
       Jedis jedis = jedisResourcePool.getResource();
       //使用原有时间
       int time = Integer.parseInt(String.valueOf(jedis.ttl(key)));
       jedis.setex(key.getBytes(),time, FrequencyVerifiy.serialize(value));
   }
```


# 一.Springboot集成Redis

- maven依赖

```
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- application.properties

```
spring.redis.host=127.0.0.1
spring.redis.port=6379
```
- 配置类

```
@Configuration
public class RedisConfig  {

    //spring 封装了 RedisTemplate 对象来进行对redis的各种操作，它支持所有的 redis 原生的 api
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory){
        RedisTemplate<String,Object> redisTemplate = new RedisTemplate<>();
        //设置序列化方式,不设置value
        RedisSerializer redisSerializer =new StringRedisSerializer();
        redisTemplate.setKeySerializer(redisSerializer);
        redisTemplate.setHashKeySerializer(redisSerializer);
        //连接
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }
}
```
- 使用

```
@Service
public class RedisDemo {
    @Autowired
    RedisTemplate redisTemplate;

    public  void setKey(String key, Object value){
        ValueOperations<String,Object> vo = redisTemplate.opsForValue();
        vo.set(key, value);
        //设置过期时间
        redisTemplate.expire(key,600,TimeUnit.SECONDS);
    }

    public  Object getKey(String key){
        ValueOperations<String,Object> vo = redisTemplate.opsForValue();
        return vo.get(key);
    }


}
```
