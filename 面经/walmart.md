# Walmart 近6个月真题 · 复习冲刺计划

> 23道真题，按题型分类 + 你的状态。距离面试(下周二三)几天冲刺。
> ⭐=已练过 | 🔥=高频重点 | 💤=Hard只看思路不硬啃

---

## 分类速览 + 状态

### 数组 / 双指针 / 哈希（强项区）
- 1 Two Sum (E) ⭐已会
- 121 Best Time Buy/Sell (E) ⭐已会
- 283 Move Zeroes (E) — 双指针，简单
- 1200 Min Abs Difference (E) ⭐已做
- 15 3Sum (M) 🔥 — 双指针进阶，**最高频必会**
- 75 Sort Colors (M) — 计数排序(好懂)或三指针
- 11 Container With Most Water (M) — 双指针对撞
- 49 Group Anagrams (M) — dict分组
- 870 Advantage Shuffle (M) — 田忌赛马贪心(你OA考过)

### 滑动窗口 / 前缀和（刚练的强项）
- 3 Longest Substring (M) ⭐已会
- 560 Subarray Sum=K (M) ⭐啃过
- 567 Permutation in String (M) ⭐已理解
- 713 Subarray Product<K (M) — 滑窗，209同类
- 239 Sliding Window Max (H) 💤 — 单调队列，看思路

### 字符串
- 6 Zigzag Conversion (M) — 模拟题

### 图 / BFS / DFS / 拓扑
- 200 Number of Islands (M) ⭐学过
- 207 Course Schedule (M) — 拓扑排序(新)

### 区间
- 56 Merge Intervals (M) 🔥 — 排序+合并，**高频**

### 设计 / Hard（只看思路）
- 460 LFU Cache (H) 💤
- 42 Trapping Rain Water (H) 💤
- 22 Generate Parentheses (M) — 回溯(时间够再看)

---

## 每日复习计划

> 原则：一天3-4题。上午攻新的高频，下午重刷已会的(默写)，晚上英文话术。
> 每题走清单流程：读题(光标指+抄约束)→ set/dict?下标/取值?→ 暴力→优化。

### Day 1 — 巩固强项 + 双指针高频
- **重刷**(限时默写)：560、3、1200
- **新** 🔥：**15 (3Sum)** — 最高频，双指针进阶，重点投入
- **新**：283 (Move Zeroes) — 简单双指针，快速拿下

### Day 2 — 双指针 + 哈希高频
- **新**：11 (Container With Most Water) — 双指针对撞
- **新**：49 (Group Anagrams) — dict分组(和242同源)
- **新**：75 (Sort Colors) — 先学计数排序(好懂)
- **重刷**：567

### Day 3 — 区间 + 图（补没碰的类型）
- **新** 🔥：56 (Merge Intervals) — 高频，排序+合并
- **重刷**：200 (Number of Islands) — 复习BFS/DFS
- **新**：207 (Course Schedule) — 拓扑排序(新概念)
- **新**：713 (Subarray Product<K) — 滑窗，你会滑窗就好上

### Day 4（面试前一天）— 轻量 + Hard看思路 + 模拟
- **只看思路**💤：42、239、460 — 理解怎么做，能讲，不默写
- **870** Advantage Shuffle — 田忌赛马思路(你OA考过)
- **6** Zigzag — 看一遍(模拟题，不难但烦)
- 挑1道已会的，**全程英文出声讲**，模拟面试25分钟
- **早睡**！状态 > 多刷一道

### 面试当天
- 面试前做1道简单的(1或121)热身

---

## 优先级（时间不够就按这个）

**必须能写**(高频+中等)：
15、49、56、11、75、283、200、560、3、567、713、207、1200、1、121

**只看思路**(Hard，能讲即可)：
460 LFU、42 接雨水、239 滑窗最大值

**时间够再看**：22 回溯、6 Zigzag、870

---

## 重要提醒

1. **704二分不在这份真题里！** Walmart近6个月没考纯二分模板题，所以降优先级，先按真题高频来。
2. **每题走读题流程**——你读题的坑(漏k、术语permutation)面试要特别小心。光标指读、抄约束。
3. **出声讲英文**，从现在练。每题写完过复杂度。
4. **Hard别碰最优解**，Walmart现场极少逼你写Hard。时间给中等高频题。
5. **同类题连做**巩固：15↔11↔167(双指针)；49↔242(哈希)；560↔3↔567↔713(滑窗/前缀和)。

---

## 你已掌握的模式（自信来源）
- 前缀和：560/1480/724/303 ✅
- 滑窗：3/209/643/485/424/567 ✅
- 双指针：26/88/167/1200 ✅
- 哈希：1/217/242/128/136 ✅
- BFS/DFS：200/733 ✅
这些覆盖了Walmart真题的大半。你不是从零，是查漏+冲刺。

---

## Python技巧备忘（这几天学的）
- `Counter(s)` = 数每个字符几次(= 手写`d.get(ch,0)+1`)
- `defaultdict(int)` = 不存在的key自动返回0
- `sorted(range(n), key=lambda i: nums[i])` = 排序保留原下标
- `{b: [] for b in B}` = 字典推导式
- `"".join(list)` = 拼字符串
- set运算：`a & b`(交) `a | b`(并) `a - b`(差)
- **LeetCode用Python3！** 避免 `/` 整除坑

## 面试话术
1. Restate: "So I need to..., input is..., right?"
2. Clarify: "Empty? Negatives? Duplicates?"
3. Brute force: "Let me start with O(n²)..."
4. Optimize: "I notice... so O(n) using..."
5. Test: "Let me trace [example]..."
6. Complexity: "Time O(n), space O(1)."
卡住: "Let me think out loud... I'm stuck on X, let me try Y."