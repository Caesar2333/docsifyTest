# dp中的排列和组合问题是什么？？

## ✅ 关键本质（不记模板，记哲学）

| 目标 | 状态定义                  | 循环顺序           | 本质哲学                               |
| ---- | ------------------------- | ------------------ | -------------------------------------- |
| 组合 | f[j]：凑出 j 的**组合数** | 外层物品，内层体积 | 每个组合只考虑一次，用“顺序先后”来去重 |
| 排列 | f[j]：凑出 j 的**排列数** | 外层体积，内层物品 | 所有排列都要考虑，不去重               |

* #### 排列的话，[1,2]和[2,1]算两种

* #### 组合的话，[1,2]和[2,1]算一种。



对，你总结得 **💯 精准**：

> ✅ **只要你用了偏移（比如 `f[i + 2]`），那你他妈就得把 `f[0]`、`f[1]` 这些被空出来的位置初始化填上**，否则就是在玩火，早晚炸锅！
