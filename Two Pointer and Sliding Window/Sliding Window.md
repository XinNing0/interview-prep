# 滑动窗口 (Sliding Window) 刷题笔记

> 用于**连续**子数组/子串的问题。右指针扩张、左指针按需收缩，窗口在数组上"滑动"。
> 核心判断：**这题是"最长" "最短" 还是 "计数"？** → 决定 while 条件和记录位置。

---

## 一、三种类型（先分清用哪个）

| 类型 | 求什么 | while条件 | 记录位置 | 答案初始 | 典型题 |
|---|---|---|---|---|---|
| **最长型** | 最长合法窗口 | **违规了** | while**外**记max | 0 | 3, 424, 1004 |
| **最短型** | 最短满足窗口 | **满足了** | while**里**记min | inf | 209 |
| **计数型** | 子数组个数 | **违规了** | while外 count+= | 0 | 713 |
| **定长型** | 固定窗口k | (无while) | 每步更新 | — | 643, 567 |

**怎么判断用哪个：**
- 求**最长** → 最长型（扩到违规→缩回合法→记max）
- 求**最短** → 最短型（扩到满足→边缩边记min）
- 求**个数** → 计数型（`count += right-left+1`）
- 窗口**固定大小** → 定长型（进一个出一个）

---

## 二、通用模板（先背死骨架）

```python
def 滑窗(nums):
    left = 0
    状态 = 初始值          # sum=0 / product=1 / 计数dict{} / zeros=0
    答案 = 初始值          # 最长:0   最短:float('inf')   计数:0
    
    for right in range(len(nums)):
        纳入 nums[right]              # ① 右进(while外)
        
        while 窗口违规:
            移出 nums[left]           # ② 左出(while里)
            left += 1
        
        更新答案                      # ③ 合法时记录
    
    return 答案
```

### 三句口诀
1. **右进** — 纳入 nums[right]（while外，for里）
2. **违规缩** — 移出 nums[left]，left++（while里）
3. **合法记** — 记录答案（位置看类型）

### 核心思想
- **right（for）每轮必进一格**：主导扩张
- **left（while）按需缩**：只在违规时缩，可能不缩、可能缩多次
- 两指针**同向**（都往右），right在前left在后
- 窗口 = `[left...right]`，长度 = `right - left + 1`

---

## 三、各类型模板

### 最长型（3, 424）
```python
for right in range(len(s)):
    纳入 s[right]
    while 违规:                       # 有重复 / 要替换>k
        移出 s[left]; left += 1
    ans = max(ans, right - left + 1)  # while外记max
```

### 最短型（209）
```python
for right in range(len(nums)):
    cur_sum += nums[right]
    while cur_sum >= target:              # 满足了
        ans = min(ans, right - left + 1)  # while里记min
        cur_sum -= nums[left]; left += 1
return ans if ans != float('inf') else 0
```

### 计数型（713）
```python
if k <= 1: return 0
for right in range(len(nums)):
    product *= nums[right]
    while product >= k:
        product //= nums[left]; left += 1
    count += right - left + 1             # 以right结尾的合法子数组个数
```

### 定长型（643）
```python
window_sum = sum(nums[:k])           # 先算第一个窗口
max_sum = window_sum
for i in range(k, len(nums)):
    window_sum += nums[i]            # 右进
    window_sum -= nums[i-k]          # 左出(i-k是滑出位置)
    max_sum = max(max_sum, window_sum)
return max_sum / k
```

---

## 四、核心难点：713 的 count（我卡过的点）

`count += right - left + 1` 数的是**"以 right 结尾的所有合法子数组个数"**。

- 窗口 [left...right] 合法 → 它所有"以right结尾的子段"都合法（更短→乘积更小→必合法）
- 以right结尾往左延伸：[right], [right-1..right], ... [left..right]，共 `right-left+1` 个
- **不是 +1**（那只数1个，漏了），是 **+窗口长度**（一次数完以right结尾的全部）
- 每个子数组只在"它结尾那轮"被数一次 → 不重不漏

验证 `[1,2,3], k=10`（全<10，共6个子数组）：right=0加1、right=1加2、right=2加3 → 1+2+3=6 ✅

---

## 五、⚠️ 易错点（我踩过的坑）

1. **不能排序！** 滑窗处理"连续"，排序破坏顺序。
   （128可排序=不要求连续；209/713要连续=不能排）

2. **算和/积用 `nums[right]`，不是 `right`**（下标 vs 值的坑）。

3. **左出对应右进**：
   - 和：右进 `+= nums[right]`，左出 `-= nums[left]`
   - 积：右进 `*= nums[right]`，左出 `//= nums[left]`（除掉，整除）

4. **713 count 是 `+= right-left+1`**，不是 `+= 1`。

5. **最短型初始 inf，最后判断**：`return ans if ans != inf else 0`。

6. **while 不是 if**：收缩可能缩多次。

7. **窗口长度 = `right - left + 1`**（下标差+1）。

---

## 六、判断"用不用滑窗"

**用滑窗：** "连续"子数组/子串 + "最长/最短/个数" + 状态单调
- 209, 3, 713, 424, 1004, 567

**不用滑窗：**
| 情况 | 用什么 |
|---|---|
| 和=某值+个数+可能负数 | 前缀和+哈希（560）|
| 数值连续(非位置) | set（128）|
| 有序找一对 | 对撞双指针（167）|

**滑窗前提**：窗口扩大时状态**单调**（和越加越大）。有负数不单调 → 不能用（所以209/713要正数）。

---

## 七、滑窗 vs 前缀和（子数组两大流派）

| | 滑动窗口 | 前缀和+哈希 |
|---|---|---|
| 结构 | for套while，有left | 一个for，无left，dict |
| 适用 | ≥/< + 最长/最短/个数 + 正数 | =某值/整除 + 个数 + 可能负数 |
| 例 | 209, 713, 3 | 560, 974 |

---

## 八、刷过的题

- [x] **3** Longest Substring Without Repeating — 最长型，set/dict判重复
- [x] **209** Minimum Size Subarray Sum — 最短型，sum
- [x] **643** Maximum Average Subarray I — 定长，进一个出一个
- [x] **485** Max Consecutive Ones — 计数连续1
- [x] **424** Longest Repeating Char Replacement — 最长型，dict+max_freq，`长度-max_freq<=k`
- [x] **713** Subarray Product Less Than K — 计数型，`count += right-left+1`
- [x] **567** Permutation in String — 定长，Counter比频率
- [x] **1004** Max Consecutive Ones III — 最长型，数0的个数≤k

---

## 九、关键感悟

1. **先分类型**（最长/最短/计数/定长）→ 决定 while条件 和 记录位置。
2. **三种类型区别**：
   - 最长：违规缩，while外记max
   - 最短：满足缩，while里记min
   - 计数：count += right-left+1
3. **713 的 count 是精髓**：数"以right结尾的所有合法子数组"。
4. **滑窗(同向) vs 对撞(相向)**：看指针方向判断。
5. **负数不能用滑窗**（破坏单调）→ 用前缀和+哈希。
6. **反复默写**：看懂≠会写，关答案从空白写，隔天再练。