# 🗺️ 图论（Graph BFS / DFS）

> 🚀 **核心思想：**
> 图是由**节点（Node）**和**边（Edge）**组成的数据结构。  
> BFS 像水波纹一样**一层一层往外扩**，适合求最短路径/最少步数。  
> DFS 像走迷宫一样**一条路走到底再回头**，适合找所有路径/连通性。

---

## 💡 图的三种表示方式

| 表示方式 | 输入形式 | 找邻居方法 |
|----------|----------|------------|
| 矩阵（二维数组） | `grid / board / image` | 四方向 `[(0,1),(0,-1),(1,0),(-1,0)]` |
| 邻接表 | `edges = [[0,1],[1,2]]` | `graphs[curr]` |
| 邻接矩阵 | `isConnected[i][j]` | `for nei in range(n) if matrix[curr][nei]==1` |

---

## 🎯 适用场景

| 场景 | 用哪个 |
|------|--------|
| 最短路径 / 最少步数 / 最短时间 | BFS |
| 所有路径 | DFS |
| 连通块数量 | BFS 或 DFS |
| 层级距离 | BFS |
| 是否有环 | DFS |
| 扩散 / 传播 | BFS |

---

## 🧠 判断起点：单起点 vs 多起点

> **看描述定起点，看输入定方向**

| 题目描述 | 起点类型 |
|----------|----------|
| "从 (sr, sc) 开始" | 单起点 |
| "所有 X 同时扩散" | 多起点，扫描所有 X 入队 |
| "边界上的 X" | 多起点，扫描四条边 |
| "找最近的 X" | 多起点，所有 X 同时出发 |

---

## 📘 练习记录表（Graph Practice Log）

| 日期 | 题号 | 题目 | 难度 | 起点类型 | 思路 / 技巧 | 备注 |
|------|------|------|------|----------|--------------|------|
| | 733 | Flood Fill | 🟢 Easy | 单起点 | 固定(sr,sc)出发，邻居颜色==original才扩 | ✅ 掌握 |
| | 200 | Number of Islands | 🟡 Medium | 多次单起点 | 扫描所有格子，遇到没访问的1启动BFS，count+1 | ✅ 掌握 |
| | 695 | Max Area of Island | 🟡 Medium | 多次单起点 | BFS内部记area，外层取max | ✅ 掌握 |
| | 994 | Rotting Oranges | 🟡 Medium | 多起点 | 所有2同时入队，记time，最后检查fresh==0 | ✅ 掌握 |
| | 542 | 01 Matrix | 🟡 Medium | 多起点 | 所有0同时入队，dist矩阵记距离 | ✅ 掌握 |
| | 1162 | As Far from Land | 🟡 Medium | 多起点 | 所有1同时入队，返回max(dist) | ✅ 掌握 |
| | 130 | Surrounded Regions | 🟡 Medium | 边界多起点 | 边界O入队，BFS标记安全，最后没visited的O改X | ✅ 掌握 |
| | 547 | Number of Provinces | 🟡 Medium | 多次单起点 | 邻接矩阵建图，连通块计数 | |
| | 261 | Graph Valid Tree | 🟡 Medium | 单起点 | 判断有环 + 连通 | |
| | 133 | Clone Graph | 🟡 Medium | 单起点 | BFS + HashMap复制节点 | |
| | 127 | Word Ladder | 🔴 Hard | 单起点 | BFS + 枚举每个字符替换 | |
| | 417 | Pacific Atlantic Water Flow | 🟡 Medium | 两组多起点 | 两次BFS反向从边界出发 | |

> 💡 每练完一道题就追加一行  
> "思路 / 技巧" 写核心思想或坑点  
> "备注" 标明掌握程度（✅ 掌握 / ⚠️ 待巩固 / 🔁 复习）

---

## 🐍 Python 模板

### 矩阵 BFS（最常用）

```python
import collections

def matrixBFS(grid):
    rows = len(grid)
    cols = len(grid[0])
    directions = [(0,1),(0,-1),(1,0),(-1,0)]
    visited = set()

    def bfs(r, c):
        queue = collections.deque([(r, c)])
        visited.add((r, c))
        while queue:
            curr_r, curr_c = queue.popleft()
            for dr, dc in directions:
                nr, nc = curr_r + dr, curr_c + dc
                if 0 <= nr < rows and 0 <= nc < cols \
                and grid[nr][nc] == 1 \          # 条件根据题目改
                and (nr, nc) not in visited:
                    visited.add((nr, nc))
                    queue.append((nr, nc))

    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1 and (r, c) not in visited:
                bfs(r, c)
                count += 1
    return count
```

### 多起点 BFS（994 / 542 风格）

```python
import collections

def multiSourceBFS(grid):
    rows = len(grid)
    cols = len(grid[0])
    directions = [(0,1),(0,-1),(1,0),(-1,0)]

    # 第一步：所有起点同时入队
    queue = collections.deque()
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 起点条件:
                queue.append((r, c))

    # 第二步：BFS扩散
    time = 0
    while queue:
        time += 1
        for _ in range(len(queue)):      # 处理这一层
            curr_r, curr_c = queue.popleft()
            for dr, dc in directions:
                nr, nc = curr_r + dr, curr_c + dc
                if 0 <= nr < rows and 0 <= nc < cols \
                and 满足条件:
                    # 更新状态
                    queue.append((nr, nc))

    return time
```

### 邻接表 BFS（图题通用）

```python
import collections

def graphBFS(edges, n):
    # 建图
    graphs = collections.defaultdict(list)
    for edge in edges:
        graphs[edge[0]].append(edge[1])
        graphs[edge[1]].append(edge[0])  # 无向图双向加

    # BFS
    queue = collections.deque([0])
    visited = {0}

    while queue:
        curr = queue.popleft()
        for nei in graphs[curr]:
            if nei not in visited:
                visited.add(nei)
                queue.append(nei)

    return len(visited) == n
```

### DFS 模板（找所有路径）

```python
def dfs(graph, node, target, path, result):
    if node == target:
        result.append(list(path))
        return
    for nei in graph[node]:
        path.append(nei)
        dfs(graph, nei, target, path, result)
        path.pop()  # 回溯，撤销选择
```

---

## ☕ Java 模板

### 矩阵 BFS

```java
import java.util.*;

public void matrixBFS(int[][] grid) {
    int rows = grid.length;
    int cols = grid[0].length;
    int[][] directions = {{0,1},{0,-1},{1,0},{-1,0}};
    Set<String> visited = new HashSet<>();

    Queue<int[]> queue = new LinkedList<>();
    queue.offer(new int[]{startR, startC});
    visited.add(startR + "," + startC);

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int currR = curr[0], currC = curr[1];

        for (int[] dir : directions) {
            int nr = currR + dir[0];
            int nc = currC + dir[1];
            String key = nr + "," + nc;
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                && grid[nr][nc] == 1                    // 条件根据题目改
                && !visited.contains(key)) {
                visited.add(key);
                queue.offer(new int[]{nr, nc});
            }
        }
    }
}
```

### 邻接表 BFS

```java
import java.util.*;

public void graphBFS(int[][] edges, int n) {
    // 建图
    Map<Integer, List<Integer>> graph = new HashMap<>();
    for (int[] edge : edges) {
        graph.computeIfAbsent(edge[0], k -> new ArrayList<>()).add(edge[1]);
        graph.computeIfAbsent(edge[1], k -> new ArrayList<>()).add(edge[0]);
    }

    // BFS
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    queue.offer(0);
    visited.add(0);

    while (!queue.isEmpty()) {
        int curr = queue.poll();
        for (int nei : graph.getOrDefault(curr, new ArrayList<>())) {
            if (!visited.contains(nei)) {
                visited.add(nei);
                queue.offer(nei);
            }
        }
    }
}
```

### DFS 模板

```java
public void dfs(List<Integer> path, int curr, int target,
                List<List<Integer>> result, Map<Integer, List<Integer>> graph) {
    if (curr == target) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int next : graph.getOrDefault(curr, new ArrayList<>())) {
        path.add(next);
        dfs(path, next, target, result, graph);
        path.remove(path.size() - 1);  // 回溯
    }
}
```

---

## ⚠️ 常见坑点

| 坑点 | 正确做法 |
|------|----------|
| `visited` 要在入队时加，不是出队时 | `visited.add()` 和 `queue.append()` 写在一起 |
| 矩阵越界 | `0 <= nr < rows and 0 <= nc < cols` |
| 多起点忘记同时入队 | 初始化时扫描所有起点 |
| `deque` 用 `popleft()` 不是 `pop()` | BFS固定用 `popleft()` |
| 二维坐标存 `set` 用元组 | `visited.add((r, c))` 不是 `visited.add([r, c])` |