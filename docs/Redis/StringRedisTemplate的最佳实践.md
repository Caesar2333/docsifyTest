# 🚀 Redis 最佳使用指南（基于 `StringRedisTemplate`）

* #### 一切都是基于String的。

我们分类讲解：

------

## ✅ 1. 存 Java 对象（如 `QuestionDTO`）

### ✔ 适合场景：

- 用户登录信息、缓存的试题、一个对象一个键
- 可以单独修改字段（如修改 user 的 name）

### 🔧 推荐存储结构：**Redis Hash**

### ✅ 存入：

```java
QuestionDTO question = new QuestionDTO(...);

Map<String, Object> map = BeanUtil.beanToMap(
    question,
    new HashMap<>(),
    CopyOptions.create()
        .setIgnoreNullValue(true)
        .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString())  // 强转成 String
);
stringRedisTemplate.opsForHash().putAll("question:1", map);
```

### ✅ 读取并还原：

```java
Map<Object, Object> entries = stringRedisTemplate.opsForHash().entries("question:1");
QuestionDTO dto = BeanUtil.fillBeanWithMap(entries, new QuestionDTO(), false);
```













