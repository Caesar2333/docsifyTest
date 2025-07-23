# deque双端队列——队列和栈

# ✅ LeetCode 中各种场景统一用法指南

## 📌 一、你只需要一个接口 + 一个实现类：

```java
Deque<Integer> deque = new LinkedList<>();
```

- ✅ `Deque` 是统一处理 **队列（Queue）+ 栈（Stack）+ 双端操作** 的接口
- ✅ `LinkedList` 实现类最灵活，性能够用，支持双端插入删除

------

## 📌 二、常用操作对应场景（你背这个表就完了）

| 你要干的事     | 对应方法               | 翻译                      |
| -------------- | ---------------------- | ------------------------- |
| ✅ 入队（尾部） | `deque.offerLast(x);`  | 正常队列入队（enqueue）   |
| ✅ 出队（头部） | `deque.pollFirst();`   | 正常队列出队（dequeue）   |
| ✅ 看队首       | `deque.peekFirst();`   | 队首元素（不删）          |
| ✅ 看队尾       | `deque.peekLast();`    | 队尾元素（不删）          |
| ✅ 入栈（头部） | `deque.offerFirst(x);` | 栈 push 操作              |
| ✅ 出栈（头部） | `deque.pollFirst();`   | 栈 pop 操作               |
| ❌ 不要用       | `add/remove/element()` | 会抛异常，LeetCode 不稳妥 |

## ✅ `Deque` 常用 API 成功 / 失败返回总结表（LeetCode 实战推荐）

| 你要干的事 | 对应方法        | 成功返回                   | 失败返回（如队空或容量满） |
| ---------- | --------------- | -------------------------- | -------------------------- |
| 入队（尾） | `offerLast(x)`  | `true`                     | `false`（仅容量满会失败）  |
| 出队（头） | `pollFirst()`   | 队首元素 `E`               | `null`（队列为空）         |
| 看队首     | `peekFirst()`   | 队首元素 `E`               | `null`（队列为空）         |
| 看队尾     | `peekLast()`    | 队尾元素 `E`               | `null`（队列为空）         |
| 入栈（头） | `offerFirst(x)` | `true`                     | `false`（仅容量满会失败）  |
| 出栈（头） | `pollFirst()`   | 栈顶元素 `E`（= 头部元素） | `null`（栈空 = 队列空）    |

## 📌 三、你怎么判断用“栈”还是“队列”？

| 题目描述       | 用法                                                         |
| -------------- | ------------------------------------------------------------ |
| “先进先出”     | `offerLast + pollFirst` —— 正常队列（尾巴入栈，头出战），队列正序的时候，头在左边。尾巴入队，头出队。 |
| “后进先出”     | `offerFirst + pollFirst` —— 模拟栈——栈正序的时候，头在右边   |
| “双端滑动窗口” | `Deque` 本来就是专门干这个的                                 |

### ✅ 二、按「栈」与「队列」的语义重新整理

#### ✔️ 栈结构（LIFO）：

你要的是“从**头部加**，从**头部取**”，即：

| 动作   | 方法            |
| ------ | --------------- |
| 入栈   | `offerFirst(x)` |
| 出栈   | `pollFirst()`   |
| 看栈顶 | `peekFirst()`   |

✅ 是不是很像你平时自己写的 `stack.push(x)`、`stack.pop()`？
 其实就是把“**头部**”当作“栈顶”。

------

#### ✔️ 队列结构（FIFO）：

你要的是“**尾部入**、头部出”，即：

| 动作   | 方法           |
| ------ | -------------- |
| 入队   | `offerLast(x)` |
| 出队   | `pollFirst()`  |
| 看队首 | `peekFirst()`  |

就是我们熟悉的 `enqueue` → `从尾巴进`，`dequeue` → `从头部出`。







# Deque怎么遍历

### ➤ 最稳妥的遍历方式（**从队首开始**）：

```java
for (int x : deque) {
    System.out.println(x); // 从队首 peekFirst() 一路向后
}
```

### 反向遍历

### 示例：逆序遍历 `Deque`

```java
Iterator<Integer> it = deque.descendingIterator();
while (it.hasNext()) {
    System.out.println(it.next()); // 从队尾往队首
}
```

## ✅ 五、总结你需要背的实战句式

```java
// 正序遍历（队首 → 队尾）
for (int x : deque) {
    ...
}

// 逆序遍历（队尾 → 队首）
Iterator<Integer> it = deque.descendingIterator();
while (it.hasNext()) {
    int x = it.next();
}
```



# Collections常用的api

## ✅ 二、`java.util.Collections` 常见 API（针对 List）

| 方法名                             | 用法       | 场景                     | 说明                    |
| ---------------------------------- | ---------- | ------------------------ | ----------------------- |
| `Collections.sort(list)`           | 排序       | 排序 List<Integer>       | 原地升序                |
| `Collections.sort(list, cmp)`      | 自定义排序 | 排序 List<String> 等对象 | 常配合 lambda           |
| `Collections.reverse(list)`        | 反转       | 翻转链表/字符串等        | 原地修改                |
| `Collections.shuffle(list)`        | 洗牌       | 随机打乱顺序             | 常用于出题构造数据      |
| `Collections.max(list)`            | 最大值     | 找最大元素               | 要求元素实现 Comparable |
| `Collections.min(list)`            | 最小值     | 找最小元素               | 同上                    |
| `Collections.frequency(list, val)` | 计数       | 统计某元素出现次数       | O(n)                    |

### 📌 LeetCode 常用组合

```java
Collections.sort(list);
Collections.reverse(list);
Collections.max(list);
Collections.min(list);
```



## 求最大值，最小值，总和，以及反转

## ✅ 1. 数组求最大值 / 最小值 / 求和

### 🔸 int[] 版本（最常见）

```java
int[] arr = {3, 5, 1, 9};

// 最大值
int max = Arrays.stream(arr).max().getAsInt();  // ➜ 9

// 最小值
int min = Arrays.stream(arr).min().getAsInt();  // ➜ 1

// 求和
int sum = Arrays.stream(arr).sum();             // ➜ 18
```

> ✅ `int[]` 流是 `IntStream`，所以结果用 `.getAsInt()`。



### ✅ 替代方案 A：先变 List，再反转

```java
Integer[] arr = {1, 2, 3, 4, 5};

// 转 List
List<Integer> list = Arrays.asList(arr); // 只有 Integer才能流畅使用 否则的话int[]会被当做一个元素

// 反转
Collections.reverse(list);

// 如果需要再转回数组
arr = list.toArray(new Integer[0]);
```



# Arrays常用的api

## ✅ 一、`java.util.Arrays` 常见 API（针对数组）

| 方法名                              | 用法          | 场景                             | 说明                                   |
| ----------------------------------- | ------------- | -------------------------------- | -------------------------------------- |
| `Arrays.sort(arr)`                  | 排序          | 排序 int[]、char[]、String[] 等  | 默认升序；原地排序                     |
| `Arrays.sort(arr, cmp)`             | 自定义排序    | 排序 Integer[] / String[] 等对象 | 用 `Comparator` 指定规则               |
| `Arrays.toString(arr)`              | 打印          | 调试、打印数组                   | 输出如 `[1, 2, 3]`                     |
| `Arrays.copyOf(arr, newLen)`        | 截取 or 扩容  | 数组拷贝、新建副本               | 超过原长补 0                           |
| `Arrays.copyOfRange(arr, from, to)` | 子数组        | 获取子数组                       | 包头不包尾 [from, to)                  |
| `Arrays.fill(arr, val)`             | 全填充        | 初始化数组为固定值               | 整体赋初值                             |
| `Arrays.equals(arr1, arr2)`         | 判断相等      | 比较两个数组                     | 要素、顺序相同才 true                  |
| `Arrays.binarySearch(arr, key)`     | 二分查找      | 有序数组中查找元素               | 若没找到返回 `-(插入点)-1`             |
| `Arrays.stream(arr)`                | 包装为 Stream | 配合 `filter/map/collect`        | 基本类型用 `.mapToObj()` 或 `.boxed()` |

### 📌 LeetCode 最常用组合

```java
Arrays.sort(arr); // 排序
Arrays.copyOf(arr, n); // 拷贝子数组
Arrays.fill(dp, INF);  // 初始化DP数组
Arrays.binarySearch(arr, target); // 二分
Arrays.stream(arr).boxed().collect(Collectors.toList()); // int[] -> List<Integer>
```



# StringBuilder

## ✅ StringBuilder API 全套（按回溯场景分类整理）

| 用法类别           | 方法                               | 含义                                 | 示例                                      |
| ------------------ | ---------------------------------- | ------------------------------------ | ----------------------------------------- |
| 📌 初始化           | `new StringBuilder()`              | 创建空字符串构造器                   | `StringBuilder sb = new StringBuilder();` |
| 📌 添加字符         | `sb.append(char)`                  | 追加一个字符（常用于 dfs）           | `sb.append('a');`                         |
| 📌 添加字符串       | `sb.append(String)`                | 追加一个字符串                       | `sb.append("ab");`                        |
| 📌 删除最后一个字符 | `sb.deleteCharAt(sb.length() - 1)` | 回溯：恢复现场（像 path.removeLast） | `sb.deleteCharAt(sb.length() - 1);`       |
| 📌 清空内容         | `sb.setLength(0)`                  | 清空所有字符（重用 sb）              | `sb.setLength(0);`                        |
| 📌 转成字符串       | `sb.toString()`                    | 返回 `String` 结果                   | `String s = sb.toString();`               |
| 📌 获取字符         | `sb.charAt(i)`                     | 获取第 i 个字符（只读）              | `char c = sb.charAt(0);`                  |
| 📌 获取长度         | `sb.length()`                      | 当前长度                             | `int len = sb.length();`                  |

## ✅ 最推荐的核心套路（背下来能写一切）

```java
StringBuilder sb = new StringBuilder();

sb.append(char);              // 添加字符
dfs(..., sb);                // 把 sb 传进去
sb.deleteCharAt(sb.length() - 1); // 回溯：删除最后一个字符
```

可以完美模拟 List<Character> 的 add/removeLast 操作，但性能更好。



# 给知道长度的某一个数组附上初值

###  场景二：知道长度，统一赋某个值（如填充成某字符）

用 `Arrays.fill()` 是最方便的：

```java
char[] chars = new char[5];
Arrays.fill(chars, 'x'); // 所有位置都变成 'x'
int[] nums = new int[4];
Arrays.fill(nums, 42); // 全部是 42
```



# 如果一个数组中出现负数怎么办？？

* 



# ceiling的快捷方式是什么

* #### x + k-1 / k 这个就是 ceiling的快捷方式



# Map中的常见api

## ✅ 一：`getOrDefault(key, defaultValue)`

### 📌 作用：

> 如果 key 存在，返回对应的值；如果 key 不存在，返回你传的默认值（不会插入）

### 🧪 示例代码：

```java
Map<String, Integer> map = new HashMap<>();
map.put("apple", 3);

// key 存在，返回 3
int a = map.getOrDefault("apple", 0); // 3

// key 不存在，返回默认值 0（⚠️不会插入进 map）
int b = map.getOrDefault("banana", 0); // 0
```

### ✅ 特点：

- 不改变原 map
- 只是“查不到就临时返回默认值”
- 常用于**计数初始化**前的判断

## ✅ 二：`computeIfAbsent(key, k -> new ...)`

### 📌 作用：

> 如果 key 不存在，就自动调用 lambda 表达式创建一个默认值，并 put 到 map 中

### 🧪 示例代码：

```java
Map<String, List<Integer>> map = new HashMap<>();

// 如果 key 不存在，就插入 new ArrayList<>()
// 然后把 42 添加进去
map.computeIfAbsent("apple", k -> new ArrayList<>()).add(42);
```

相当于下面的老写法：

```java
if (!map.containsKey("apple")) {
    map.put("apple", new ArrayList<>());
}
map.get("apple").add(42);
```

### ✅ 特点：

- 会自动插入 key → defaultValue（如果 key 不存在）
- 如果 key 已经存在，就不会执行 lambda
- 常用于**Map 的嵌套结构初始化**



# Java List 常用方法对照表

| 方法                              | 说明                                | 示例                                         |
| --------------------------------- | ----------------------------------- | -------------------------------------------- |
| `add(E e)`                        | 在末尾添加元素                      | `list.add("A")`                              |
| `add(int index, E e)`             | 在指定位置插入元素                  | `list.add(1, "B")`                           |
| `addAll(Collection<? extends E>)` | 添加另一个集合的所有元素            | `list.addAll(otherList)`                     |
| `get(int index)`                  | 获取指定位置的元素                  | `String val = list.get(2)`                   |
| `set(int index, E element)`       | 替换指定位置的元素                  | `list.set(1, "New")`                         |
| `remove(int index)`               | 删除指定下标位置的元素              | `list.remove(3)`                             |
| `remove(Object o)`                | 删除首次出现的指定元素（按值删）    | `list.remove("A")`                           |
| `contains(Object o)`              | 判断列表是否包含某元素              | `list.contains("A")`                         |
| `isEmpty()`                       | 判断列表是否为空                    | `list.isEmpty()`                             |
| `size()`                          | 返回当前元素个数                    | `list.size()`                                |
| `clear()`                         | 清空所有元素                        | `list.clear()`                               |
| `indexOf(Object o)`               | 返回首次出现的索引（找不到返回 -1） | `list.indexOf("B")`                          |
| `subList(int from, int to)`       | 切片（左闭右开）                    | `list.subList(1, 4)`                         |
| `list.stream()`                   | 获取 Stream 流（支持 map/filter）   | `list.stream().map(...).toList()`            |
| `list.toArray()`                  | 转为 Object[] 数组                  | `Object[] arr = list.toArray()`              |
| `list.toArray(new String[0])`     | 转为具体类型数组（⚠️推荐）           | `String[] arr = list.toArray(new String[0])` |

## 🔥 衍生操作技巧

### 1️⃣ 添加元素时常用的套路：

```java
List<List<Integer>> g = new ArrayList<>();
for (int i = 0; i < n; i++) g.add(new ArrayList<>());

g.get(0).add(1); // 给第 0 个节点的子节点列表中添加一个儿子
```

------

### 2️⃣ 初始化列表的一种写法（JDK 9+）：

```java
List<Integer> list = List.of(1, 2, 3); // 但这个是不可变的
```

⚠️ 注意：`List.of()` 是 **不可变列表**，不能 `.add()`，否则抛异常。

如果你想要“可变的”：

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
```

------

### 3️⃣ 你可能会遇到的 `addAll` 用法：

```java
List<Integer> a = new ArrayList<>(List.of(1, 2));
List<Integer> b = new ArrayList<>(List.of(3, 4));
a.addAll(b); // a = [1, 2, 3, 4]
```





# 父亲节点数组 转化成邻接表

```java
List<List<Integer>> list = new ArrayList<>();
int n = parent.length;
for(int i = 0; i < n; i++) list.add(new ArrayList<>()); // 从0开始初始化

for(int i = 1; i < n; i++) list.get(parent[i]).add(i); // 得到父亲，加入儿子，数据从开始维护
```







