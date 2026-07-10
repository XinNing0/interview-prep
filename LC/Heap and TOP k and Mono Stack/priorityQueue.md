## 💡 PriorityQueue (PQ) 终极指南

### 1. 数组模式 (高效)
* `PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);`
* 适用：多字段排序（如和、索引、坐标）。

### 2. 外部关联模式 (省空间)
* `PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> map.get(a) - map.get(b));`
* 适用：根据频率排序，堆内不存频率值。

### 3. 多重判定模式
* `if (f1 != f2) return f1 - f2; else return b.compareTo(a);`
* 适用：692 题，频率相同看字母序。

### ⚡ 核心口诀
* **找前 K 大：** 用小顶堆 `(a, b) -> a - b`，踢掉小的，留下大的。
* **找前 K 小：** 用大顶堆 `(a, b) -> b - a`，踢掉大的，留下小的。