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

### ★最底层的统一:所有树递归 = 一个骨架 + "处理自己"放前/中/后
```python
def dfs(node):
    if not node: return 底
    # 【前序位置】← 处理自己放这:我→左→右
    left = dfs(node.left)
    # 【中序位置】← 放这:左→我→右
    right = dfs(node.right)
    # 【后序位置】← 放这:左→右→我
```

**"前中后序"由"处理自己(node.val 或 return)相对 dfs(left)/dfs(right) 的位置"决定**——这是所有树题(分治+遍历)共用的判断法:

| "处理自己"放哪 | 是什么序 | 谁在用 |
|---|---|---|
| 【前】记录值 `res.append(node.val)` | 前序 | 遍历:自顶向下、序列化 |
| 【中】记录值 | 中序 | 遍历:**BST专用**(吐出来有序!) |
| 【后】记录值 | 后序 | 遍历:先处理孩子后处理自己 |
| 【后】`return 拼(left,right)` | 后序 | **分治**(几乎都在这!) |

**关键洞察:分治 ≈ 后序** —— 因为"先拿孩子返回值、自己最后拼"天然是后序(左右先、自己后)。104的 `left=dfs(l); right=dfs(r); return max+1` 就是后序位置处理自己。**你写分治时其实一直在用后序。**

**怎么选序(拿到题问自己):**
> **我处理"自己"时,需要孩子的结果吗?**
> - 需要(深度/平衡/路径和)→ 放【后】(后序/分治)
> - 不需要,从上往下传就行 → 放【前】(前序)
> - **BST且要利用有序** → 放【中】(中序)
>
> ⚠️ 没有"默认前序"!序由题目需求决定。求深度用前序根本写不出(自己的深度依赖孩子,孩子还没算)。

### 30秒判断法(分治还是遍历)
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
def solve(self, root):
    def dfs(node):
        if not node:
            return 底值               # ← 格2
        left = dfs(node.left)         # 问儿子要答案(会"等"儿子return)
        right = dfs(node.right)
        return 拼法(left, right, node.val)   # ← 格3
    return dfs(root)
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

### or/and 判断只看一件事(别被题目细节带偏)
> **题目问"存在/一条/能不能" → or;问"所有/每条/是否都" → and**
> 和路径和正负、结账条件松紧**无关**——那些只决定"什么算数",不决定"要一条还是要全部"。

### 110 的 -1 暗号(复合返回值的经典设计)
一个return要交两样(高度+平衡吗)→ **用不可能的高度值-1兼职当"不平衡"警报**:
- 交 ≥0 的数 = "平衡,高度是它"
- 交 -1 = "别看高度了,下面烂了"
- -1 像112的True一样**层层冒泡到根**,还顺带剪枝(儿子-1我直接-1,不白算)

```python
def dfs(node):
    if not node: return 0
    left = dfs(node.left)
    if left == -1: return -1          # 提前剪枝:左边已烂,右边不用算了(更优)
    right = dfs(node.right)
    if right == -1: return -1
    if abs(left - right) > 1: return -1         # 我这层差超1→举旗
    return max(left, right) + 1                 # 都好→正常交高度
# 主函数: return dfs(root) != -1
```
(abs必须有:right比left高时 left-right 是负数,`>1` 永远抓不到!)

**更好懂的替代写法(不用暗号,返回tuple)**——dfs需要返回多个信息时的通用技巧:
```python
def dfs(node):
    if not node: return (True, 0)              # (平衡吗, 高度)
    leftBal, leftH = dfs(node.left)             # 拆包接收
    rightBal, rightH = dfs(node.right)
    balanced = leftBal and rightBal and abs(leftH-rightH) <= 1
    height = max(leftH, rightH) + 1
    return (balanced, height)
# 主函数: return dfs(root)[0]
```
两版都对:-1版更"炫技"简洁;tuple版更直白、更好迁移(236 LCA等也能用"返回tuple"这招)。

### 分治的"自下而上"到底是什么
不是有东西在树里爬——是**每层函数停在 `left = dfs(...)` 那行等儿子return,拿到后算自己的,再return给等自己的人**。和 `a = f(5)` 是同一个动作,只是层数多。104:9交1、20交2、3拼出max(1,2)+1=3。

### ★递归是"V字形":先下去,再上来(理解树的命门)
一次dfs调用要走完 **去程(往下)→ 到底 → 回程(往上)** 的完整来回,**方向相反的两段**:

```
   去程 ↓(找base case)          回程 ↑(算答案)
dfs(3) 暂停,派人问9           dfs(3) 最后算出3
   ↓                              ↑
dfs(9) 暂停,派人问None        dfs(9) 算出1
   ↓                              ↑
dfs(None) 到底                dfs(None) 返回0 ← 答案从最底下开始长
```

- **"从根开始看node空不空" = 去程**(控制流往下走,找底)
- **"答案从叶子往上长出来(0→1→2→3)" = 回程**(返回值往上传)
- **两个都对,是同一次递归的两半,不矛盾!**(我卡过:以为"从上看空不空"和"从下长答案"冲突——其实是V字的两条边)

### 分治 vs 遍历,真正的区别 = 在哪一段干活
| | 去程(往下)干活 | 回程(往上)干活 | 所以 |
|---|---|---|---|
| **分治** | 不干,只往下派人 | **拼答案**(拿孩子返回值max+1) | 自下而上,重点在回程 |
| **遍历** | **干活**(path.append、走到就记) | 只pop擦除,不产生答案 | 自上而下,重点在去程 |

- 分治的活在**回程**:去程啥也不算,到底了往回走才开始算(所以104不需要pop——它去程没往path里放东西)
- 遍历的活在**去程**:往下走时就append记录(所以257需要pop——去程放了东西,离开要擦)

**判断口诀升级**:分治=去程下探/回程拼返回值;遍历=去程append收集/回程pop擦除。

---

## 二.5、【精讲】104 从头到尾走一遍(树的母题)

**思维转变(树题命门)**:别想"遍历整棵树找最深路径"——只想**当前一个节点**,问:
> "假设我两个孩子已经把各自深度算好告诉我了,我怎么算我的?"
> → 我的深度 = max(左孩子深度, 右孩子深度) + 1;孩子怎么知道?同样方法问下去;问到空树=0(底)

```python
def maxDepth(self, root):
    def dfs(node):
        if not node:                    # ①底:空,深度0
            return 0
        left = dfs(node.left)           # ②问左孩子多深(装进left)
        right = dfs(node.right)         # ③问右孩子多深
        return max(left, right) + 1     # ④深的那个+我这层,交给父亲
    return dfs(root)
```

**答案诞生的时间顺序(全在回程,自下而上):**
```
dfs(None)=0 (最先,最底) → dfs(9)=1 → dfs(20)=2 → dfs(3)=3 (最后,最顶)
```
每个dfs只干一件事:拿到左右深度、max+1、交上去。**不关心整棵树,只关心"我和我俩孩子"。**

**三个自测(答对=懂了):**
- `dfs(None)` 返回0,必须有——否则无限递归没有底
- `left = dfs(node.left)` 执行完,left里装的是**左子树的深度**(一个数)
- `+1` = 算上我自己这一层

**母题变形练习**(改一个字就是新题):
- 求节点总数:`return left + right + 1`(不是max,是相加)
- 求最小深度:`return min(left,right)+1`(注意单边空的坑)

## 三、遍历三件套:144/94/145(一行三个位置)

三道题代码几乎一样,只差 `res.append(node.val)` 放的位置:

```python
def dfs(node):
    if not node: return
    # res.append(node.val)   ← 放这 = 前序(144):我→左→右
    dfs(node.left)
    # res.append(node.val)   ← 放这 = 中序(94):左→我→右
    dfs(node.right)
    # res.append(node.val)   ← 放这 = 后序(145):左→右→我
```

同一棵树 `[1,[2],[3]]`:前序 1,2,3 / 中序 2,1,3 / 后序 2,3,1 —— **只有根"1"的位置在变**。

**连回V字(去程/回程):**
- **前序**=一进节点就记 → 干活在**去程**
- **后序**=离开节点才记 → 干活在**回程**(所以分治=后序=回程干活)
- 中序=走完左、正要走右时记 → 中间

**★BST中序 = 从小到大有序!**(BST"左<我<右",中序"左→我→右"正好按大小走)
→ 98验证BST(中序检查严格递增)、230第k小(中序数到第k个)

## 四、遍历模板(= backtrack 原样搬进树)

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

## 四.5、辅助def的判断("形状匹配"规则)

> **递归前对形状:主函数的参数,够不够递归每层要传的?**
> - 够 → 直接递归自己(100:p,q正好一对;112:root,targetSum正好节点+账)
> - 不够 → 写辅助def,参数按递归需要的来(101:主函数只有root,递归要一对→isMirror(a,b))
> - **拿不准就写内部def——永远不会错**,还能绕开self、给参数起真名(remaining)

---

## 五、BST(二叉搜索树)= 二叉树 + 一条铁律

### 定义(精确版!)
> **对每个节点:整棵左子树所有值 < 我 < 整棵右子树所有值**

关键是**"整棵"**和**"每个节点都要满足"**——不是"左孩子<我<右孩子"(那只是局部)!
反例:根5,右孩子8,8的左孩子2 → 2<8局部对,但2在5右边该>5,2<5 → **不是BST**。
正因为约束跨越多层祖先(深层节点看不见祖先),才需要往下传边界(见武器2的98)。

⚠️ 读题看到 "binary **search** tree"/"BST" 要条件反射想两个武器,别当普通binary tree做(否则O(log n)题写成O(n),面试被追问)。这是漏词老坑的新高发区。

### 两个武器

**武器1:比大小,只走一边**(O(h)≈O(log n))
```python
if val < node.val:   往左   # 小的在左
elif val > node.val: 往右   # 大的在右
else:                就是它
```
用于:搜索(700)、插入(701)、删除(450)、LCA(235)、最接近值(270)

**武器2:中序遍历 = 从小到大有序**
BST"左<我<右",中序"左→我→右"正好按大小走 → 吐出升序。
```python
# 通用起手式:中序存进array,在有序数组上做事
arr = []
def dfs(node):
    if not node: return
    dfs(node.left); arr.append(node.val); dfs(node.right)
```
用于:验证BST(98)、第k小(230)、最小差(530)、累加树(538)

### 拿到BST题先问
> 查找/插入/删除某个值 → 武器1(比大小走一边)
> 验证有序 / 第k小 / 最接近 → 武器2(中序有序)

### ★BST武器1 = 二叉树上的二分!(跨专题连线)

BST武器1 和 二分查找(704)**本质是同一个东西**:

```python
# 704 二分(有序数组)          # 700 BST搜索(树)
while left <= right:            while node:
    mid=(left+right)//2             if val==node.val: return node
    if nums[mid]==target: ...       elif val<node.val: node=node.left  # 小→左
    elif nums[mid]<target:          else: node=node.right             # 大→右
        left=mid+1  # 往右半
    else: right=mid-1  # 往左半
```

| | 二分(数组) | BST武器1(树) |
|---|---|---|
| 靠什么砍半 | 数组有序 | 左<我<右 |
| "中间" | nums[mid] | 当前node |
| 小了往 | 左半 | 左孩子 |
| 大了往 | 右半 | 右孩子 |
| 复杂度 | O(log n) | O(h)≈O(log n) |

**BST = 把有序数组组织成树**:数组 `[1,3,5,8,10,14]` 二分跳中点8 ≈ BST根是8、左子树是左半、右子树是右半。"往左孩子走"="往左半查"。

**整张记忆网**:对撞双指针(挪1格)→ 有序能砍半 → 二分(数组砍半)→ BST武器1(树上砍半)。同一个"比大小、砍一半、走一边"的思想。

### 递归 vs 循环:什么时候不用递归

**判据:走完一个分支还要回头走另一个吗?**
- **要回头/分叉**(左右都处理)→ **递归**(需要"记住回来的路"):104/98/230/257
- **只走一条路,永不回头**(单路下探)→ **循环够了**:700/701/235

所以**BST武器1(比大小)全能用循环**(=树上二分,像704那样while):
```python
def searchBST(self, root, val):     # 700循环版
    node = root
    while node:
        if val == node.val: return node
        node = node.left if val < node.val else node.right
    return None
```
**武器2(中序)还得递归**(要遍历全树)。
能循环的循环更好(不占递归栈,O(1)空间);面试能写一种即可,用循环是加分(懂"不用递归也行")。

### 98 上下界解法(low/high = 把所有祖先约束打包传下来验票)

**为什么必须传low/high**:一个节点光看自己和孩子,判断不了合不合法(看不到祖先定的规矩)。
low/high就是把"从根到我,所有祖先给的约束"打包成两个数,一路传下来验票:
- 往**左**走:左子树都要**<我** → 上界(high)收紧成我
- 往**右**走:右子树都要**>我** → 下界(low)收紧成我
- 两个都要传:一个节点可能同时被"要>某祖先"和"要<某祖先"两头夹

```python
def dfs(node, low, high):
    if not node: return True
    if node.val <= low or node.val >= high: return False   # 不在允许范围
    return (dfs(node.left, low, node.val)         # 往左:上界收紧成我
            and dfs(node.right, node.val, high))  # 往右:下界收紧成我
# 主函数: return dfs(root, float('-inf'), float('inf'))
```
这个"往下传约束、越传越紧"和112传remaining同构——你早会这个套路了,只是纸条换成了两个边界。

也可用中序做(存array检查严格递增),两个武器都行。

### BST 已刷题

- [x] **700** 搜索 — 武器1,循环单路(=树上二分)
- [x] **701** 插入 — 武器1+走到空位`return TreeNode(val)`+接住重挂
- [x] **98** 验证BST — 上下界(low/high往下传,收紧范围);也可中序检查递增
- [x] **230** 第k小 — 武器2:中序存array取`arr[k-1]`(或边走边计数,找到即停更省空间)
- [x] **235** LCA — 武器1循环:都小往左/都大往右/分叉了我就是LCA(比普通树236简单!)
- [ ] **450** 删除 — 武器1找到+三种情况(叶子/单孩子/双孩子找替身)+接住重挂(最难,压轴,待啃)

**450的思路(暂放,回头啃)**:
- 用大小关系单路找到key
- 找到后三种情况:①叶子→直接删(return None) ②只一个孩子→那孩子顶上(return 存在的那个孩子) ③两个孩子→找右子树最小的"替身"(后继),把它的值搬上来,再去右子树递归删掉那个替身
- 全程"接住重挂":`root.left = self.deleteNode(root.left, key)`

---

## 六、改结构必用"接住重挂"(分治的实战应用)

**凡是会改变树结构的操作(删除/插入/翻转/剪枝),必须用分治:dfs返回"处理好的子树根",父亲用 `root.left = dfs(root.left)` 接住重挂。**

**为什么遍历做不到**:遍历没有向上交付的通道,深处对node的重新赋值只改了局部变量,树上的指针(父亲的.left/.right)没跟着动,"接不上"。
**分治能做到**:父亲拿返回值覆盖自己的指针,不管下面发生了什么变化,`root.left = 新的` 一次赋值就完成接线。

```python
# 701插入(最干净的例子):
def insertIntoBST(self, root, val):
    if not root:
        return TreeNode(val)      # 走到空位→造新节点,交上去(插入发生在这!)
    if val < root.val:
        root.left = self.insertIntoBST(root.left, val)    # 接住重挂
    else:
        root.right = self.insertIntoBST(root.right, val)
    return root
```
对比700(只读,底返回None="没找到")vs 701(改写,底返回新节点="给你造一个")——**一行之差,只读变改写**。

---

## 七、⚠️ 易错点(全是实战踩的)

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
9. **分治里没有裸return**——每个出口都要带值(110踩过:底写成裸return,该是return 0)。分治的return后面永远有东西;遍历才有裸return。
10. **BST题看清是不是要用大小关系/中序**——读题漏看"search"这个词,会把O(log n)题写成O(n)遍历全树。

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
| BST走一边 | `node = node.left if val < node.val else node.right` |
| 拆包接收tuple | `leftBal, leftH = dfs(node.left)` |

---

## 八、刷过的题(总览)

**分治:**
- [x] **104** Maximum Depth — 模板原题,max+1(白板✓)
- [x] **226** Invert — 交节点,交换挂回+return root
- [x] **100** Same Tree — 布尔,and,两个底(白板✓)
- [x] **101** Symmetric — 100的镜像:交叉比(左vs右)
- [x] **112** Path Sum — 布尔,or,叶子结账remaining==val(白板✓)
- [x] **110** Balanced — -1暗号 或 tuple版(白板✓,两种写法都会)

**遍历:**
- [x] **144/94/145** 前中后序 — 一行三位置
- [x] **257** Binary Tree Paths — path+join收集+pop
- [x] **113** Path Sum II — 257骨架+112结账的合体

**BST:**
- [x] **700** 搜索、**701** 插入、**98** 验证、**230** 第k小、**235** LCA
- [ ] **450** 删除(思路已懂,待默写)

**待刷:**
- [ ] **102** 层序遍历(BFS+deque)—— 单独开专场练,树递归之外唯一的新形态
- [ ] 236(普通树的LCA,对比235看BST的优势)
- [ ] 1120 / 549 / 114(进阶,能讲即可)

---

## 九、复习节奏(给自己的)

- 白板特训已过一批:104/110/100/112全部白板默写成功
- 待白板确认:94/257/226(理解了,还没纸上默过)
- 450:等BST其余题隔几天保温后再啃(三种情况+找替身,一次学新东西不要太多)
- BFS/102:单独安排一个时段,别和递归树题混着学