# BFS (广度优先搜索) 刷题笔记

> DFS一条路走到底再回头；**BFS一层一层往外扩，像水波纹**。核心工具：队列（先进先出）。
> 三大应用场景：**层级遍历**、**拓扑排序**、**最短路径**（无权图用BFS本身，有权图用BFS+Heap=Dijkstra）。
> **能用BFS解决的问题，别用DFS去做** 尤其"最短/最少步数"类，BFS第一次碰到符合条件的，就是最近的，天然优势。

---

## 一、写BFS前，先问自己这几个问题（决策清单）

1. **初始的种子是什么？**（单个起点，还是多个起点一起入队——比如"多源BFS"）
2. **队列里存的是什么结构？**（单个节点？坐标(x,y)？还是(权重,节点)这种pair？）
3. **要不要记录层级信息(size)？初始的 level 是 0 还是 -1？**（看题目要不要"第几层/第几步"这个数字）
4. **需不需要 visited 记录访问过的点，防止死循环？**（树天然无环不用；网格/图有环，必须要）

这四问答完，骨架基本就定了——树的层级遍历不需要问1和4（种子固定是root，树没有环），网格/图这两个才是新增的判断项。

---

## 二、树的层级遍历（已完全掌握，Python已刷，Java待做）

**骨架**：队列初始塞根节点 → 每轮while先"拍照"记住这一层有几个（`size = len(queue)`）→ for循环只处理这么多次 → 处理时把孩子塞进队尾（下一层，这次for不会碰到它们）。

```python
queue = collections.deque([root])       # 队列初始化,塞进根节点
while queue:                            # 只要队列不空,还有节点没处理,就继续
    size = len(queue)                   # ★拍照:记住此刻队列里有几个 = 这一层的节点数
    level = []                          # 这一层的值,先建个空列表装
    for _ in range(size):               # 只处理这一层的数量,不多不少
        node = queue.popleft()          # 从队头取一个(先进先出,按层顺序取)
        level.append(node.val)          # 记下这个节点的值
        if node.left:
            queue.append(node.left)     # 有左孩子,塞进队尾,留给下一轮(下一层)处理
        if node.right:
            queue.append(node.right)    # 有右孩子,同样塞进队尾
    res.append(level)                   # 这一层处理完,整个level存进res
```

**课程笔记的两个提醒**：
- 如果要处理"空节点也入队"的写法（`if curr:` 判断在弹出之后），要注意**收集结果前判断temp/level是不是空**，最后一轮可能全是None导致空list。
- 层数(step/level)什么时候+1：**在整层处理完之后**才加，不是处理每个节点时加。

**已刷题（Python，全部完成）**：102（基础层序）、107（自底向上，翻转res）、199（右视图，只存每层最后一个）、103（锯齿形，奇偶层反转）、515（每层最大值）、1161（哪层sum最大）。
**Java版**：还没写，思路和树的递归Java化一样——骨架照搬，`Queue<TreeNode> queue = new LinkedList<>();`，`.offer()`/`.poll()`对应`.append()`/`.popleft()`。

---

## 三、网格坐标类BFS（全新，课程模板，还没刷）

**和树BFS的两个新增麻烦**：
1. **没有.left/.right了，"孩子"变成上下左右四个方向的坐标运算**
2. **会有环（走来走去绕回起点），必须用visited记录访问过的格子，不然死循环**

### Python 模板
```python
def bfs(self, grid, queue, visited):
    step = -1                                          # 步数计数器,初始-1(具体值看题意)
    while len(queue) > 0:                              # 队列不空,继续往外扩
        size = len(queue)                              # 拍照:这一步能到达的格子数
        for _ in range(size):                          # 只处理这一层(这一步)的格子
            x, y = queue.popleft()                     # 取出一个坐标,拆成x和y
            for dx, dy in [(0,1),(0,-1),(1,0),(-1,0)]:  # 四个方向的偏移量(右/左/下/上)
                newx, newy = x + dx, y + dy             # 算出新坐标
                if self.isValid(newx, newy, grid, visited):   # 新坐标合法才处理
                    visited.add((newx, newy))           # 标记访问过,防止重复入队
                    queue.append((newx, newy))          # 塞进队尾,下一轮(下一步)处理
        step += 1                                       # 这一步扩散完,步数+1
    return step

def isValid(self, x, y, grid, visited):
    return (0 <= x < len(grid)              # x没越界(行范围内)
            and 0 <= y < len(grid[0])       # y没越界(列范围内)
            and (x, y) not in visited       # 没有访问过
            and grid[x][y] == 1)            # 这个格子本身合法(比如是陆地)
```

### Java 模板
```java
public int bfs(Queue<Pair> queue, int[][] grid, boolean[][] visited) {
    int[] rowMovement = {-1, 0, 1, 0};       // 上下左右,行方向的偏移量
    int[] colMovement = {0, 1, 0, -1};       // 对应的列方向偏移量(和上面按下标一一配对)
    int step = -1;                           // 步数计数器
    while (queue.size() > 0) {               // 队列不空,继续往外扩
        int size = queue.size();             // 拍照:这一步能到达的格子数
        for (int count = 0; count < size; count++) {   // 只处理这一层(这一步)的格子
            Pair curr = queue.poll();        // 取出一个坐标
            for (int i = 0; i < 4; i++) {     // 依次试四个方向
                int newX = curr.x + rowMovement[i];   // 算出新坐标的行
                int newY = curr.y + colMovement[i];   // 算出新坐标的列
                if (this.isValid(newX, newY, grid, visited)) {   // 新坐标合法才处理
                    visited[newX][newY] = true;        // 标记访问过,防止重复入队
                    queue.offer(new Pair(newX, newY)); // 塞进队尾,下一轮处理
                }
            }
        }
        step += 1;                            // 这一步扩散完,步数+1
    }
    return step;
}

public boolean isValid(int x, int y, int[][] grid, boolean[][] visited) {
    return x >= 0 && x < grid.length          // x没越界(行范围内)
           && y >= 0 && y < grid[0].length    // y没越界(列范围内)
           && !visited[x][y]                  // 没有访问过
           && grid[x][y] == 1;                // 这个格子本身合法(比如是陆地)
}
```

**四个方向的写法是网格BFS/DFS的固定起手式**：Python用坐标tuple列表`[(0,1),(0,-1),(1,0),(-1,0)]`，Java常用两个平行数组`rowMovement[]`/`colMovement[]`配合for循环，效果一样。

### isValid 四个检查点（顺序：越界→visited→是否合法值，别漏任何一条）
1. `x`/`y` 有没有越界（`0<=x<行数`，`0<=y<列数`）
2. 这个格子是不是已经访问过（visited）
3. 这个格子本身值是否合法（比如`grid[x][y]==1`，是陆地才能走）

### visited 两种更新时机（课程笔记的重点，容易踩坑）
- **入队的时候更新**：加进队列那一刻就标记visited=true（这份模板用的是这种）。**优点**：防止同一个格子被"重复塞进队列"（不然还没被处理就可能又被别的方向指向它、再塞一次）。
- **出队的时候更新**：弹出来处理时才标记。**这种写法必须多一步检查**：入队前不仅要查visited，还要查"队列里是不是已经有它了"，否则同一个格子会被塞进队列多次。
- **起点的visited要提前标记好**（在造queue、塞进第一个种子的同时，就把起点标记为已访问），别漏了这一步。

### 待刷题（全部还没做，按这个顺序刷）
- **200 岛屿数量**：每个未访问的'1'当种子，BFS把整片相连的'1'"淹没"标记掉，扫完整个网格数种子被用了几次。
- **994 腐烂的橘子**：**多源BFS**——所有腐烂橘子（不止一个）同时作为初始种子一起入队，一层一层"腐烂扩散"，层数就是需要的时间。
- **286 墙和门**：多源BFS的变体，多个门同时当种子，从门往外扩，记录每个房间离最近门的距离。
- **490 迷宫**：网格BFS的变体，注意小球是"一直滚到撞墙才停"，不是走一格——移动逻辑要按题目改。

---

## 四、拓扑排序（全新，课程只列了主题，模板待补）

**用于**：有依赖关系的问题（"先修课"这种），判断能不能把所有任务排出一个不冲突的顺序。

**核心思路（Kahn算法，本质是BFS）**：
1. 统计每个节点的"入度"（有多少个前置依赖指向它）
2. 把**入度为0**的节点（没有任何前置依赖）全部作为初始种子入队
3. 每次弹出一个节点，把它"处理掉"，同时把它指向的节点的入度都减1——**入度减到0的，说明它的依赖都处理完了，入队**
4. 全部处理完，能形成一个合法顺序；如果还有节点入度没到0过（说明有循环依赖），则不可能

**待刷**：207（课程表）——判断能不能修完所有课，就是"能不能形成合法拓扑序"。

---

## 五、BFS + Heap：Dijkstra（全新，课程模板，还没刷）

**什么时候需要它，而不是普通BFS**：普通BFS假设"每一步的代价都一样"（比如网格走一步就是1）。**如果边有不同的权重**（比如飞机票价、路网距离不一样），普通BFS的"层数"不再代表真实的最短距离——这时候要用**优先队列(heap)**代替普通队列，**每次弹出"当前代价最小"的节点**，保证第一次到达某点时的代价就是真正的最短。

**核心区别**：普通BFS用`Queue`（先进先出）；Dijkstra用`PriorityQueue`/heap（每次弹出代价最小的），配合一个`visited`（记录每个点"确定的最短代价"）。

### Python 模板
```python
def dijkstra(self, heap, n, graph):
    visited = {}                                    # 记录每个点"已确定"的最短代价
    while len(heap) > 0:                            # heap不空就继续
        curr, node = heapq.heappop(heap)             # 弹出当前代价最小的(curr=代价,node=节点)
        if node in visited and visited[node] < curr:
            continue                                 # 已有更小的记录,这条是过时数据,跳过不处理
        visited[node] = curr                         # 记下这个点当前已知的最短代价
        if len(visited) == n:                        # 所有点都确定了最短代价
            return curr                               # 直接返回(最后确定的这个curr就是答案)
        for val, nextNode in graph[node]:             # 遍历当前点能到达的所有邻居
            if nextNode not in visited:               # 邻居还没确定最短代价
                heapq.heappush(heap, (val + curr, nextNode))  # 把"到邻居的新代价"放进heap候选
    return -1                                          # heap空了还没覆盖所有点,说明到不了
```

### Java 模板
```java
public int dijkstra(Map<Integer, List<MyPair>> graph, int n, int k) {
    // 优先队列,按val(代价)从小到大弹出
    Queue<MyPair> heap = new PriorityQueue<>((p1, p2) -> p1.val - p2.val);
    heap.add(new MyPair(0, k));                      // 起点k,代价0,作为初始种子放入
    Map<Integer, Integer> visited = new HashMap<>(); // 记录每个点已确定的最短代价
    while (heap.size() > 0) {                        // heap不空就继续
        MyPair curr = heap.poll();                   // 弹出当前代价最小的一条记录
        if (visited.containsKey(curr.node) && visited.get(curr.node) < curr.val) {
            continue;                                 // 已有更小的记录,这条过时,跳过
        }
        visited.put(curr.node, curr.val);            // 记下这个点当前已知的最短代价
        if (visited.size() == n) {                    // 所有点都确定了
            return curr.val;                          // 返回答案
        }
        for (MyPair pair : graph.getOrDefault(curr.node, new ArrayList<>())) {  // 遍历邻居
            if (!visited.containsKey(pair.node)) {    // 邻居还没确定
                heap.add(new MyPair(pair.val + curr.val, pair.node));  // 新代价放入候选
            }
        }
    }
    return -1;                                         // 到不了所有点
}
```

**新语法点**：
- Python的heap是`heapq`模块+元组（元组第一项自动被当排序依据）；Java要显式造`PriorityQueue`，传一个`Comparator`（`(p1,p2) -> p1.val - p2.val`表示"val小的排前面"）
- `if node in visited and visited[node] < curr: continue`——这行是精髓：**heap里可能存在同一个node的多条"过时"记录**（之前用更大代价放进去过），弹出时如果发现"已经有更小的记录"，说明这条是废弹出的旧账，直接跳过，不处理

**待刷**：743（网络延迟时间）、787（K站中转内最便宜的航班）——这两道是Dijkstra的标准应用。

---

## 六、⚠️ 易错点（课程笔记 + 树BFS阶段已踩过的坑）

1. **层数(step/level)+1的时机**：在整层的for循环处理完之后才加，不是每处理一个节点就加。
2. **level初始值看题意**：有的题从0开始数层，有的题用-1（比如"经过几步"这种，起点不算一步，最后要减掉起点那一层）。
3. **level遍历收集结果时，注意最后一轮temp/level可能是空的**——如果用了"None也入队"的写法，加进res前先判断非空。
4. **visited要不要在入队时就标记**：不标记的话，同一个点可能被多个方向同时指到，重复塞进队列多次，浪费还可能出错。
5. **网格题的越界检查要在visited检查之前**（先确认合法范围内，再查有没访问过，再查值是否合法——顺序错了可能数组越界报错）。
6. **Dijkstra的visited不是"去过就不再看"，是"记录当前已知的最短代价"**，弹出时要对比，旧的过时记录直接跳过，不是无条件当新数据处理。
7. **多源BFS（994/286）：初始种子是所有起点一起入队，不是一个个单独跑BFS**——一起入队才能保证"层数=真实的最短时间/距离"。

---

## 七、Java 词组卡（BFS专用新词）

| 意思 | Java写法 |
|---|---|
| 队列（普通BFS） | `Queue<T> queue = new LinkedList<>();` |
| 队列（存坐标，需要自定义类/记录） | 自定义 `Pair`（或用 `int[]{x,y}`） |
| 入队/出队 | `.offer(x)` / `.poll()` |
| 优先队列(heap)，自定义排序 | `new PriorityQueue<>((a,b) -> a.val - b.val)` |
| 二维visited数组 | `boolean[][] visited = new boolean[rows][cols];` |
| Map取默认值 | `graph.getOrDefault(key, new ArrayList<>())` |

---

## 八、刷过的题 / 待刷（现状核对，别搞混）

**已完成（Python，树层级遍历）**：
- [x] 102、107、199、103、515、1161

**待刷（全新内容，一个没做）**：
- [ ] 200 岛屿数量（网格BFS入门）
- [ ] 994 腐烂的橘子（多源BFS）
- [ ] 286 墙和门（多源BFS变体）
- [ ] 490 迷宫（网格BFS变体，移动规则特殊）
- [ ] 207 课程表（拓扑排序）
- [ ] 743 网络延迟时间（Dijkstra）
- [ ] 787 K站中转内最便宜的航班（Dijkstra）

**Java版**：树层级遍历那批还没转换；网格/拓扑/Dijkstra这批本来就是Python也没写过，学的时候可以两边一起上手。

---

## 九、学习节奏建议

1. **先200**（网格BFS最基础，建立"visited+四方向"的手感）
2. **994**（多源BFS，比200多一个"多个种子一起入队"的新概念）
3. **207**（拓扑排序，和BFS/Dijkstra是完全不同的判断逻辑，值得单独消化）
4. **743或787**（Dijkstra，全场最新的内容，建议放最后，前面的BFS手感建立好再上）
5. 286、490 穿插练，都是200/994的变体，巩固用