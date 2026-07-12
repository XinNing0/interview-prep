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

## 二、通用模板(骨架固定)

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

### 写代码前的思考:三问半(参数不是背的,是问出来的)

拿到题,先在纸上答完再动手:

> **① 每步什么在变?** → 就是参数(要落到**变量名**:idx/start/remaining,不是描述)
> **②-前:站在当前状态,我的"选择"是什么?有几种?** → 就是for要遍历的东西(最容易漏的半问!)
> **②-后:做了一个选择,状态怎么变?** → 递归括号里的表达式(remaining-x / start+seg_len)
> **③ 状态到什么值算到头?** → 结束条件(要落到**判断式**:==0 / ==len)

⚠️ 我的"想一半"规律:状态(①)每次都能想到,卡的永远是"选择"(②-前)——补上这半问,for自动长出来。

### "选择"只有四种(②-前卡住时对着扫)
1. 选数组/字符串里的**一个元素** → `for i in range(start, len)`(39/40/78)
2. 从固定候选**挑一个**(字母/括号/方向)→ `for x in 候选` 或几个if(17/22)
3. 决定**切多长/填多大** → `for 长度 in range(1, 上限+1)`(93/131/面试题workingHours)
4. 当前元素**选或不选** → 两个分支

### 什么参数要传、什么不用传(办公室比喻)
- path/result/nums = **墙上的白板**,所有层都看得见 → **不传**
- idx/remaining/start = **每层不同、白板上看不出来** → 必须**写纸条传进去**
- 特例46:进度由 `len(path)` 自带 → `backtrack()` 空括号,啥都不传
- 判断标准:**只传"每层不同、且共享变量里看不出来"的信息**

### 三段式写码流程(从"想出大概"到"写出代码"的桥)
1. **三问半**(想法层,答案落到变量/表达式/判断式)
2. **中文行**(桥,不许跳!把答案组装成一行行中文注释)
3. **逐行翻译**成代码——**手指点着中文行逐字对**,不许凭印象重造
4. **对照自查**(写完30秒,拿中文行核代码:"从start遍历"→range里有start吗?)

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

## 四.5、映射类:17 电话号码字母组合

每个数字对应几个字母,返回所有组合。**每格的候选不一样**(第1格从"abc"挑,第2格从"def"挑)。

```python
def letterCombinations(digits):
    if not digits: return []
    phone = {'2':'abc','3':'def','4':'ghi','5':'jkl',
             '6':'mno','7':'pqrs','8':'tuv','9':'wxyz'}
    result = []
    path = []
    def backtrack(idx):                      # idx:处理到第几个数字
        if idx == len(digits):               # 每个数字都配上了 → 收
            result.append("".join(path))
            return
        for letter in phone[digits[idx]]:    # 这格的候选=映射表查出来的
            path.append(letter)
            backtrack(idx + 1)
            path.pop()
    backtrack(0)
    return result
```

---

## 四.6、排列类:46 全排列(不用start,不用传参!)

顺序有关([1,2]≠[2,1])→ 每格从头挑,用"不在path里"筛已用过的。

```python
def permute(nums):
    result = []
    path = []
    def backtrack():                     # 空括号!需要的nums/path都是共享变量
        if len(path) == len(nums):       # 进度由len(path)自带
            result.append(path[:])
            return
        for num in nums:                 # 每格从头挑(排列不用start!)
            if num not in path:          # 已用过的跳过
                path.append(num)
                backtrack()
                path.pop()               # append谁pop谁,成对!
    backtrack()
    return result
```

- 组合用start(只往后挑),排列不用start(每格全候选、靠"没用过"筛)
- `num not in path` 是O(n),更快的用 `used=[False]*n` 数组(进阶)

---

## 四.7、切分类:93 复原IP / 131 分割回文串(切字符串)

**题型特征**:把字符串切成若干段,每段满足某条件,返回所有切法。
**选择 = "这刀切多长"**(四种选择里的第3种)。

### 93 复原IP地址
切成4段,每段0~255、无前导零。

```python
def restoreIpAddresses(s):
    result = []
    path = []
    def backtrack(start):                # start:切到哪了(剩余=s[start:])
        if len(path) == 4 and start == len(s):   # 双条件:4段 且 用完!
            result.append(".".join(path))         # 用点连成IP字符串
            return
        if len(path) == 4:               # 4段满但没用完 → 死路
            return
        for seg_len in range(1, 4):      # IP段长1~3
            if start + seg_len > len(s): # 切出界(切之前查!防int("")崩)
                break
            segment = s[start : start + seg_len]
            if int(segment) > 255:       # 超255 → 更长更大,break
                break
            if len(segment) > 1 and segment[0] == '0':   # 前导零 → 跳
                continue
            path.append(segment)
            backtrack(start + seg_len)
            path.pop()
    backtrack(0)
    return result
```

### 131 分割回文串
每段是回文,返回所有分法(List[List[str]])。

```python
def partition(s):
    result = []
    path = []
    def backtrack(start):
        if start == len(s):              # 单条件:切完即收(没有段数限制)
            result.append(path[:])       # 收列表,不join!(看返回值类型)
            return
        for seg_len in range(1, len(s) - start + 1):   # 上限=剩余全长,+1别忘
            segment = s[start : start + seg_len]
            if segment != segment[::-1]: # 不是回文 → 跳(continue前是"坏情况"!)
                continue
            path.append(segment)
            backtrack(start + seg_len)
            path.pop()
    backtrack(0)
    return result
```

### 93 vs 131 对照(同型不同细节,别互相照抄!)
| | 93 | 131 |
|---|---|---|
| 段长范围 | 1~3(固定) | 1~剩余全长 |
| 到头条件 | **双**:4段且用完 | **单**:用完即可 |
| 越界检查 | 要(for固定1~3可能超) | 不要(上限天然不越界) |
| 合法判断 | ≤255、无前导零 | 是回文 `seg == seg[::-1]` |
| 收集格式 | `".".join(path)`(字符串) | `path[:]`(列表) |

---

## 五、⚠️ 易错点(我踩过的)

1. **下标 vs 值(老坑在回溯重现!)**:`for i in range(...)` 里,存path、和remaining比的是**值 `candidates[i]`**,传start的是**下标 i**。写前问:"我要位置还是里面的数?"
2. **剪枝比错对象**:`if candidates[i] > remaining`(值 vs 还差的),不是 `i > target`。
3. **剪枝不是必填项**:有约束才剪(39/40的和),78没约束就不写,别硬编一个永假的。
4. **`x == a or b` 语法陷阱**:非空字符串恒True!必须 `ch == a or ch == b` 或 `ch in '([{'`。
5. **40双结束条件**:216要"满k个**且**remaining==0"两个都满足才收。
6. **字符↔数字转换**:`int('8')→8`,`str(8)→'8'`;判断数字字符 `ch.isdigit()`。
7. **字符串不能原地改**:先 `list(s)` 转列表,填完 `"".join()` 拼回。
8. **continue前面永远是"坏情况"**:93是"坏的跳"(超255跳),131是"好的才要"→ 条件是 `!=回文` 才跳。写之前念:"什么情况该跳?"——我把93的形状搬到131,方向反了,结果全颠倒。
9. **收集格式看返回值类型**:List[List]→`path[:]`;字符串→`".".join(path)`/`"".join(path)`。join不是回溯标配!收之前瞄一眼函数签名。
10. **range右端不含**:要切到长度L,写 `range(1, L+1)`。93是range(1,**4**),131是range(1, len(s)-start**+1**)。
11. **for循环变量就地诞生,不用提前定义**(letter/seg_len/num);但名字要反映含义,不叫string这种类型名。
12. **函数里退出用return,break只能跳出循环**。
13. **`x == a or b` 陷阱升级版:比较用`==`不用`is`**;字符串成员判断用 `ch in '([{'`。
14. **换题不换脑的坑**:骨架可以搬,细节必须按本题校准——每个判断问一句"**这道题**需要这个检查吗?该跳的是什么?收什么格式?"
15. **收完必须return**——不return会继续往下跑for,idx越界报错(17踩过)。

### 常用词组卡(见一个存一个,当单词背)
| 意思 | 词组 |
|---|---|
| 收快照 | `result.append(path[:])` |
| 收字符串 | `result.append("".join(path))` / `".".join(path)` |
| 切一段 | `s[start : start + seg_len]` |
| 是回文 | `segment == segment[::-1]` |
| 字符转数字 / 数字转字符 | `int(ch)` / `str(h)` |
| 试几种长度 | `for seg_len in range(1, 上限+1)` |
| 前导零 | `len(seg) > 1 and seg[0] == '0'` |
| 没用过才选(排列) | `if num not in path:` |
| 同层去重(组合) | `if i > start and nums[i] == nums[i-1]: continue` |

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
- [x] **17** Letter Combinations — 映射类,每格候选不同(phone表)
- [x] **46** Permutations — 排列类,不用start不传参,num not in path
- [x] **93** Restore IP Addresses — 切分类,双到头条件,三条合法规则
- [x] **131** Palindrome Partitioning — 切分类,单到头条件,回文判断
- [x] **Walmart面试原题** workingHours — 见下

**待刷**:79单词搜索(网格类,接200的DFS)、47全排列II(46+去重)、78/90隔天重写

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