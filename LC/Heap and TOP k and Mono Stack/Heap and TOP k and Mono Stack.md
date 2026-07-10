# 🧭 堆与 Top K 问题 (Heap / Priority Queue)

> 🚀 **核心思想：**
> 堆是一种特殊的完全二叉树。**大顶堆**保证根节点最大，**小顶堆**保证根节点最小。  
> 它能以 **O(1)** 取极值，以 **O(log n)** 插入或删除，是处理“动态最值”和“前 K 个”问题的核心工具。

---

## 💡 为什么使用 Heap？

- **全量排序：** 需要 **O(N log N)**。
- **Heap 优化：** 维护一个大小为 K 的堆，复杂度降至 **O(N log K)**。
- **流式处理：** 堆不需要一次性加载所有数据，非常适合处理海量数据流（Data Stream）。

---

## 🎯 适用场景

- **Top K 问题：** 找最大 K 个（用小顶堆）、最小 K 个（用大顶堆）。
- **合并 K 个有序序列：** 如 Merge K Sorted Lists。
- **中位数查找：** 双堆法（对顶堆）。
- **优先级调度：** 总是处理当前优先级最高/低的任务。

---
## 💡 什么时候用 PQ？

- **只要看到**“最大/最小”且“集合在变”**，首选 PQ。

- **只要看到**“Top K”**，首选 PQ。

- **如果需要**“全局有序”，用 Arrays.sort()；如果只需要“局部最值”**，用 PriorityQueue。

---

## 🧠 思考重点 (Interview Pitfalls)

1. **“找大用小，找小用大”：**
   - 找**最大** K 个数 $\rightarrow$ 维持 **Min-Heap**。堆满时踢掉最小的，剩下的就是最大的。
   - 找**最小** K 个数 $\rightarrow$ 维持 **Max-Heap**。堆满时踢掉最大的。
2. **比较器 (Comparator) 逻辑：**
   - Java 中 `(a, b) -> a - b` 是小顶堆（默认）。
   - Java 中 `(a, b) -> b - a` 是大顶堆。
   - 如果频率相同要按字母序，逻辑会变复杂，需特别注意。

   - 找第 K 大： 用小顶堆，poll 掉最小值，剩下的都是大值，堆顶是**“大值里的最小”**（即第 K 大）。
   - PriorityQueue<Integer> pq = new PriorityQueue<>();
   - 找第 K 小： 用大顶堆，poll 掉最大值，剩下的都是小值，堆顶是**“小值里的最大”**（即第 K 小）。
   - PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> b - a);

---   

## 📘 练习记录表 (Heap Practice Log)

| 日期 | 题号 | 题目 | 难度 | 思路 / 技巧 | 备注 |
|------|------|------|------|--------------|------|
| 2026-03-01 | 215 | Kth Largest Element in an Array | 🟡 Medium | 基础 Top K，维持大小为 K 的小顶堆 | ✅ 掌握 |
| 2026-03-01 | 347 | Top K Frequent Elements | 🟡 Medium | HashMap 计数 + Heap 排序 | ✅ 掌握 |
| 2026-03-02 | 692 | Top K Frequent Words | 🟡 Medium | 难点：频率相同时的字母序处理 | ⚠️ 待巩固 |
| 2026-03-02 | 23 | Merge k Sorted Lists | 🔴 Hard | 多指针+堆维护最小值 | 🔁 复习 |
| 2026-03-03 | 295 | Find Median from Data Stream | 🔴 Hard | 对顶堆：大顶堆存左半，小顶堆存右半 | ⚠️ 难点 |

---

## 🐍 Python 模板

Python 默认只提供小顶堆（`heapq`）。

```python
import heapq
from collections import Counter

def topKFrequent(nums, k):
    # 1. 统计频率 O(N)
    count = Counter(nums)
    
    # 2. 维护大小为 k 的小顶堆 O(N log K)
    # heap 存储形式为 (freq, num)
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
            
    # 3. 提取结果
    return [item[1] for item in heap]
```
---

## Java 模版

Java 的 PriorityQueue 是 FT SDE 面试中最常被要求手写的 API。

```java
import java.util.*;

public class HeapSolution {
    /**
     * Top K Frequent Words 完整实现 (包含频率与字典序综合排序)
     */
    public List<String> topKFrequent(String[] words, int k) {
        // Step 1: 计数 - HashMap 记录频率 O(N)
        Map<String, Integer> freqMap = new HashMap<>();
        for (String w : words) {
            freqMap.put(w, freqMap.getOrDefault(w, 0) + 1);
        }

        // Step 2: 初始化优先队列 (小顶堆) O(N log K)
        // 规则：
        // 1. 如果频率不同，频率小的排在堆顶 (升序)
        // 2. 如果频率相同，字母序大的排在堆顶 (降序)，以便 poll 掉字母序靠后的
        PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> {
            int f1 = freqMap.get(a);
            int f2 = freqMap.get(b);
            if (f1 == f2) {
                return b.compareTo(a); 
            }
            return f1 - f2;
        });

        // Step 3: 维持堆的大小为 k
        for (String word : freqMap.keySet()) {
            pq.offer(word);
            if (pq.size() > k) {
                pq.poll(); // 弹出堆顶，留下频率更高的
            }
        }

        // Step 4: 输出结果并反转 O(K log K)
        List<String> res = new ArrayList<>();
        while (!pq.isEmpty()) {
            res.add(pq.poll());
        }
        Collections.reverse(res); // 小顶堆出来的顺序是从小到大，需要反转
        return res;
    }
}

```
