# dp的两种解法

## ✅ 你这句话说出了本质真理：

> **“递归 + 记忆数组 = f 数组”**
>  🟰 **记忆化搜索（Top-Down） = 动态规划（Bottom-Up）**

它们是**同一个问题**的两种解法方式：

------

### ✅ 从递归的角度看：

你写一个递归函数 `dfs(i)` 表示到第 i 个房子为止的最大金额：

```
def dfs(i):
    if i < 0: return 0
    return max(dfs(i-1), dfs(i-2) + nums[i])
```

这叫做：**“自顶向下”分解问题**，但是会有大量重复调用 → 所以你加上 `memo[i]` 缓存。

------

### ✅ 从动态规划的角度看：

你直接建立一个数组 `f[i]` 表示第 i 个位置的最优解：

```
f[i] = max(f[i-1], f[i-2] + nums[i])
```

这是：**“自底向上”构建答案**，中间每一个状态都用数组 `f` 保存了。

------

### ✅ 核心共识是：

> DP 的本质就是：**重叠子问题 + 最优子结构 + 缓存中间状态**
>  而你用“缓存中间状态”的方式，不管是数组 `f` 还是 `memo[]`，都是一样的逻辑。

------

## 🔄 最强心法：记忆化搜索 vs 递推 DP 转换图

| 点         | 记忆化搜索（递归写法）  | 递推 DP（for写法）     |
| ---------- | ----------------------- | ---------------------- |
| 思维方式   | 自顶向下                | 自底向上               |
| 写法       | 有返回值的递归 + memo[] | 明确的数组 f[]         |
| 是否剪枝   | 需要手动剪枝            | 默认不需要             |
| 是否容易调 | 简单直观，递归易调试    | 写起来偏死板，但更高效 |
| 是否需要栈 | 会用到调用栈（易爆栈）  | 不用，for循环稳定      |

你现在看这个图，就知道为什么：

### ✅ “`f` 数组”其实就是：“手动控制的记忆栈帧 + 缓存值”。

> 你把操作系统的**递归栈+memo缓存**手动摊平到了数组上，自己来搞状态转换！

这就是为什么 DP 被叫做：

> 「**递归的显式模拟 + 最优子结构记忆表**」





# 二者的时间复杂度怎么样？

## ✅ 一、时间复杂度分析：**没有发生优化**

无论是记忆化搜索还是递推 DP，**都保证每个子状态只计算一次**：

### 🔁 递归 + 缓存（记忆化搜索）：

```python
def dfs(i):
    if memo[i] != -1: return memo[i]
    memo[i] = max(dfs(i-1), dfs(i-2) + nums[i])
    return memo[i]
```

- 每个 `dfs(i)` 只执行一次，之后走的是缓存
- 总共调用 ≈ `O(n)` 次

### 🔁 for 循环 DP：

```python
for i in range(n):
    f[i+2] = max(f[i+1], f[i] + nums[i])
```

- 明确循环一次计算一格
- 总共也就是 `O(n)` 步

✅ **结论：时间复杂度都是 O(n)**

## ✅ 二、空间复杂度分析：发生了明显优化 ✅

我们分为两部分来看：

------

### 🔴 1. 显式空间：数组部分（memo 或 f）

| 写法             | 空间                   |
| ---------------- | ---------------------- |
| 递归 + memo      | `O(n)`（memo[n] 数组） |
| for 循环 + f数组 | `O(n)`（f[n] 数组）    |

在这个方面：**没有本质优化**

------

### ✅ 2. 隐式空间：递归调用栈

| 写法             | 栈空间                 |
| ---------------- | ---------------------- |
| 递归 + memo      | `O(n)` 最深调用栈深度  |
| for 循环 + f数组 | `O(1)`（无递归，无栈） |

✅ **优化点就在这里：彻底消除函数调用栈空间**

* #### 由于这里的空间复杂度是相加的。所以总的来说，二者时间复杂度和空间复杂度都是`O(n)`

## 🔍 所以你的猜想是完全正确的：

> ✅ ✅ ✅ 「**从递归 + 缓存 → DP递推**，**时间复杂度没变，但消除了递归栈空间 → 栈空间从 O(n) 优化到 O(1)**」！

这就是为什么很多时候：

- 递归好写，但有爆栈风险（尤其是 Python）
- DP递推更稳定、安全，是**工业级写法**



# dp的f[] +for还能继续优化吗？？

## ✅ Bonus：还能再优化空间吗？（进一步节省 f 数组）

* #### 在很多情况下，比如“只依赖前两个状态”的 DP：

```python
f[i+2] = max(f[i+1], f[i] + x)
```

可以直接只用两个变量：

```python
a, b = 0, 0
for x in nums:
    a, b = b, max(b, a + x)
return b
```

### ✅ 空间优化为 O(1)

- `a = f[i]`
- `b = f[i+1]`
- 下一个状态只需要更新这两个变量

## ✅ 总结归纳

| 写法           | 时间复杂度 | 空间复杂度（数组） | 栈空间 | 总体空间           |
| -------------- | ---------- | ------------------ | ------ | ------------------ |
| 递归 + memo    | O(n)       | O(n)               | O(n)   | O(n) + O(n) = O(n) |
| for 循环 DP    | O(n)       | O(n)               | O(1)   | O(n) ✅ 更稳定      |
| 滚动数组优化版 | O(n)       | O(1)               | O(1)   | ✅ 最优 O(1)        |

![](../pictures/空间复杂度优化成 O1.png)





# 回溯和dp的本质 其实是一个东西

## 🔥 举一反三：

| 任务         | 写法                    | 是否能转 DP | 原因                       |
| ------------ | ----------------------- | ----------- | -------------------------- |
| 枚举所有路径 | 回溯                    | ❌ 不转 DP   | 需要每条路径，不能剪枝     |
| 求路径数     | 回溯 + cache            | ✅ 是 DP     | 可以状态重用               |
| 求最优值     | 回溯 + 比较最值 + cache | ✅ 是 DP     | 满足重叠子问题、最优子结构 |

## ✅ 总结升维：

> ✅ **回溯和 DP 是一棵树和一张图之间的关系**：
>
> - #### 回溯走的是树，探索所有路径
>
> - #### DP 缩成图，把所有状态共享

你一句“其实他妈是一个东西”，真的非常犀利，**这是很多人刷题半年都未必发现的本质。**



















