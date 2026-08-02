# 二叉树 (Binary Tree) 刷题笔记

> 树的题 90% 是递归。递归分两大流派:**分治**(拼返回值)和**遍历**(带状态走+全局收集)。
> 先判断流派,再套对应模板——**两套零件不能混装!**(我在257混装过:遍历骨架里写了 left or right)
>
> ⚠️ **BST(二叉搜索树)内容已独立成 binary_search_tree_notes.md**——两个方法、和二分的连线、700/701/98/230/235/530/501/450/669/776/270 全部在那份里。本文件专注普通二叉树(分治+遍历两大流派)。
>
> 正在把已掌握的题从Python换成Java重写,思路不用重推,重点是语法转换。

---

## 〇、前置:TreeNode(三格小盒子)

### Python
```python
node.val      # 盒子里的数字
node.left     # 通往左孩子盒子的线(没有 = None)
node.right    # 通往右孩子盒子的线
```

### Java
```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```
点号用法完全一样:`node.val`、`node.left`、`node.right`。**唯一的语言差异**:空节点判断,Python `if not node:`，Java是 **`if (node == null)`**（不是null，不是None，是Java自己的null）。

- 点号可连用:`root.left.left.val`
- **建树永远是判题系统的活**——p、q、root 都是建好的树直接用
- Python的class里递归主函数要 `self.xxx(...)`;**Java不需要！类内方法互相调用直接写方法名，天然可见**，这一点比Python省心（不用纠结self./内部def的取舍）。

---

## 一、两大流派:分治 vs 遍历(核心地图)

### ★最底层的统一:所有树递归 = 一个骨架 + "处理自己"放前/中/后

**Python**
```python
def dfs(node):
    if not node: return 底
    # 【前序位置】← 处理自己放这:我→左→右
    left = dfs(node.left)
    # 【中序位置】← 放这:左→我→右
    right = dfs(node.right)
    # 【后序位置】← 放这:左→右→我
```

**Java**
```java
private 返回类型 dfs(TreeNode node) {
    if (node == null) return 底;
    // 【前序位置】← 处理自己放这
    返回类型 left = dfs(node.left);
    // 【中序位置】← 放这
    返回类型 right = dfs(node.right);
    // 【后序位置】← 放这
    return 拼法;
}
```
`返回类型`换成具体类型：`int`（数字）、`boolean`（布尔）、`TreeNode`（节点）——这就是"填三格"的格①，Java里要显式写出这个类型，Python不用管。

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
> - **BST且要利用有序** → 放【中】(中序,详见BST笔记)
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

### Python
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

### Java
```java
public 返回类型 solve(TreeNode root) {
    return dfs(root);
}

private 返回类型 dfs(TreeNode node) {
    if (node == null) {
        return 底值;                          // ← 格2
    }
    返回类型 left = dfs(node.left);           // 问儿子要答案(不用self.,直接调)
    返回类型 right = dfs(node.right);
    return 拼法(left, right, node.val);       // ← 格3
}
```

**写分治 = 填三格**(写码前先口头填!):

| 格子 | 问自己 |
|---|---|
| ① 上交什么类型? | 数字(`int`)?布尔(`boolean`)?节点(`TreeNode`)? |
| ② 底(空树)交什么? | 类型定了底就定:数字→0,布尔→true/false,节点→null |
| ③ 拿到left/right怎么拼? | max+1?&&?交换挂回? |

### 已刷题的三格表

| 题 | ①上交 | ②底 | ③拼法 |
|---|---|---|---|
| **104** 最大深度 | 数字(高度) | 0 | `max(l,r)+1` |
| **226** 翻转 | 节点(翻好的子树) | None/null | `root.left,root.right = r,l` 再 `return root` |
| **100** 相同的树 | 布尔 | 都空True/一空一不空False | `val相等 and 左同 and 右同` |
| **101** 对称 | 布尔 | 同100两个底 | **交叉比**:`(a.left,b.right)` 和 `(a.right,b.left)` |
| **112** 路径总和 | 布尔 | False | `left or right`(一条就够→or!) |
| **110** 平衡树 | 数字+暗号 | 0 | 见-1传-1;差>1交-1;否则max+1 |

### 布尔拼法两式
- 题说"**都/全部**要满足" → Python `and` / Java `&&` (100/101)
- 题说"**存在/任一/一条就够**" → Python `or` / Java `||` (112)

### or/and 判断只看一件事(别被题目细节带偏)
> **题目问"存在/一条/能不能" → or;问"所有/每条/是否都" → and**
> 和路径和正负、结账条件松紧**无关**——那些只决定"什么算数",不决定"要一条还是要全部"。

### 110 的 -1 暗号(复合返回值的经典设计)
一个return要交两样(高度+平衡吗)→ **用不可能的高度值-1兼职当"不平衡"警报**:
- 交 ≥0 的数 = "平衡,高度是它"
- 交 -1 = "别看高度了,下面烂了"
- -1 像112的True一样**层层冒泡到根**,还顺带剪枝(儿子-1我直接-1,不白算)

**Python**
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

**Java**
```java
public boolean isBalanced(TreeNode root) {
    return dfs(root) != -1;
}
private int dfs(TreeNode node) {
    if (node == null) return 0;
    int left = dfs(node.left);
    if (left == -1) return -1;               // 提前剪枝
    int right = dfs(node.right);
    if (right == -1) return -1;
    if (Math.abs(left - right) > 1) return -1;   // 我这层差超1→举旗
    return Math.max(left, right) + 1;            // 都好→正常交高度
}
```

**更好懂的替代写法(不用暗号,返回tuple)**——dfs需要返回多个信息时的通用技巧:

**Python**
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

**Java(⚠️ Java没有tuple!用int[]代替)**
```java
public boolean isBalanced(TreeNode root) {
    return dfs(root)[0] == 1;                  // 数组第0项:1=平衡,0=不平衡
}
private int[] dfs(TreeNode node) {
    if (node == null) return new int[]{1, 0};  // {平衡吗(用1/0代替true/false), 高度}
    int[] left = dfs(node.left);
    int[] right = dfs(node.right);
    boolean balanced = left[0] == 1 && right[0] == 1
                        && Math.abs(left[1] - right[1]) <= 1;
    int height = Math.max(left[1], right[1]) + 1;
    return new int[]{balanced ? 1 : 0, height};
}
```
**Java没有Python的tuple(`(a,b)`一次打包两个值)。** 常见替代方案:
- **两个值都是同类型(这里都能编码成int)**→ 用 `int[]`（数组），最简单
- **类型不同**（比如要同时返回一个`boolean`和一个`String`）→ 数组塞不下,要么用 `Object[]`（丑），要么自己定义一个小类（`class Result { boolean balanced; int height; }`），更规范但代码量多。BST的776(拆成两棵树)也会遇到这个选择。

两版都对:-1版更"炫技"简洁;tuple/数组版更直白、更好迁移。

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

骨架就三句(Python和Java一样):空→返回0;问左要深度、问右要深度;两者取大的+1,交给父亲。

**答案诞生的时间顺序(全在回程,自下而上):**
```
dfs(None)=0 (最先,最底) → dfs(9)=1 → dfs(20)=2 → dfs(3)=3 (最后,最顶)
```
每个dfs只干一件事:拿到左右深度、max+1、交上去。**不关心整棵树,只关心"我和我俩孩子"。**

**三个自测(答对=懂了):**
- `dfs(null)`(Java写法) 返回0,必须有——否则无限递归没有底
- `left = dfs(node.left)` 执行完,left里装的是**左子树的深度**(一个数)
- `+1` = 算上我自己这一层

**母题变形练习**(改一个字就是新题):
- 求节点总数:`return left + right + 1`(不是max,是相加)
- 求最小深度:`return Math.min(left,right)+1`(注意单边空的坑)

## 三、遍历三件套:144/94/145(一行三个位置)

三道题代码几乎一样,只差 `res.append(node.val)`(Java是`res.add(node.val)`) 放的位置:

**Python**
```python
def dfs(node):
    if not node: return
    # res.append(node.val)   ← 放这 = 前序(144):我→左→右
    dfs(node.left)
    # res.append(node.val)   ← 放这 = 中序(94):左→我→右
    dfs(node.right)
    # res.append(node.val)   ← 放这 = 后序(145):左→右→我
```

**Java**
```java
private void dfs(TreeNode node, List<Integer> res) {
    if (node == null) return;
    // res.add(node.val);   ← 放这 = 前序(144)
    dfs(node.left, res);
    // res.add(node.val);   ← 放这 = 中序(94)
    dfs(node.right, res);
    // res.add(node.val);   ← 放这 = 后序(145)
}
```
`res.append(x)` → `res.add(x)`（Java的List用add不用append，这是最容易顺手打错的一个词）。

同一棵树 `[1,[2],[3]]`:前序 1,2,3 / 中序 2,1,3 / 后序 2,3,1 —— **只有根"1"的位置在变**。

**连回V字(去程/回程):**
- **前序**=一进节点就记 → 干活在**去程**
- **后序**=离开节点才记 → 干活在**回程**(所以分治=后序=回程干活)
- 中序=走完左、正要走右时记 → 中间

**★BST中序 = 从小到大有序!**——中序的这个特性是BST方法2的核心,详见 binary_search_tree_notes.md。

## 四、遍历模板(= backtrack 原样搬进树)

### Python
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

### Java
```java
private List<结果类型> res = new ArrayList<>();      // 全局收集器(类的字段,类似Python的闭包res)
private List<Integer> path = new ArrayList<>();       // 草稿纸

public List<结果类型> solve(TreeNode root) {
    dfs(root, 初始纸条);
    return res;
}

private void dfs(TreeNode node, 纸条类型 参数) {
    if (node == null) {
        return;                                        // 裸return,不带值
    }
    path.add(node.val);                                // 进门:记上我
    if (node.left == null && node.right == null) {      // 到叶子
        if (结账条件) {
            res.add(new ArrayList<>(path));             // ★复制一份快照!不能直接res.add(path)
        }
    } else {
        dfs(node.left, 新纸条);
        dfs(node.right, 新纸条);
    }
    path.remove(path.size() - 1);                        // 出门:擦掉我(Java没有.pop(),这样删最后一个)
}
```

**两个Java专属新坑**:
1. **没有 `path[:]` 切片快照!** 要用 `new ArrayList<>(path)`——**拷贝构造器**,传入一个现有集合，造一份新的独立副本。直接 `res.add(path)` 存的是"同一个path的引用"，path后面被清空/修改，res里存的也会跟着变空——和Python忘记切片是同一个坑，只是Java的解法语法不同。
2. **List没有 `.pop()`！** 删除"最后一个元素"要写 `path.remove(path.size() - 1)`（先算出最后一个的下标，再删它）。

**257(路径拼字符串)的额外麻烦**：Python `"->".join(path)` 直接拼；Java的`String.join("->", list)`要求`list`里装的是`String`不是`Integer`，所以path要存`List<String>`（append时`path.add(String.valueOf(node.val))`），或者用`StringBuilder`手动拼接。

### 已刷题
| 题 | 纸条 | 叶子处 | 收集格式 |
|---|---|---|---|
| **257** 所有路径 | 无(只path) | 无条件收 | `String.join("->", path)`(path要存String) |
| **113** 路径总和II | remaining | `remaining == node.val` 才收 | `new ArrayList<>(path)`(数字列表,不join!) |

**113 = 257的骨架 + 112的结账**——两派零件各取一半的合体题。
收集格式看返回值类型:`List<List<Integer>>`→`new ArrayList<>(path)`;`List<String>`→join。

---

## 五、辅助函数的判断("形状匹配"规则)

> **递归前对形状:主函数的参数,够不够递归每层要传的?**
> - 够 → 直接递归自己(100:p,q正好一对;112:root,targetSum正好节点+账)
> - 不够 → 写辅助函数,参数按递归需要的来(101:主函数只有root,递归要一对→isMirror(a,b))
> - **拿不准就写辅助函数——永远不会错**。Java里不涉及Python那个"self.还是内部def"的选择——辅助函数直接写成类的另一个`private`方法即可，天然能互相调用。

---

## 六、改结构必用"接住重挂"(分治的实战应用)

**凡是会改变树结构的操作(删除/插入/翻转/剪枝),必须用分治:dfs返回"处理好的子树根",父亲用 `root.left = dfs(root.left)` 接住重挂。**Java写法完全一样，`root.left = dfs(root.left);`一字不改。

**为什么遍历做不到**:遍历没有向上交付的通道,深处对node的重新赋值只改了局部变量,树上的指针(父亲的.left/.right)没跟着动,"接不上"。
**分治能做到**:父亲拿返回值覆盖自己的指针,不管下面发生了什么变化,`root.left = 新的` 一次赋值就完成接线。

普通二叉树里226(翻转)就是这个模式(Python `root.left, root.right = right, left` 再 `return root`；**Java没有一行互换写法**，要么用临时变量`TreeNode temp = left; left = right; right = temp;`再赋值，要么分两行`root.left = right; root.right = left;`——顺序要小心，别先改了left导致right那行用错值，建议先都存到临时变量再赋值);BST的701/450/669/776是这个模式的深度应用,完整讲解见 binary_search_tree_notes.md 第七节。

---

## 七、⚠️ 易错点(全是实战踩的)

1. **两派零件不混装**:遍历骨架里不写 `left = dfs(...)` 和 `or`(257踩过);分治里没有res和pop。
2. **Java不用像Python纠结self./内部def**——类内方法互调直接写方法名，天然可见。
3. **叶子判断别丢**:112没有"是叶子吗"那层,单节点树会误判(正确路径多走一步进空节点)。四段剧本:①我空吗→②我是叶子吗(结账)→③委托孩子→④汇总。
4. **结账用remaining,不再碰targetSum**:账一路减在remaining里,叶子只比 `remaining == node.val`;targetSum只在发起时用一次。
5. **纸条 vs 白板**:remaining每层不同→传参;path/res共享→Python不传(闭包)，Java声明成类字段。
6. **Python缩进三查**(内容全对+缩进错=逻辑错,Java背景高发):
   - else和哪个if对齐?(它否定的是那句吗)
   - append和pop**同级**吗?(一进一出一对门)
   - 发起调用和return在主函数层吗?(没滑进dfs肚子里)
7. **`abs(left-right)`的abs别省**——负差抓不到(Java是`Math.abs`)。
8. **空是None不是null(Python)；Java反过来，空就是null，没有None这个词**。"都空"=`not p and not q`(Java:`p==null && q==null`)；"(排除都空后)一空一不空"=`not p or not q`(Java:`p==null || q==null`)，**顺序有讲究**(前一个先拦截)。
9. **分治里没有裸return**——每个出口都要带值(110踩过:底写成裸return,该是return 0)。分治的return后面永远有东西;遍历才有裸return。
10. **Java没有tuple**——需要一次返回多个值时(110的复合信息、776拆两棵树)，用`int[]`（同类型）或自定义小class（不同类型）。
11. **List没有`.pop()`，用`.remove(size()-1)`；没有切片快照，用`new ArrayList<>(path)`拷贝构造。**

### 词组卡(树专用)
| 意思 | Python | Java |
|---|---|---|
| 都空→True | `if not p and not q: return True` | `if (p == null && q == null) return true;` |
| 一空一不空→False | `if not p or not q: return False` | `if (p == null || q == null) return false;` |
| 我是叶子 | `if not node.left and not node.right:` | `if (node.left == null && node.right == null)` |
| 且起来直接交 | `return a and b and c` | `return a && b && c;` |
| 一条就够 | `return left or right` | `return left \|\| right;` |
| 高度拼法 | `return max(left, right) + 1` | `return Math.max(left, right) + 1;` |
| 交换挂回 | `root.left, root.right = right, left` | 用临时变量分两步赋值 |
| 路径收集(快照) | `res.append(path[:])` | `res.add(new ArrayList<>(path));` |
| 路径收集(字符串) | `res.append("->".join(path))` | `res.add(String.join("->", path));`(path要是List\<String\>) |
| 拆包接收tuple | `leftBal, leftH = dfs(node.left)` | `int[] left = dfs(node.left); left[0]...left[1]` |
| List删最后一个 | `path.pop()` | `path.remove(path.size() - 1);` |

---

## 八、刷过的题(总览)

**分治(Python全部完成):**
- [x] **104** Maximum Depth — 模板原题,max+1(白板✓)
- [x] **226** Invert — 交节点,交换挂回+return root
- [x] **100** Same Tree — 布尔,and,两个底(白板✓)
- [x] **101** Symmetric — 100的镜像:交叉比(左vs右)
- [x] **112** Path Sum — 布尔,or,叶子结账remaining==val(白板✓)
- [x] **110** Balanced — -1暗号 或 tuple版(白板✓,两种写法都会)

**遍历(Python全部完成):**
- [x] **144/94/145** 前中后序 — 一行三位置
- [x] **257** Binary Tree Paths — path+join收集+pop
- [x] **113** Path Sum II — 257骨架+112结账的合体

**BST专题(700/701/98/230/235/530/501/450/669/776/270)全部完成** → 详见 binary_search_tree_notes.md

**待刷(Python也还没做):**
- [ ] **102** 层序遍历(BFS+deque)—— 单独开专场练,详见bfs_notes.md
- [ ] 236(普通树的LCA,对比235看BST的优势)
- [ ] 1120 / 549 / 114(进阶,能讲即可)

### Java 重写进度（思路已会，只练语法转换）
- [ ] 104
- [ ] 226
- [ ] 100
- [ ] 101
- [ ] 112
- [ ] 110（两种写法都练：-1版 和 int[]版）
- [ ] 144 / 94 / 145
- [ ] 257（注意path要存String，String.join）
- [ ] 113

---

## 九、复习节奏(给自己的)

- 白板特训已过一批(Python):104/110/100/112全部白板默写成功
- 待白板确认(Python):94/257/226(理解了,还没纸上默过)
- BFS/102:单独安排一个时段,别和递归树题混着学,详见bfs_notes.md