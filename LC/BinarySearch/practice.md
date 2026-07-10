# 🔍 Binary Search (二分查找) 专项练习

> **核心心法**：利用有序性，每次排除一半的搜索空间。$O(\log N)$ 的效率是多巴胺的源泉。

## 🟢 基础巩固 (Easy) - 已通关

| 题号 | 题目 | 状态 | 核心逻辑 / 多巴胺点 | 关键代码/笔记 |
| :--- | :--- | :---: | :--- | :--- |
| 704 | [Binary Search](https://leetcode.com/problems/binary-search/) | 🟢 | 标准模板 | `while (left <= right)`，最稳的基石。 |
| 35 | [Search Insert](https://leetcode.com/problems/search-insert-position/) | 🟢 | 找不到返回 `left` | 发现 `left` 停在插入点的瞬间非常治愈。 |
| 278 | [First Bad Version](https://leetcode.com/problems/first-bad-version/) | 🟢 | 找左边界 | 练习调用 API，学会“锁定第一个”的技巧。 |
| 69 | [Sqrt(x)](https://leetcode.com/problems/sqrtx/) | 🟢 | 数值二分 | 用 `x / mid` 防止溢出，跨界打击数学题。 |
| 744 | [Next Letter](https://leetcode.com/problems/find-smallest-letter-greater-than-target/) | 🟢 | 字符二分 | `char` 本质是整数；利用 `left % n` 处理循环。 |
| 852 | [Peak Index](https://leetcode.com/problems/peak-index-in-a-mountain-array/) | 🟢 | 趋势二分 | 比较 `mid` 和 `mid+1`，在山上定位峰顶。 |
| 1351 | [Count Negatives](https://leetcode.com/problems/count-negative-numbers-in-a-sorted-matrix/) | 🟢 | **矩阵二分** | **外层 Row 遍历 + 内层 Col 二分**。 |
| 1608| 
### 💡 1351 专项总结 (降维打击)
- **二维数组定义**：`m = grid.length` (行), `n = grid[0].length` (列)。
- **二分策略**：在降序数组中找“第一个负数”。
- **计算公式**：如果第一个负数在 `mid`，则该行负数个数 = `n - mid`。