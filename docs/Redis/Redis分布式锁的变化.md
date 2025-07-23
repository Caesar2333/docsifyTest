# Redis分布式的变化历史

# Redis分布式锁演进过程详解

## 目录

1. [分布式锁的基本需求](#分布式锁的基本需求)
2. [第一阶段：SETNX的简单实现](#第一阶段setnx的简单实现)
3. [第二阶段：加入过期时间](#第二阶段加入过期时间)
4. [第三阶段：原子性操作](#第三阶段原子性操作)
5. [第四阶段：Lua脚本保证原子性](#第四阶段lua脚本保证原子性)
6. [第五阶段：看门狗机制](#第五阶段看门狗机制)
7. [第六阶段：可重入锁](#第六阶段可重入锁)
8. [第七阶段：红锁算法](#第七阶段红锁算法)
9. [常见问题总结](#常见问题总结)

## 分布式锁的基本需求

### 核心要求

- **互斥性**: 任意时刻只有一个客户端能持有锁
- **安全性**: 只有持有锁的客户端才能释放锁
- **活性**: 不会发生死锁，最终一定能获取到锁
- **容错性**: 部分节点宕机后，锁服务依然可用

## 第一阶段：SETNX的简单实现

### 实现方式

[复制代码](#)

```java
SETNX lock_key ``"client_id"
```

### 代码示例

[复制代码](#)

```java
public boolean tryLock(String lockKey, String clientId) {
    // 尝试设置锁
    String result = jedis.set(lockKey, clientId, "NX");
    return "OK".equals(result);
}

public void unlock(String lockKey, String clientId) {
    // 简单删除锁
    jedis.del(lockKey);
}

```

### 存在的问题

#### 1. 死锁问题

[复制代码](#)

```
场景：客户端获取锁后宕机，锁永远不会被释放
问题：其他客户端永远无法获取锁
影响：系统完全阻塞

```

#### 2. 误删锁问题

[复制代码](#)

```java
// 客户端A获取锁
SETNX lock_key "client_A"

// 客户端A业务执行时间过长
// 客户端B在客户端A还在执行时删除了锁
DEL lock_key

// 客户端C获取到锁，与客户端A同时执行
SETNX lock_key "client_C"

```

## 第二阶段：加入过期时间

### 实现方式

[复制代码](#)

```java
SETNX lock_key ``"client_id"``EXPIRE lock_key ``30
```

### 代码示例

[复制代码](#)

```java
public boolean tryLock(String lockKey, String clientId, int expireSeconds) {
    // 设置锁
    String result = jedis.set(lockKey, clientId, "NX");
    if ("OK".equals(result)) {
        // 设置过期时间
        jedis.expire(lockKey, expireSeconds);
        return true;
    }
    return false;
}

```

### 存在的问题

#### 1. 非原子性操作

[复制代码](#)

```
时序问题：
1. SETNX 成功
2. 客户端宕机（EXPIRE未执行）
3. 锁永远不会过期
结果：死锁问题依然存在

```

#### 2. 过期时间设置困难

[复制代码](#)

```
问题：
- 设置太短：业务未完成锁就过期
- 设置太长：异常情况下锁释放慢
- 业务执行时间不确定

```

## 第三阶段：原子性操作

### 实现方式

[复制代码](#)

```java
SET lock_key ``"client_id"` `NX EX ``30
```

### 代码示例

[复制代码](#)

```java
public boolean tryLock(String lockKey, String clientId, int expireSeconds) {
    // 原子性设置锁和过期时间
    String result = jedis.set(lockKey, clientId, "NX", "EX", expireSeconds);
    return "OK".equals(result);
}

public void unlock(String lockKey, String clientId) {
    // 检查是否是自己的锁
    String value = jedis.get(lockKey);
    if (clientId.equals(value)) {
        jedis.del(lockKey);
    }
}

```

### 存在的问题

#### 1. 锁误删问题

[复制代码](#)

```java
// 时序问题
String value = jedis.get(lockKey);        // 1. 获取锁值
if (clientId.equals(value)) {             // 2. 判断是自己的锁
    // 此时锁可能已经过期，被其他客户端获取
    jedis.del(lockKey);                   // 3. 删除锁（可能删除了别人的锁）
}

```

#### 2. 锁过期问题

[复制代码](#)

```java
场景：业务执行时间 > 锁过期时间
问题：锁在业务执行过程中过期，其他客户端获取到锁
结果：多个客户端同时执行临界区代码

```



### 为什么我们在使用 `SETNX` 加锁时，**可能误删别人的锁**？

### 🧨 问题根源：**检查锁和删除锁不是原子操作**

> 换句话说，**你以为是你加的锁，其实锁已经过期或被别人抢了，而你还在释放它**。

### ⚠️ 问题场景重现（竞态条件）

1. 线程 A 执行完 `GET my_lock == "UUID-A"`，准备删除。
2. 这时锁恰好**过期了**。
3. 线程 B 设置了新的锁，value 是 `"UUID-B"`。
4. **线程 A 继续执行 `DEL my_lock`，把线程 B 的锁误删了。**

这样，**线程 A 把自己已经“失效”的锁给删了，而这个锁其实是别人新设置的！**

------

### ✅ 正确做法：使用 Lua 脚本保证原子性

你要确保 **“判断 + 删除”是原子的**，这就需要 Redis 提供的 Lua 脚本机制：



## 第四阶段：Lua脚本保证原子性

### 释放锁的Lua脚本

[复制代码](#)

```java
-- 释放锁的Lua脚本
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end

```

### 代码示例

[复制代码](#)

```java
public class DistributedLock {
    private static final String UNLOCK_SCRIPT = 
        "if redis.call('get', KEYS[1]) == ARGV[1] then " +
        "    return redis.call('del', KEYS[1]) " +
        "else " +
        "    return 0 " +
        "end";
    
    public boolean tryLock(String lockKey, String clientId, int expireSeconds) {
        String result = jedis.set(lockKey, clientId, "NX", "EX", expireSeconds);
        return "OK".equals(result);
    }
    
    public boolean unlock(String lockKey, String clientId) {
        Object result = jedis.eval(UNLOCK_SCRIPT, 
                                  Collections.singletonList(lockKey), 
                                  Collections.singletonList(clientId));
        return "1".equals(result.toString());
    }
}

```

### 解决的问题

- ✅ 原子性释放锁
- ✅ 避免误删其他客户端的锁

### 仍存在的问题

#### 1. 锁续期问题

[复制代码](#)

```
场景：业务执行时间不确定
问题：
- 锁过期时间设置困难
- 业务执行中锁过期导致并发问题
- 无法动态调整锁的持有时间

```

## 第五阶段：看门狗机制

### Redisson的看门狗实现原理

#### 1. 自动续期机制

[复制代码](#)

```java
// Redisson内部实现原理
public class RedissonLock {
    private void scheduleExpirationRenewal(long threadId) {
        ExpirationEntry entry = new ExpirationEntry();
        entry.addThreadId(threadId);
        
        // 每10秒续期一次（默认30秒过期时间的1/3）
        Timeout task = commandExecutor.getConnectionManager()
            .newTimeout(new TimerTask() {
                @Override
                public void run(Timeout timeout) {
                    // 续期Lua脚本
                    renewExpiration();
                }
            }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
    }
}

```

#### 2. 续期Lua脚本

[复制代码](#)

```java
-- 续期脚本
if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then 
    return redis.call('pexpire', KEYS[1], ARGV[1])
else 
    return 0
end

```



### 看门狗的机制设计以及为什么也要lua脚本续期

#### 二、🔐 看门狗机制的设计本质

Redisson 的做法是：

#### ✅ 为每个成功加锁的线程，**启动一个“看门狗”任务**：

- 定时每 **internalLockLeaseTime / 3** 秒（默认 10 秒）尝试**续期**
- 把锁的过期时间重新设置为默认的 `30 秒`
- 相当于「你活着，就续租；你死了，锁就自然失效」

------

#### 三、🧠 为什么 Redisson 的看门狗需要用 Lua 脚本？

##### 这是你提问中最核心的点。

#### 📌 原因是：**续期的“判断 + 设置过期”也需要是原子的！**





### 看门狗机制特点

[复制代码](#)

```
优势：
✅ 自动续期，避免业务执行中锁过期
✅ 客户端宕机时，看门狗停止，锁自然过期
✅ 无需预估业务执行时间

工作原理：
1. 获取锁时启动看门狗
2. 定时检查锁是否还被当前线程持有
3. 如果持有，则续期锁的过期时间
4. 释放锁或客户端宕机时，看门狗停止

```

### 🔥 场景复现（非原子操作会出问题）：

```java
if (hexists("myLock", "UUID-A")) {
    pexpire("myLock", 30000); // 续期30秒
}
```

执行到一半：

1. `hexists("myLock", "UUID-A")` == true ✅
2. **你刚判断完，就挂了（线程挂了/GC 卡顿/网络延迟）**
3. 此时 Redis 中的 `myLock` 过期并被删除 🧨
4. 线程 B 获得了 `myLock`，它用的是 `"UUID-B"` 设置的新锁
5. 你恢复了，继续执行 `pexpire("myLock", 30000)`
6. ❌ **你把线程 B 的锁给续期了！**

结果：

- 你自己不再拥有锁
- 却给别人续了锁
- **造成了“锁误续”，影响了别人的锁释放计划，严重破坏分布式锁语义**



### 使用了看门狗就不用自动续期了吗？

#### ✅ 正确答案：**用了看门狗后，你“\**不需要自己续期\**”，Redisson 会帮你续**

#### 🔁 Redisson 看门狗机制是用来**自动续期**的！

它的设计目标就是为了这件事：

- 如果你没有设置 `leaseTime`，Redisson 自动认为你“业务执行时间不确定”
- 那么它会自动每隔 `10 秒` 续一次期，默认续成 `30 秒`
- 只要你还持有锁，Redisson 就继续给你续，不会过期
- 直到你 `unlock()`，才会停止续期任务并释放锁

### 看门狗续期机制

* #### 没有自动设置续期的话，那就自动启动

* #### 如果设置了续期的话，就不启动。



### 代码示例

[复制代码](#)

```java
// Redisson使用示例
public void businessLogic() {
    RLock lock = redissonClient.getLock("myLock");
    try {
        // 获取锁，启动看门狗
        boolean acquired = lock.tryLock(3, TimeUnit.SECONDS);
        if (acquired) {
            // 执行业务逻辑，无需担心锁过期
            doBusinessLogic();
        }
    } finally {
        // 释放锁，停止看门狗
        if (lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}

```

## 第六阶段：可重入锁

### 可重入锁的需求

[复制代码](#)

```java
public void methodA() {
    lock.lock();
    try {
        methodB(); // 需要再次获取同一把锁
    } finally {
        lock.unlock();
    }
}

public void methodB() {
    lock.lock(); // 同一线程再次获取锁
    try {
        // 业务逻辑
    } finally {
        lock.unlock();
    }
}

```

### Redisson可重入锁实现

#### 1. 数据结构

[复制代码](#)

```java
# 使用Hash结构存储锁信息
HSET lock_key thread_id count
# 例如：
HSET myLock "thread_123" 2  # 线程123持有锁，重入次数为2

```

#### 2. 获取锁的Lua脚本

[复制代码](#)

```java
-- 可重入锁获取脚本
if (redis.call('exists', KEYS[1]) == 0) then 
    redis.call('hset', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return nil
end

if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then 
    redis.call('hincrby', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return nil
end

return redis.call('pttl', KEYS[1])

```

#### 3. 释放锁的Lua脚本

[复制代码](#)

```java
-- 可重入锁释放脚本
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then 
    return nil
end

local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1)
if (counter > 0) then 
    redis.call('pexpire', KEYS[1], ARGV[2])
    return 0
else 
    redis.call('del', KEYS[1])
    return 1
end

```

## lua脚本的问题——原子性

| 操作       | 目的是啥？                     | 如何做？                                       |
| ---------- | ------------------------------ | ---------------------------------------------- |
| **释放锁** | 如果锁是我的，我来释放它       | 判断锁是不是我的 → 是就删除（DEL）             |
| **续期锁** | 如果锁是我的，我来延长它的寿命 | 判断锁是不是我的 → 是就更新过期时间（PEXPIRE） |

* #### 让检查和操作是原子操作。

* #### 防止检查完了之后，将要操作的时候，因为某种原因停止了，锁过期了，其他人抢到了，那线程又活了，对不属于自己的锁进行了操作，比如释放或者是续期。

## 常见问题总结

### 1. 死锁问题

[复制代码](#)

```java
原因：锁没有过期时间或客户端宕机
解决：设置合理的过期时间 + 看门狗机制
```

### 2. 锁误删问题

[复制代码](#)

```java
原因：删除锁时没有验证锁的所有者
解决：使用Lua脚本原子性验证和删除
```

### 3. 锁过期问题

[复制代码](#)

```java
原因：业务执行时间超过锁的过期时间
解决：看门狗机制自动续期
```

### 4. 可重入问题

[复制代码](#)

```java
原因：同一线程无法多次获取同一把锁
解决：使用Hash结构记录重入次数
```

### 5. 主从切换问题

[复制代码](#)

```java
原因：同一线程无法多次获取同一把锁
解决：使用Hash结构记录重入次数
```

### 6. 性能问题

[复制代码](#)

```java
原因：频繁的网络交互和Lua脚本执行
优化：
- 合理设置锁粒度
- 使用连接池
- 批量操作
- 监控锁的获取成功率和耗时
```

### 7. 时钟偏移问题

[复制代码](#)

```java
原因：不同服务器时钟不同步
影响：锁的过期时间计算不准确
解决：使用NTP同步时钟，或使用相对时间
```

## 最佳实践建议

### 1. 锁设计原则

- 锁粒度要合适，避免粗粒度锁
- 锁的key要有业务含义
- 设置合理的超时时间

### 2. 异常处理

- 必须在finally块中释放锁
- 处理获取锁超时的情况
- 记录详细的日志便于排查

### 3. 监控告警

- 监控锁的获取成功率
- 监控锁的持有时间
- 监控Redis的性能指标

### 4. 降级策略

- Redis不可用时的降级方案
- 锁获取失败时的业务处理
- 本地锁作为备选方案



# Redis传统锁和Redisson的锁

## 🚀 一步一步来：

### ✅ 传统 Redis 锁做法：

```
SET myLock UUID-A NX EX 30
```

- 是一个简单的字符串结构
- 释放锁时用 `GET myLock` 看是否等于 UUID-A
- 非结构化，不支持可重入，也不支持多线程扩展

### 🔒 Redisson 的锁做法（核心差异）

Redisson 不用 `SET`，而是：

```
HSET myLock UUID-A 1
PEXPIRE myLock 30000
```

你注意到了没？

> Redisson 的锁结构，是 Hash 类型：
>  键：`myLock`
>  Field：`UUID-A`（即 threadId）
>  Value：`1`（代表重入次数）

------

### 🧠 所以为什么用 `HEXISTS` 而不用 `GET`？

- `GET` 是针对 **字符串键值** 的操作
- `HEXISTS` 是针对 **Hash 的 Field 存在性判断** 的操作

## 👇 举个真实例子，你就秒懂：

### 假设我们有一个锁叫 `order:lock:123`

#### 传统方式：

```java
SET order:lock:123 UUID-A NX EX 30
GET order:lock:123  --> UUID-A
```

#### Redisson 方式：

```java
HSET order:lock:123 UUID-A 1       // UUID-A 是线程 ID
PEXPIRE order:lock:123 30000       // 设置整个 Hash 的 TTL
HEXISTS order:lock:123 UUID-A      // 判断这个线程是否持有这个锁
```

## 🔍 Redisson 为什么设计成 Hash？

| 目的             | 原因                                        |
| ---------------- | ------------------------------------------- |
| ✅ 可重入锁       | 用 field-value 来存储线程 ID 和重入次数     |
| ✅ 多线程竞争支持 | Hash 中可以区分不同线程的锁拥有情况         |
| ✅ 精准续期       | 用 `HEXISTS` 判断是否还是自己持有           |
| ❌ 不能用 GET     | 因为 `GET` 根本不能操作 Hash 类型！会报错！ |

## 🧠 总结精华：

| 操作                | 用于哪种锁结构？    | 操作类型                 | 用途                       |
| ------------------- | ------------------- | ------------------------ | -------------------------- |
| `GET key`           | 字符串锁（SETNX）   | 字符串                   | 判断是否自己持有锁         |
| `HEXISTS key field` | Hash 锁（Redisson） | Hash                     | 判断 threadId 是否仍拥有锁 |
| `PEXPIRE key time`  | 两者都能用          | 通用                     | 设置 key 的过期时间        |
| `DEL key`           | 字符串锁            | 删除 key                 |                            |
| `HDEL key field`    | Hash 锁             | 删除 threadId 所持锁字段 |                            |



# 锁的扩展策略

## 🧩 一、锁的核心争议点：拿不到怎么办？

分布式环境中，锁**可能正在被别人占用**。这时你的程序要不要一直等？怎么等？

常见的扩展策略有两种：

| 策略类型             | 说明                                             | 代表框架/产品                      |
| -------------------- | ------------------------------------------------ | ---------------------------------- |
| ✅ 基于**时间**的重试 | 在给定时间内，**不断尝试获取锁**，直到拿到或超时 | Redisson 默认                      |
| ✅ 基于**次数**的重试 | 最多尝试 `N` 次，每次间隔 `delay` 毫秒           | Sentinel、ZooKeeper、定制 Redisson |
| 🚫 不等待直接失败     | 只尝试一次                                       | Redis 的 `SETNX` 单独用法          |



## 🚦 二、Redisson 是哪种？默认策略如何？

Redisson 默认走的是**时间+自旋**机制：

```java
boolean tryLock(long waitTime, TimeUnit unit)
```

这个方法意味着：

- 最长等 `waitTime` 秒；
- 在这段时间内，它会采用 **间隔性重试 + 可中断性等待机制** 来不断尝试去抢锁。



## 🔧 四、你要的锁扩展能力有哪些？

为你整理下，**一个成熟的锁框架应该支持以下扩展策略**，你在实现或选型时可以直接按这个 checklist 看：

| 扩展能力           | 说明                           | Redisson 是否支持        |
| ------------------ | ------------------------------ | ------------------------ |
| ⏳ 获取等待时间     | 最长等待多长时间抢锁           | ✅ 支持                   |
| 🔁 重试次数限制     | 最多尝试 N 次                  | ❌ 默认不支持（需要封装） |
| 🧠 重试间隔策略     | 固定间隔 / 指数退避 / 随机抖动 | ❌（需自定义封装）        |
| 🕐 锁 TTL           | 获取成功后自动释放时间         | ✅ 支持                   |
| 🐶 看门狗续期       | 自动延长锁 TTL                 | ✅ 支持（默认30s续期）    |
| ❌ 获取失败快速失败 | 尝试一次拿不到就放弃           | ✅ 通过 `tryLock()` 实现  |
| ☠️ 强制释放         | 超时/重启后强制释放            | ✅ 由 Redis key TTL 实现  |
| 🌐 可重入           | 当前线程重复加锁不死锁         | ✅ 支持                   |
| 🔒 公平锁           | 排队获取锁，谁先等谁先拿       | ✅ `getFairLock()` 支持   |
| 📦 锁元数据记录     | 锁被谁加的、多久了等           | ❌ 默认无（可拓展）       |

## ✅ 最后一句话总结你当前的疑惑

> **Redisson 的 `tryLock(waitTime)` 是“时间驱动”的重试方式：在 `waitTime` 时间内反复尝试获取锁，不是一次；**
>  而“重试次数驱动”的是另一种扩展方式，适合你需要控制冲突频率、快速失败、或轻量任务时手动实现。

