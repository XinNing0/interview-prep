# 二分查找 (Binary Search) 刷题笔记

> 在"每步能判断答案在哪一半"的问题上,一次砍掉一半。O(log n)。
> **本质不是"数组有序",而是"每步能二选一"**——153(半有序)、875(猜答案)都能二分。
> 正在把已掌握的题从Python换成Java重写,思路不用重推,重点是语法转换。

---

## 一、和对撞双指针的关系(我的理解入口)

二分 = 对撞双指针的近亲:**都是left/right夹一个范围,"小了左边往右,大了右边往左"**。

| | 对撞双指针(167) | 二分(704) |
|---|---|---|
| 每轮砍多少 | 挪1格 → O(n) | 跳到mid,砍一半 → O(log n) |
| while条件 | `left < right`(剩1个配不成对) | `left <= right`(剩1个还要查) |
| 靠什么砍半 | — | 有序/可判断 → mid一比,整半排除 |

> 有序数组+找一对 → 对撞;有序数组+找一个 → 二分。

⚠️ **两者代码长得像(都是left/right + if/else),容易把"动多少"这一件事写串**——我在69把二分写成了`left++`/`right--`(对撞的挪1格），该是`left=mid+1`/`right=mid-1`(二分的跳整半)。写下`left`/`right`的动作之前，先问一句：**"这一半能不能被完全排除，不会漏答案？"** 能→跳(`mid±1`)是二分；不能，只能一步步逼近→挪(`++`/`--`)是对撞。

---

## 二、标准模板:闭区间(只养这一个,别混流派!)

### Python
```python
left, right = 0, len(nums) - 1
while left <= right:                 # 有等号!剩一个候选也要查
    mid = (left + right) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        left = mid + 1               # mid查过了,砍掉它(+1!)
    else:
        right = mid - 1
return -1                            # 循环内查干净,出来即没有
```

### Java
```java
int left = 0, right = nums.length - 1;
while (left <= right) {                        // 有等号!剩一个候选也要查
    int mid = left + (right - left) / 2;       // ★不写(left+right)/2,防止溢出(见下)
    if (nums[mid] == target) {
        return mid;
    } else if (nums[mid] < target) {
        left = mid + 1;                        // mid查过了,砍掉它(+1!)
    } else {
        right = mid - 1;
    }
}
return -1;                                      // 循环内查干净,出来即没有
```

**口诀:`<=` 继续、mid查完就 `±1` 砍掉、出来即-1。**

⚠️ 二分两大流派(`<=`配`mid±1` vs `start+1<end`配`=mid`)**不能混用,混了必死循环**。我统一用闭区间。

⚠️ **Java专属坑:`mid = (left + right) / 2` 有溢出风险!** Python的int没有上限,永远不会溢出;**Java的int有范围上限(约21亿)**,如果`left`和`right`都很大,`left+right`可能超过int上限而出错(变成负数)。**统一写成 `mid = left + (right - left) / 2`**——数学上等价，但不会先算一个可能超范围的和。这是Java二分的经典面试考点，Python不用管这个。

---

## 三、第二形态:找边界(找到 ≠ 停!)

**基础二分找到即return;找边界的二分:找到后记账+继续往一侧压。**

### 34 找target的第一个和最后一个位置

**Python**
```python
def findBound(nums, target, is_left):
    left, right = 0, len(nums) - 1
    bound = -1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            bound = mid                  # 记账:目前找到的
            if is_left:
                right = mid - 1          # 找左界:假装不够好,往左继续压
            else:
                left = mid + 1           # 找右界:往右继续压
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return bound
# 答案 = [findBound(True), findBound(False)],两遍二分还是O(log n)
```

**Java**
```java
private int findBound(int[] nums, int target, boolean isLeft) {
    int left = 0, right = nums.length - 1;
    int bound = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            bound = mid;                  // 记账:目前找到的
            if (isLeft) {
                right = mid - 1;          // 找左界:往左继续压
            } else {
                left = mid + 1;           // 找右界:往右继续压
            }
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return bound;
}
// 答案: new int[]{findBound(nums, target, true), findBound(nums, target, false)}
```
辅助函数用`private`修饰（这个类内部用，不对外暴露）；Java没有`is_left`这种下划线命名习惯，驼峰式写`isLeft`。

**"跑两遍"不是笨,是标准解**——两次O(log n)还是O(log n),各司其职,比"一遍找两头"干净得多。
(面经原题"找第一个位置" = 只要左边界那半。)

### 35 搜索插入位置
找不到时返回该插哪 = **循环结束后left的位置**。704模板最后 `return left;` 即可（Python和Java这里逻辑完全一样，只是外壳）。

---

## 四、第三形态:半有序也能二分(思维升级)

### 153 旋转数组找最小值 🔥
`[4,5,6,7,0,1,2]` → 0。数组不全有序,但**每步仍能判断最小值在哪半**:

**Python**
```python
left, right = 0, len(nums) - 1
while left < right:                  # 注意这题用 <(留下的那个就是答案)
    mid = (left + right) // 2
    if nums[mid] > nums[right]:      # mid比右端大 → 断崖在右半
        left = mid + 1               # 最小值在右
    else:                            # 右半是顺的
        right = mid                  # 最小值在左半(mid自己可能就是,不砍)
return nums[left]
```

**Java**
```java
int left = 0, right = nums.length - 1;
while (left < right) {                          // 注意这题用 <(留下的那个就是答案)
    int mid = left + (right - left) / 2;
    if (nums[mid] > nums[right]) {               // mid比右端大 → 断崖在右半
        left = mid + 1;                          // 最小值在右
    } else {                                     // 右半是顺的
        right = mid;                             // 最小值在左半(mid自己可能就是,不砍)
    }
}
return nums[left];
```

### 33 旋转数组搜索target 🔥(153强化,大厂高频)
每步两层判断:
1. **哪半是顺的?** `nums[left] <= nums[mid]` → 左半顺;否则右半顺
2. **target在顺的那半的范围里吗?** 在→砍进顺的半;不在→去另一半
(顺的那半是正常有序区间,才能用范围判断;策略="查顺的,二选一")
逻辑和153同构，Java翻译时套上面153的外壳即可，多加一层if判断"哪半顺"。

---

## 五、第四形态:二分答案(对象不是数组!)

**在"可能的答案范围"上猜**,配一个可行性验证函数。

### 69 x的平方根(入门)
猜0~x之间的数,`mid*mid` 和 x 比,砍半。Java注意`mid*mid`可能超int范围要小心（比如用`long`存乘积），Python不用担心这个。

### 875 爱吃香蕉的珂珂(标准形态)
求能在h小时吃完的**最小**速度k:

**Python**
```python
# 骨架:二分答案 + 可行性验证
left, right = 1, max(piles)          # 答案范围:速度1 ~ 最大堆
while left <= right:
    mid = (left + right) // 2
    if canFinish(mid):               # 这个速度来得及
        ans = mid; right = mid - 1   # 记账,试更小的 ← 就是34找左边界的动作!
    else:
        left = mid + 1               # 太慢,加速

# 可行性:每堆耗时=向上取整(pile/k),总和<=h?
```

**Java**
```java
int left = 1, right = 0;
for (int pile : piles) right = Math.max(right, pile);   // Java数组没有max()内建方法,手动找最大值
int ans = right;
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (canFinish(piles, mid, h)) {          // 这个速度来得及
        ans = mid;
        right = mid - 1;                     // 记账,试更小的
    } else {
        left = mid + 1;                      // 太慢,加速
    }
}
// 可行性:每堆耗时 = (pile + mid - 1) / mid（整数向上取整技巧，和Python的math.ceil效果一样）
```
**Java坑**：数组求最大值没有像Python `max(piles)` 这样的一步到位写法，要么手写循环找最大，要么用`Arrays.stream(piles).max().getAsInt()`。

**求"最小可行值" = 在答案空间上找左边界** —— 34的动作换了个空间。

### 74 搜索二维矩阵(降维)
把矩阵当拉直的一维数组二分,mid换算坐标——这两行语法完全一样，Python和Java没有差异：
```
row = mid // cols(Python) / mid / cols(Java写法一致，都是整数除法)
col = mid % cols
```

---

## 六、⚠️ 易错点

1. **`while left <= right` 的等号别丢**(闭区间流派)——丢了漏查最后一个候选。
2. **`left = mid + 1` / `right = mid - 1` 的±1别丢**——mid已查过,不砍会死循环。(153是例外:`right = mid` 因为mid可能就是答案,配 `left < right`。)
3. **两个流派不能混**:`<=`必配`±1`;`start+1<end`必配`=mid`+循环后补查。
4. **找边界:找到不return!** 记bound,继续往目标侧压。
5. **`mid = (left + right) // 2` 别忘写**(草稿里真漏过)。
6. **判断相等用 `==` 不是 `=`**。
7. 二分答案题:先想清**答案的范围**(875:1~max堆)和**可行性怎么验证**,再套骨架。

### Java专属新坑
8. **`mid = (left + right) / 2` 有溢出风险**——统一写 `mid = left + (right - left) / 2`。
9. **数组没有内建`max()`/`min()`**——要么手写循环，要么`Arrays.stream(arr).max().getAsInt()`。
10. **向上取整**：整数技巧 `(a + b - 1) / b` 和Python一样能用；如果用`Math.ceil()`，参数必须先转成`double`（`Math.ceil((double) a / b)`），否则`a/b`会先做整数除法把小数部分截断，`ceil`已经来不及起作用。

### 词组卡
| 意思 | Python | Java |
|---|---|---|
| 取中(防溢出) | `mid = (left + right) // 2` | `mid = left + (right - left) / 2` |
| 向上取整 | `math.ceil(a/k)` 或 `(a + k - 1) // k` | `(a + k - 1) / k` 或 `Math.ceil((double)a/k)` |
| 二维拉直换算 | `row = mid // cols; col = mid % cols` | 语法相同 |
| 数组求最大值 | `max(arr)` | 手写循环 或 `Arrays.stream(arr).max().getAsInt()` |
| 找最小可行 | 可行→记账+`right=mid-1`;不可行→`left=mid+1` | 同 |

---

## 七、什么时候想到二分

- **有序数组找一个/找位置** → 基础二分(704/35)
- **有序+有重复,找第一个/最后一个** → 找边界(34)
- **旋转/半有序** → 判断哪半顺(153/33)
- **"最小的满足条件的值"/"最大的可行值"+能写出验证函数** → 二分答案(875)
- 关键词:O(log n)要求、sorted、"minimum k such that..."

---

## 八、刷过的题

- [x] **704** Binary Search — 闭区间模板
- [x] **35** Search Insert Position — 循环后left即插入位
- [x] **34** Find First and Last Position — 找边界,记账+继续压(面经"找第一个位置"=左半)
- [x] **69** Sqrt(x) — 二分答案入门
- [x] **74** Search a 2D Matrix — 拉直+坐标换算
- [x] **153** Find Minimum in Rotated Sorted Array — 半有序,和右端比
- [x] **33** Search in Rotated Sorted Array — 153+两层判断
- [ ] **875** Koko Eating Bananas — 二分答案标准形态(待做)

### Java 重写进度（思路已会，只练语法转换）
- [ ] 704
- [ ] 35
- [ ] 34
- [ ] 153
- [ ] 33
- [ ] 74 / 69 / 875（875Python也还没做，可以Python/Java一起上手）