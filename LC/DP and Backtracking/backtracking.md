# 回溯 (Backtracking) 刷题笔记

> 用于**"返回所有方案"**的问题。一格一格试,走不通退回来换,走通了收集。
> 核心判断:**看返回值!** 要"所有组合/方案"(List of List)→ 回溯;要"最少/最多/几种"(一个数)→ DP。

---

## 一、回溯 vs DP(先分清,面试血泪教训)

| | 回溯 | DP |
|---|---|---|
| 题目问 | 返回**所有**方案 | 最少/最多/**几种**(只要数量) |
| 返回值 | List[List] / List[str] | 一个数字 |
| 例 | 22括号、39组合、77、排班题 | 322最少硬币、518几种凑法、70爬楼梯 |

⚠️ "**几种**"(518)和"**所有**"(39)一字之差:前者只要数量→DP,后者要列出来→回溯。
⚠️ 我面试时把"返回所有排班"往DP想,卡住了——**看到"all possible"就是回溯!**

---

## 二、通用模板(骨架固定,背死)

```python
result = []
path = []                        # 草稿纸:当前走到一半的方案

def backtrack(状态参数):
    if 结束条件:                  # ← 变化点1
        result.append(path[:])   # 收集!必须 path[:] 拷贝
        return
    for 选择 in 候选范围:          # ← 变化点2
        if 不合法: break/continue # ← 变化点3(剪枝,有约束才写)
        path.append(选择)         # ① 选
        backtrack(新状态)         # ② 递归往下
        path.pop()                # ③ 撤销(换下一个试)

backtrack(初始状态)
return result
```

### 三行心脏(所有回溯题不变)
```python
path.append(选择)    # 拿起
backtrack(...)       # 探索
path.pop()           # 放下,换一个
```
pop 就是"回溯"的本体:退回来,换条路。

### 两大必踩的坑
1. **`result.append(path[:])` 必须切片拷贝** —— 直接 append(path) 存的是引用,最后全变空!
2. **append 和 pop 必须成对** —— 忘了pop,草稿纸越写越乱。

---

## 三、组合类五兄弟(核心题组)

| 题 | 收集时机 | 递归传什么 | 去重? | 新知识 |
|---|---|---|---|---|
| **77** 组合 | 选满k个 | num+1 | 无 | 裸骨架 |
| **39** 凑target(可重复用) | remaining==0 | **i**(可再用自己) | 无 | remaining+剪枝 |
| **40** 凑target(每个一次,有重复元素) | remaining==0 | **i+1** | **要** | 同层去重 |
| **78** 所有子集 | **每步都收**(无结束条件) | i+1 | 无 | 收集时机灵活 |
| **90** 子集(有重复元素) | 每步都收 | i+1 | **要** | 78+40组合 |

### 递归参数的"旋钮"(一个参数,四种题)
```
backtrack(i)       → 元素可无限重复用(39硬币)
backtrack(i + 1)   → 每个位置只用一次(40/78/90)
backtrack(i + 2)   → 用一次且不能选相邻(198打家劫舍类)
backtrack(num + 1) → 选数字本身,只能越选越大(77/216)
```

### 去重(数组有重复元素时)
```python
candidates.sort()                                  # 先排序,重复挨在一起
if i > start and candidates[i] == candidates[i-1]:
    continue                                       # 同层跳过重复值
```
- **和 3Sum 的去重同一招**:排序+相邻跳过
- **`i+1` 和去重是两个独立的开关,互不相干**:
  - `i+1` 管**纵向**(一条路径里,同一**位置**不用两次)
  - `i>start` continue 管**横向**(同一格,相同**数值**不重试)
  - 纵向能接力用重复值([1,1,6]合法),横向不重试(防重复组合)

### 写题前问两个独立问题
1. 元素能重复用吗? → 能:传i / 不能:传i+1
2. 数组有重复元素吗? → 有:排序+去重行 / 没有:不加

---

## 四、动作类:22 生成括号(候选不是数组!)

n对括号的所有合法组合。**没有数组、没有start** —— 每步只有两个动作:放'(' 或 放')'。
for循环被两个if取代:

```python
def generateParenthesis(n):
    result = []
    path = []
    def backtrack(open_count, close_count):
        if len(path) == 2 * n:               # 放满2n个 → 收
            result.append("".join(path))     # 拼成字符串
            return
        if open_count < n:                   # 左括号有余额 → 能放'('
            path.append('(')
            backtrack(open_count + 1, close_count)
            path.pop()
        if close_count < open_count:         # 右<左 → 能放')'
            path.append(')')
            backtrack(open_count, close_count + 1)
            path.pop()
    backtrack(0, 0)
    return result
```

**数学本质**('('=+1, ')'=-1):合法 ⟺ 总和=0 **且** 前缀和全程≥0。
- `open<n` 保证总量;`close<open` 保证前缀和不为负
- 判定类题(只有一种括号)可直接用+1/-1前缀和;**多种括号(LC20)必须用栈**(计数丢失类型信息,"(]"会误判)

**启示**:for那层本质是"枚举当前合法动作"——数组题=选下标,22=放哪种括号,迷宫题=走哪个方向。骨架不变。

---

## 五、⚠️ 易错点(我踩过的)

1. **下标 vs 值(老坑在回溯重现!)**:`for i in range(...)` 里,存path、和remaining比的是**值 `candidates[i]`**,传start的是**下标 i**。写前问:"我要位置还是里面的数?"
2. **剪枝比错对象**:`if candidates[i] > remaining`(值 vs 还差的),不是 `i > target`。
3. **剪枝不是必填项**:有约束才剪(39/40的和),78没约束就不写,别硬编一个永假的。
4. **`x == a or b` 语法陷阱**:非空字符串恒True!必须 `ch == a or ch == b` 或 `ch in '([{'`。
5. **40双结束条件**:216要"满k个**且**remaining==0"两个都满足才收。
6. **字符↔数字转换**:`int('8')→8`,`str(8)→'8'`;判断数字字符 `ch.isdigit()`。
7. **字符串不能原地改**:先 `list(s)` 转列表,填完 `"".join()` 拼回。

---

## 六、复杂度 & 面试策略

- 回溯是**指数级**(方案本身指数多,列全没有捷径),题目规模一般很小(n≤20)
- 剪枝不改变最坏复杂度,但大幅加速
- **面试卡住时**:先说"要所有方案→回溯"的判断 → 写骨架(哪怕内部空着) → 举小例子手推 → 沟通。面试官评估的是拆解过程,不是必须解出来。
- **多道题先扫一遍,先做有把握的;难题卡20分钟主动切题**。

---

## 七、刷过的题

- [x] **77** Combinations — 裸骨架(另发现分治解法:C(n,k)=C(n-1,k)+C(n-1,k-1))
- [x] **39** Combination Sum — 可重复用,传i,remaining剪枝
- [x] **40** Combination Sum II — i+1 + 排序同层去重
- [x] **78** Subsets — 每步都收,无结束条件
- [x] **90** Subsets II — 78+去重
- [x] **22** Generate Parentheses — 动作类,open/close两规则
- [x] **Walmart面试原题** workingHours — 见下

---

## 八、Walmart 技术面原题:workingHours(2026-07)

**题目**:给 totalHours(周总工时,约束≤56,不一定正好56)、maxDaily(日最大工时,约束≤8)、pattern(7字符)。
pattern每位:**数字0~8 = 这天已固定**(0=固定休息日,不用填);**'?' = 待定**,可填0~maxDaily任何数(**包括填0**,即最终决定这天不工作也合法)。
返回**所有**填法,使7天总和 = totalHours。

**实考输入**:`56, 8, "???8???"` → 固定8小时,剩48分给6个?,48÷6=8且日上限8 → 唯一解 `["8888888"]`

**判断**:"返回所有方案" → **回溯**(面试时误判为DP,卡住)。

**步骤**(面试官引导的流程):
1. 按constraints校验输入(≤56、≤8、长度7、字符合法、至少一个?)
2. 解析pattern:统计固定工时、记录?的位置
3. remaining = totalHours − 固定工时
4. 回溯:每个?试填0~maxDaily,剪枝,填完且remaining==0收集

```python
def workingHours(totalHours, maxDaily, pattern):
    # 解析:拆开数字和?
    fixed_hours = 0
    q_positions = []
    for i in range(len(pattern)):
        if pattern[i] == '?':
            q_positions.append(i)          # 记?的下标
        else:
            fixed_hours += int(pattern[i]) # 字符转数字累加

    remaining_total = totalHours - fixed_hours
    schedule = list(pattern)               # 字符串转list才能按位置填
    result = []

    def backtrack(idx, remaining):
        # idx:正在填第几个?   remaining:还剩多少小时要分
        # 强剪枝:剩的?全填满也不够 → 死路
        if remaining > (len(q_positions) - idx) * maxDaily:
            return
        if idx == len(q_positions):        # 所有?填完
            if remaining == 0:             # 且正好分完 → 收
                result.append("".join(schedule))
            return
        for h in range(0, maxDaily + 1):   # 这个?试填0~8
            if h > remaining:              # 剪枝:比剩的还多
                break
            schedule[q_positions[idx]] = str(h)   # 填(数字转字符)
            backtrack(idx + 1, remaining - h)
        schedule[q_positions[idx]] = '?'   # 撤销:恢复成?

    backtrack(0, remaining_total)
    return result

# 面试原输入
print(workingHours(56, 8, "???8???"))   # ['8888888']
```

**注意**:这题没有path,直接在schedule上填/擦——撤销不是pop,是把那格改回'?'。**"试→递归→恢复原状"的三步心脏没变**,只是草稿纸从path换成了schedule。

**本题 = 39/216 换皮**:固定数量的空位、每个空位有取值范围(0~8)、总和固定、要所有方案。LC39/216练熟,这题就是套模板。