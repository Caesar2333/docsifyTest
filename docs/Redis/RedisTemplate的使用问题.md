# RedisTemplate vs StringRedisTemplate 本质总结

## ✅ RedisTemplate vs StringRedisTemplate 本质总结（高质量笔记版）

### 一、Redis 的底层本质

- **Redis 是字节数组数据库**：无论你存什么类型的数据（String、List、Hash、ZSet），本质上都是转成 `byte[]` 存储的。
- 所以：Java 中的任何数据类型 → 都需要经过“序列化器”转成 `byte[]` 才能写入 Redis。

------

### 二、两个模板类的设计区别

| 对比项           | StringRedisTemplate                      | RedisTemplate（需手动配置序列化器）                     |
| ---------------- | ---------------------------------------- | ------------------------------------------------------- |
| 目标类型         | 只支持 Java 的 `String` 类型（操作简单） | 支持任意 Java 对象（Object）                            |
| 默认序列化器     | `StringRedisSerializer`                  | 默认 `JdkSerializationRedisSerializer`，一般会换成 JSON |
| 可操作的数据结构 | ✅ 支持 `opsForValue/Hash/List/ZSet/...`  | ✅ 一样支持所有结构                                      |
| 使用场景         | Token、验证码、纯字符串映射              | 缓存 Java 对象（DTO/VO/实体类）、结构化对象列表等       |
| 缺点             | 只能操作 String，不能自动处理对象        | 如果不配序列化器，默认存成二进制，不可读                |



### 三、典型场景比较

| 场景                     | 使用建议                | 原因说明                                                     |
| ------------------------ | ----------------------- | ------------------------------------------------------------ |
| 存储验证码 `"1234"`      | ✅ `StringRedisTemplate` | Java String → UTF-8 → `byte[]` 存入，无需额外配置            |
| 存储用户对象 `UserDto`   | ✅ `RedisTemplate`       | 需要将 Java 对象 → JSON 字符串 → UTF-8 → `byte[]`，需 Jackson 配合序列化器 |
| 存储 `List<String>`      | ✅ `StringRedisTemplate` | List 中都是 String，配合 `opsForList()` 写入即可             |
| 存储 `List<QuestionDto>` | ✅ `RedisTemplate`       | 复杂结构，需自动 JSON 化，不能手动 `.toString()`（那只是内存地址），必须自动序列化 |



### 四、为何很多人搞混了？

> 原因是混淆了这几个概念：

- Java 中的 `"String"` 类型 vs 广义上的“字符串”
- Redis 里看到的是 JSON 字符串，但底层其实是 `byte[]`
- 所有写进 Redis 的内容都必须是 `byte[]`，关键在于：**你用什么手段把它转成 `byte[]`**

------

### 五、总结金句（建议背熟）

> **Redis 不关心你是什么对象，它只关心你给的 `byte[]` 是不是能被写入。**
>  `StringRedisTemplate` 只负责 `String → byte[]`，适合操作纯文本；
>  `RedisTemplate` 配合 JSON 序列化器后，能自动处理 Java 中任意对象，是更通用的方案。





# jackson序列化的本质

* #### 只是将`object`的java对象，转换成了`String`的java对象，

  * #### 之后还需要经过`str.getBytes[]`来转化成字节流的。

* 



# Json本质 他就是一个字符串

* #### 其在Java中是需要使用`""`给其固定起来的。

* #### 而在别的地方，我们说json，其实就是在说字符串的。





# RedisTemplate和StringRedisTemplate的默认序列化器

- #### `StringRedisTemplate` 是 Spring Boot 自带的 Bean，内部已经写死了它使用的序列化器👇：

```java
this.setKeySerializer(new StringRedisSerializer());
this.setValueSerializer(new StringRedisSerializer());
this.setHashKeySerializer(new StringRedisSerializer());
this.setHashValueSerializer(new StringRedisSerializer());
```

```java
public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        this.setKeySerializer(RedisSerializer.string());
        this.setValueSerializer(RedisSerializer.string());
        this.setHashKeySerializer(RedisSerializer.string());
        this.setHashValueSerializer(RedisSerializer.string());
    }

    public StringRedisTemplate(RedisConnectionFactory connectionFactory) {
        this();
        this.setConnectionFactory(connectionFactory);
        this.afterPropertiesSet();
    }

    protected RedisConnection preProcessConnection(RedisConnection connection, boolean existingConnection) {
        return new DefaultStringRedisConnection(connection);
    }
}
```



## RedisTemplate 默认的defaultSerializer是jdk序列化的，很慢

```java
if (this.defaultSerializer == null) {
            this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
        }
```

```java
/*
         注：当前配置类不是必须的，因为 Spring Boot 框架会自动装配 RedisTemplate 对象，
          但是默认的key序列化器和value序列化器都是JdkSerializationRedisSerializer，
          默认序列化器导致存到Redis中的数据和原始数据有差别（可读性差），
          故设置为需要修改序列化器。
          这里使用StringRedisSerializer作为key的序列化器，转为String存储。
          使用GenericJackson2JsonRedisSerializer作为value的序列化器，转为json对象存储。
         */
        //设置redis key的序列化器
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        //设置redis value的序列化器
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
```

















