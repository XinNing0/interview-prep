# 🧭 二分法（Binary Search)

> 🚀 **核心思想：**
> 在一个有序区间内，每次取中点，判断目标在左半还是右半，从而每次将搜索范围缩小一半。  
> 这种“折半”思想让查找效率从 **O(n)** 提升到 **O(log n)**。

---

## 💡 为什么使用二分法？

- 暴力解：逐个遍历 → **O(n)**  
- 二分法：每轮排除一半元素 → **O(log n)**  
- 能在**有序数组或区间问题**中大幅提高效率。

---

## 🎯 适用场景

- 区间有序（升序或降序）
- 查找：
  - 精确目标值（exact match）
  - 最左边界 / 最右边界
  - 第一个 ≥ target 或 最后一个 ≤ target

---

## 🧠 思考重点

1. **Search 区间是什么？**
   - `[start, end]`（闭区间）还是 `[start, end)`（半开区间）
   - 不同定义影响循环条件和终止判断

2. **你找的 target 是什么？**
   - 精确值  
   - 左边界 / 右边界  
   - 满足条件的最小 / 最大位置  

## 📘 练习记录表（Binary Search Practice Log）

| 日期 | 题号 | 题目 | 难度 | 思路 / 技巧 | 备注 |
|------|------|------|------|--------------|------|
| 2025-11-06 | 704 | Binary Search | 🟢 Easy | 标准模板题，熟悉 start+1<end 写法 | ✅ 掌握 |
| 2025-11-06 | 69 | Sqrt(x) | 🟢 Easy | 条件型二分，比较 mid*mid 与 x | 注意越界处理 |
| 2025-11-06 | 34 | Find First and Last Position of Element in Sorted Array | 🟡 Medium | 左右边界二分法 | ✅ 掌握 |
| 2025-11-07 | 852 | Peak Index in a Mountain Array | 🟡 Medium | mid 与 mid+1 比较方向 | 待复习 |
| 2025-11-07 | 162 | Find Peak Element | 🟡 Medium | 比较 mid 与 mid+1 决定方向 | 🔁 复习 |
| 2025-11-08 | 74 | Search a 2D Matrix | 🟡 Medium | 二维转一维二分 | ✅ 掌握 |
| 2025-11-09 | 744 | Find Smallest Letter Greater Than Target | 🟢 Easy | 环形二分，若 target ≥ 最后元素返回 letters[0]；注意 `start + 1 < end` 写法 | ✅ 掌握 |
| 2025-11-09 | 540 | Single Element in a Sorted Array | 🟡 Medium | 利用索引奇偶性判断单个元素位置（`mid ^ 1` 技巧） | 🔁 待复习 |
| 2025-11-09 | 278 | First Bad Version | 🟢 Easy | 模板题，典型“找第一个 True”边界型二分 | ✅ 掌握 |
| 2025-11-09 | 275 | H-Index II | 🟡 Medium | 在升序引用次数中二分找满足 `citations[mid] >= n - mid` 的最小 mid | ⚠️ 待巩固 |
| 2025-11-09 | 33 | Search in Rotated Sorted Array | 🟡 Medium | 分段有序二分，判断左段/右段是否有序后决定方向 | ⚠️ 待巩固 |
| 2025-11-09 | 240 | Search a 2D Matrix II | 🟡 Medium | 每走一步排除一整行/列 | ⚠️ 待巩固 |


> 💡 每练完一道题就追加一行  
> “思路 / 技巧” 写核心思想或坑点  
> “备注” 标明掌握程度（✅ 掌握 / ⚠️ 待巩固 / 🔁 复习）

---


---

## 🐍 Python 模板（含完整注释）

```python
def binarySearch(self, nums: List[int], target: int) -> int:
    """
    二分查找模板 (Python)
    :param nums: 有序数组
    :param target: 目标值
    :return: 目标值下标，如果不存在返回 -1
    """

    # 定义左右边界
    start = 0
    end = len(nums) - 1

    # 循环条件：保证区间内至少有两个元素
    # 使用 start + 1 < end 避免死循环
    while start + 1 < end:
        # 计算中点（整除）
        mid = (start + end) // 2

        # 若中点值 < 目标，说明目标在右侧
        if nums[mid] < target:
            start = mid

        # 若中点值 > 目标，说明目标在左侧
        elif nums[mid] > target:
            end = mid

        # 若 nums[mid] == target，直接返回索引
        else:
            return mid

    # 检查剩余的两个边界
    if nums[start] == target:
        return start
    if nums[end] == target:
        return end

    # 若都不是，返回 -1 表示未找到
    return -1

# 测试 
if __name__ == "__main__":
    cases = [
        ([1, 3, 5, 7, 9], 7, 3),     # 正常命中
        ([1, 3, 5, 7, 9], 2, -1),    # 不存在
        ([], 1, -1),                 # 空数组
        ([1], 1, 0),                 # 单元素命中
        ([1], 0, -1),                # 单元素不命中
    ]
    for arr, t, expected in cases:
        got = binary_search(arr, t)
        print(arr, t, "=>", got, "| expected:", expected)
```
---

## Java 模板（含完整注释）
```java 

public class BinarySearch {
    /**
     * 二分查找模板 (Java)
     * @param nums 有序数组
     * @param target 目标值
     * @return 目标索引，若不存在返回 -1
     */
    public int search(int[] nums, int target) {
        // 边界检查：数组为空或长度为 0，直接返回 -1
        if (nums == null || nums.length == 0) {
            return -1;
        }

        // 初始化左右边界
        int start = 0;
        int end = nums.length - 1;

        // 循环条件：保证至少有两个元素
        // 防止死循环，退出时 start 和 end 相邻
        while (start + 1 < end) {
            // 计算中点，避免溢出写法
            int mid = start + (end - start) / 2;

            // 若中点值 < 目标，搜索右侧
            if (nums[mid] < target) {
                start = mid;
            }
            // 若中点值 > 目标，搜索左侧
            else if (nums[mid] > target) {
                end = mid;
            }
            // 若找到目标，直接返回
            else {
                return mid;
            }
        }

        // 检查剩余的两个边界位置
        if (nums[start] == target) {
            return start;
        }
        if (nums[end] == target) {
            return end;
        }

        // 若未找到，返回 -1
        return -1;
    }

    // ✅ 测试样例
    public static void main(String[] args) {
        BinarySearch bs = new BinarySearch();
        int[] nums = {1, 3, 5, 7, 9};
        int target = 7;
        int index = bs.search(nums, target);
        System.out.println(index);  // 输出 3
    }
}

```



