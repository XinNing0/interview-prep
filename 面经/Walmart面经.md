# Walmart 面经题目整理 + 思路

> 来源：一亩三分地面经。按"OA / Coding面 / 考语言实现 / SD"分类。
> 每题给：类型 + 思路 + 关键点。⭐=你做过的同类

---

## 一、OA 算法题

### 1. 最大化 B[i] > A[i] 的和 ⭐（=你做过的870田忌赛马变体）
**题**：两等长数组A,B。s = sum of B[i] where B[i]>A[i]。求B所有排列中s的最大值。
**思路**：贪心。排序后，用B里"刚好比A[i]大的最小值"去配A[i]（田忌赛马）。
- 排序A和B
- 对每个A[i]，用B中大于它的最小值配对（这样"浪费"最小，能配的对最多）
- 能配上的B[i]累加进s
**关键**：和870一个套路——排序+贪心配对。O(n log n)。

### 2. k-capable 模型（贪心 + 前缀和/排序）
**题**：n个模型，每个有cost和2位二进制（是否支持FeatureA/B）。
选一个集合，支持A的≥k个、支持B的≥k个。对每个k(1~n)求最小总成本，不行返回-1。
**思路**（较难，先理解）：
- 模型分4类："11"(both)、"10"(只A)、"01"(只B)、"00"(都不)
- "11"最珍贵（一个顶两个）。策略：优先用便宜的"11"，不够再用"10"+"01"配对
- 对每个k：需要k个A、k个B。用t个"11" + (k-t)个"10" + (k-t)个"01"，枚举t取最小成本
- 各类按cost排序，用前缀和快速算"最便宜的若干个之和"
**关键**：分类 + 排序 + 前缀和 + 枚举"用几个11"。这题偏难，OA能做出暴力就不错。

### 3. task scheduler（考语言，sort）
**题**：add task / execute tasks 两个API。task有priority。一次性执行所有task。
**思路**：execute时直接按priority排序执行。不用heap（因为一次性全执行，不是一个个取）。
**关键**：看清example——"一次执行所有"就是sort，不是heap。

### 4. 数据分类整合（考语言，dict）
**题**：给struct(type,id,num)的list，生成 type→sum 的mapping，总sum，array:[(id,num)]按num排序。
**思路**：
- dict累加：`d[type] += num`
- 收集(id,num)，`sorted(..., key=lambda x: x[1])`按num排
**关键**：dict分组 + sorted排序 + 输出格式（可能要字符串拼接）。

---

## 二、Coding 面题

### 5. 找零有多少种方式（DP，硬币变体）
**题**：给硬币面额和金额，问有多少种凑法（组合数）。
**思路**：完全背包DP。
```python
def change(amount, coins):
    dp = [0] * (amount + 1)
    dp[0] = 1                    # 凑0有1种(啥都不选)
    for coin in coins:           # 先遍历硬币(保证组合不重复)
        for x in range(coin, amount + 1):
            dp[x] += dp[x - coin]
    return dp[amount]
```
**关键**：`dp[x] += dp[x-coin]`。外层硬币内层金额（求组合数的顺序）。这是LC518。

### 6. 括号匹配变体（栈）⭐
**题**：判断括号是否匹配（LC20变体）。
**思路**：栈。左括号入栈，右括号弹栈匹配。
```python
def isValid(s):
    stack = []
    pairs = {')':'(', ']':'[', '}':'{'}
    for ch in s:
        if ch in pairs:              # 右括号
            if not stack or stack.pop() != pairs[ch]:
                return False
        else:                        # 左括号
            stack.append(ch)
    return not stack                 # 栈空才匹配
```
**关键**：栈 + 字典存配对。经典必会。

### 7. 找第一个位置（二分 + 处理重复）
**题**：排序数组找某数第一次出现的位置。坑：有重复要找第一个。
`[1,3,5,8,8,8,8,8,9,10,15]` 找8 → 返回3(第一个8的下标)
**思路**：二分查找左边界。
```python
def findFirst(nums, target):
    left, right = 0, len(nums) - 1
    result = -1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            result = mid             # 记录,但继续往左找
            right = mid - 1          # 关键:找到了也往左缩
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return result
```
**关键**：找到target后不停，`right = mid-1`继续往左找第一个。这是二分找左边界。

### 8. 回文加空格
**题**：回文串每个字符间加空格，但连续的非回文部分不拆。
`abcxyba` → `a b cxy b a`
**思路**：双指针从两端向中间，匹配的字符分开加空格，中间不匹配的部分作为一块。
（较少见，理解思路即可：左右指针找对称，对称部分拆开，中间整块保留）

### 9. Course Overlap（哈希/集合）
**题**：`[["17","Math"],["17","English"],["58","Math"],["61","Physics"]]`
输出每对课程的共同科目：`{"17,58":["Math"], "17,61":[], "58,61":[]}`
**思路**：
- 先建 课程id → 科目集合 的dict：`{17:{Math,English}, 58:{Math}, 61:{Physics}}`
- 对每对课程，求科目集合的**交集**：`set1 & set2`
**关键**：dict分组 + set交集(`&`)。用你会的set运算。

### 10. Validate Sudoku（矩阵遍历）
**题**：验证n×n矩阵每行每列是否都是1~n无重复。
`[[1,2,3],[2,3,1],[3,1,2]]` → true
**思路**：
- 检查每行：是不是1~n且无重复（用set）
- 检查每列：同样
```python
def validate(grid):
    n = len(grid)
    for row in grid:                     # 检查行
        if sorted(row) != list(range(1, n+1)):
            return False
    for c in range(n):                   # 检查列
        col = [grid[r][c] for r in range(n)]
        if sorted(col) != list(range(1, n+1)):
            return False
    return True
```
**关键**：行列都要查，用set判无重复或sorted比对1~n。注意先判n×n方阵。

### 11. Validate Nonogram（矩阵+模拟）
**题**：验证黑白格矩阵是否符合行/列的连续黑格数描述。
**思路**：对每行/列，数连续B('B')的段长，和给定的描述比对。
（较复杂，模拟题：遍历每行数连续黑格段，和rows描述比；列同理）
**关键**：数"连续段长度"，和描述数组比对。理解思路即可。

---

## 三、SD 面题（Walmart coding为主，SD轻准备）

### 12. 包裹delivery状态查询优化
**题**：server查carrier第三方拿包裹状态，优化server和第三方通信。
**答题方向**：缓存(cache状态，减少查询)、批量查询(batch)、异步/webhook(第三方主动推送而非轮询)、限流。

### 13. e2e广告平台（很难，不用深准备）
**方向**：太宽泛，Walmart coding岗一般不会深挖。能说出大致模块即可。

---

## 四、考语言/实现的通用点

1. **OOD补class**：给interface(如shape)，自己补init和方法。按调用写法补全。
   ```python
   class Circle:
       def __init__(self, radius):    # 补init
           self.radius = radius
       def area(self):                # 补方法
           return 3.14159 * self.radius ** 2
   ```
2. **输出格式**：可能要字符串拼接。`",".join(map(str, list))` 等。
3. **HackerRank输入**：提前熟悉怎么读input(面经里多人卡在这浪费时间!)。

---

## 五、Walmart面试特点（从面经总结）

1. **两轮**：一轮Coding + 一轮SD（有些是纯coding）
2. **要跑test case**：HackerRank上要能运行、过所有测试。**提前熟悉HackerRank输入输出**！
3. **coding是medium变体**：如"输出subarray的start/end index"(不只返回值)
4. **考语言熟悉度**：OOD补全、输出格式、字符串拼接
5. **心态**：面经说"心理压力越小越容易过"

---

## 六、按类型归纳（这些题考的模式）

| 模式 | 面经题 | 你的状态 |
|---|---|---|
| 贪心+排序 | 最大化B>A(870), k-capable | ⭐870会,k-capable难 |
| DP | 找零(518) | 新,要学 |
| 栈 | 括号匹配(20) | ⭐会 |
| 二分 | 找第一个位置 | 要学(找边界) |
| 哈希/set | Course Overlap, 数据整合 | ⭐set/dict会 |
| 矩阵遍历 | Sudoku, Nonogram | 新 |
| OOD | 补class | 要熟悉语法 |
| 双指针 | 回文加空格 | ⭐双指针会 |