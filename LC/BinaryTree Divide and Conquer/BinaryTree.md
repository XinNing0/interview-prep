# 二叉树 (Binary Tree) 刷题笔记

> 树的题 90% 是递归。递归分两大流派:**分治**(拼返回值)和**遍历**(带状态走+全局收集)。
> 先判断流派,再套对应模板——**两套零件不能混装!**(我在257混装过:遍历骨架里写了 left or right)

---

## 〇、前置:TreeNode(三格小盒子)

```python
node.val      # 盒子里的数字
node.left     # 通往左孩子盒子的线(没有 = None)
node.right    # 通往右孩子盒子的线
```

- 点号可连用:`root.left.left.val`
- 空 = `None`(不是null!那是Java)
- **建树永远是判题系统的活**——p、q、root 都是建好的树直接用
- LC的class里递归主函数要 `self.xxx(...)`;**用内部def dfs可绕开self**(推荐,和backtrack手感统一)

---

## 一、两大流派:分治 vs 遍历(核心地图)

### 30秒判断法
> **"这题的答案,是一个'算出来的值',还是一堆'沿路捡的东西'?"**
> - 求一个值(深度/是否平衡/存在吗/公共祖先)→ **分治**
> - 求"所有xxx"(所有路径/所有满足的节点)→ **遍历**

⚠️ 别拿题面熟词当判据!112和113都写着"root-to-leaf",但112问"存在吗"(布尔→分治)、113问"返回所有"(收集→遍历)。**看返回值,不看题面描述。**
(这就是"回溯vs DP"那条判断的树版:所有方案→遍历/回溯;一个数→分治/DP)

### 全方位对照

| | 分治 | 遍历(=backtrack的树版!) |
|---|---|---|
| dfs返回什么 | **有意义的值**(数字/布尔/节点) | **什么都不返回**(裸return只是走人) |
| 答案在哪 | 在返回值里,层层拼上来 | 在全局res里,沿路收集 |
| 信息流向 | **自下而上**(儿子交我,我交爸) | **自上而下**(带path/remaining往下) |
| 递归行 | `left = dfs(...)` **接返回值** | `dfs(...)` **不接** |
| 有全局res吗 | 没有 | **有** |
| 有pop吗 | 没有 | **有**(append/pop成对同级) |
| 像我会的谁 | 77分治版(C(n,k)=子问题拼) | 39回溯(append→递归→pop→收) |

**最快识别:看递归行接不接返回值。**

---

## 二、分治模板 + "填三格"

```python
class Solution:
    def 主函数(self, root):
        def dfs(node):
            if not node:
                return 【空①】          # ← 底:空树交什么
            left = dfs(node.left)        # 问左儿子要答案(照抄)
            right = dfs(node.right)      # 问右儿子要答案(照抄)
            return 【空②】               # ← 拼:用left/right/node.val拼出我的答案
        
        return dfs(root)                 # 照抄(有时是 dfs(root) != -1 之类,见下)
```

**写分治 = 填三格**(写码前先口头填!):

| 格子 | 问自己 |
|---|---|
| ① 上交什么类型? | 数字?布尔?节点? |
| ② 底(空树)交什么? | 类型定了底就定:数字→0,布尔→True/False,节点→None |
| ③ 拿到left/right怎么拼? | max+1?and?or?交换挂回? |

### 已刷题的三格表

| 题 | ①上交 | ②底 | ③拼法 |
|---|---|---|---|
| **104** 最大深度 | 数字(高度) | 0 | `max(l,r)+1` |
| **226** 翻转 | 节点(翻好的子树) | None | `root.left,root.right = r,l` 再 `return root` |
| **100** 相同的树 | 布尔 | 都空True/一空一不空False | `val相等 and 左同 and 右同` |
| **101** 对称 | 布尔 | 同100两个底 | **交叉比**:`(a.left,b.right)` 和 `(a.right,b.left)` |
| **112** 路径总和 | 布尔 | False | `left or right`(一条就够→or!) |
| **110** 平衡树 | 数字+暗号 | 0 | 见-1传-1;差>1交-1;否则max+1 |

### 布尔拼法两式
- 题说"**都/全部**要满足" → `and`(100/101)
- 题说"**存在/任一/一条就够**" → `or`(112)

### 110 的 -1 暗号(复合返回值的经典设计)
一个return要交两样(高度+平衡吗)→ **用不可能的高度值-1兼职当"不平衡"警报**:
- 交 ≥0 的数 = "平衡,高度是它"
- 交 -1 = "别看高度了,下面烂了"
- -1 像112的True一样**层层冒泡到根**,还顺带剪枝(儿子-1我直接-1,不白算)

```python
def dfs(node):
    if not node: return 0
    left = dfs(node.left); right = dfs(node.right)
    if left == -1 or right == -1: return -1     # 收到警报→上传
    if abs(left - right) > 1: return -1         # 我这层差超1→举旗
    return max(left, right) + 1                 # 都好→正常交高度
# 主函数: return dfs(root) != -1
```
(abs必须有:right比left高时 left-right 是负数,`>1` 永远抓不到!)

### 分治的"自下而上"到底是什么
不是有东西在树里爬——是**每层函数停在 `left = dfs(...)` 那行等儿子return,拿到后算自己的,再return给等自己的人**。和 `a = f(5)` 是同一个动作,只是层数多。104:9交1、20交2、3拼出max(1,2)+1=3。

---

## 三、遍历模板(= backtrack 原样搬进树)

```python
def solve(self, root):
    res = []                          # 全局收集器
    path = []                         # 草稿纸(白板,不传)
    def dfs(node, 纸条):              # 纸条:remaining等每层不同的信息
        if not node:
            return                    # 裸return,不带值
        path.append(node.val)         # 进门:记上我
        if not node.left and not node.right:   # 到叶子
            if 结账条件:               # (257无条件收;113要remaining==node.val)
                res.append(收一份)     # path[:]快照 或 "->".join(path)
        else:
            dfs(node.left, 新纸条)     # 走两边,不接返回值
            dfs(node.right, 新纸条)
        path.pop()                    # 出门:擦掉我(和append同级!)
    dfs(root, 初始纸条)
    return res
```

对照39回溯:append→递归→pop 三行心脏一字不差;树的"选择"固定俩(左/右),连for都省了。

### 已刷题
| 题 | 纸条 | 叶子处 | 收集格式 |
|---|---|---|---|
| **257** 所有路径 | 无(只path) | 无条件收 | `"->".join(path)`(记得append时str()) |
| **113** 路径总和II | remaining | `remaining == node.val` 才收 | `path[:]`(数字列表,不join!) |

**113 = 257的骨架 + 112的结账**——两派零件各取一半的合体题。
收集格式看返回值类型:List[List[int]]→path[:];List[str]→join。

---

## 四、辅助def的判断("形状匹配"规则)

> **递归前对形状:主函数的参数,够不够递归每层要传的?**
> - 够 → 直接递归自己(100:p,q正好一对;112:root,targetSum正好节点+账)
> - 不够 → 写辅助def,参数按递归需要的来(101:主函数只有root,递归要一对→isMirror(a,b))
> - **拿不准就写内部def——永远不会错**,还能绕开self、给参数起真名(remaining)

---

## 五、⚠️ 易错点(全是实战踩的)

1. **两派零件不混装**:遍历骨架里不写 `left = dfs(...)` 和 `or`(257踩过);分治里没有res和pop。
2. **class里递归自己要 `self.`**,内部def不用——统一用内部def省心。
3. **叶子判断别丢**:112没有"是叶子吗"那层,单节点树会误判(正确路径多走一步进空节点)。四段剧本:①我空吗→②我是叶子吗(结账)→③委托孩子→④汇总。
4. **结账用remaining,不再碰targetSum**:账一路减在remaining里,叶子只比 `remaining == node.val`;targetSum只在发起时用一次。
5. **纸条 vs 白板**:remaining每层不同→传参;path/res共享→不传。
6. **Python缩进三查**(内容全对+缩进错=逻辑错,Java背景高发):
   - else和哪个if对齐?(它否定的是那句吗)
   - append和pop**同级**吗?(一进一出一对门)
   - 发起调用和return在主函数层吗?(没滑进dfs肚子里)
7. **`abs(left-right)`的abs别省**——负差抓不到。
8. **空是None不是null**;"都空"=`not p and not q`;"(排除都空后)一空一不空"=`not p or not q`,**顺序有讲究**(前一个先拦截)。

### 词组卡(树专用)
| 意思 | 词组 |
|---|---|
| 都空→True | `if not p and not q: return True` |
| 一空一不空→False | `if not p or not q: return False`(放"都空"之后) |
| 我是叶子 | `if not node.left and not node.right:` |
| 且起来直接交 | `return a and b and c` |
| 一条就够 | `return left or right` |
| 高度拼法 | `return max(left, right) + 1` |
| 交换挂回 | `root.left, root.right = right, left` |
| 路径收集 | `res.append(path[:])` / `res.append("->".join(path))` |

---

## 六、刷过的题

**分治:**
- [x] **104** Maximum Depth — 模板原题,max+1
- [x] **226** Invert — 交节点,交换挂回+return root
- [x] **100** Same Tree — 布尔,and,两个底
- [x] **101** Symmetric — 100的镜像:交叉比(左vs右)
- [x] **112** Path Sum — 布尔,or,叶子结账remaining==val
- [ ] **110** Balanced — -1暗号(理解了,待默写焊牢)

**遍历:**
- [x] **257** Binary Tree Paths — path+join收集+pop
- [x] **113** Path Sum II — 257骨架+112结账的合体

**待刷(课件清单):**
- [ ] 144/94/145 前中后序遍历(最裸的遍历)
- [ ] 102 层序遍历(BFS+deque,新词组)
- [ ] 236 LCA、98 验证BST(分治进阶)
- [ ] 1120 / 549 / 114(课件进阶,能讲即可)

---

## 七、复习节奏(给自己的)

- 分治基本盘 = 填三格熟练度:104/112 隔天重默(默前先口头填三格)
- 110 明天再看(-1只是第三格花了点,基本盘牢了它就简单)
- 遍历已通(=backtrack),257/113 隔几天各默一遍保温