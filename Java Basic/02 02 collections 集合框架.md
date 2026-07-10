
> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 2: 集合框架 (Collections) ⭐ PNC 重点

## 2.1 List vs Set ⭐ 被问到(答对了)

> 中文:
> - **List** 代表**有序**的集合,**允许重复**元素(如 ArrayList、LinkedList)。
> - **Set** 保证**唯一性**,**不能包含重复**元素(如 HashSet、TreeSet)。

**English:** "A List represents an ordered collection that allows duplicates. A Set guarantees uniqueness — it cannot contain duplicate elements."

## 2.2 如何保证 Set 的唯一性 —— equals + hashCode ⭐ 被问到(答对了)

> 中文:要在自定义 Java 类里保证 Set 的唯一性,必须重写 **equals()** 和 **hashCode()** 两个方法。默认情况下 Java 从 Object 类继承这两个方法(基于对象内存地址),所以两个内容相同的对象会被当成不同的。

**English:** "To guarantee uniqueness for a custom object in a Set, you must override both equals() and hashCode(). By default Java inherits these from the Object class based on memory reference."

```java
public class Transaction {
    private String id;
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Transaction t = (Transaction) o;
        return Objects.equals(id, t.id);
    }
    @Override
    public int hashCode() { return Objects.hash(id); }
}
```
> **为什么两个都要重写?** HashSet 先用 hashCode() 找桶(bucket),再用 equals() 比较。只重写 equals 不重写 hashCode,两个相等对象可能落在不同桶里,Set 就判断不出重复。

## 2.3 ArrayList vs LinkedList

| | ArrayList | LinkedList |
|---|---|---|
| 底层 | 动态数组 | 双向链表 |
| 随机访问 get(i) | O(1) 快 | O(n) 慢 |
| 中间插入/删除 | O(n) 慢(要移动) | O(1) 快(改指针) |
| 内存 | 连续,省 | 每个节点存前后指针,费 |

> 中文:随机访问多用 ArrayList,频繁在中间增删用 LinkedList。实际大多数场景 ArrayList 更好(CPU 缓存友好)。

## 2.4 HashMap 底层原理 (高频)

> 中文:
> - 底层是**数组 + 链表 + 红黑树**。
> - put 时用 key 的 hashCode 计算数组下标(桶)。
> - 哈希冲突时,同一个桶里用链表串起来。
> - **Java 8 优化:** 链表长度超过 8 且数组长度 ≥ 64 时,链表转**红黑树**,查询从 O(n) 降到 O(log n)。
> - 默认初始容量 16,负载因子 0.75,超过就扩容(翻倍)。

## 2.5 HashMap vs ConcurrentHashMap

> 中文:HashMap **非线程安全**,多线程 put 可能死循环(Java 7)或数据丢失。ConcurrentHashMap 线程安全:
> - **Java 7:** 分段锁(Segment),把数据分成多段,每段独立加锁。
> - **Java 8:** 放弃分段锁,改用 **CAS + synchronized**,只锁单个桶的头节点,并发度更高。

## 2.6 fail-fast 机制

> 中文:遍历集合时如果被其他线程(或自己)结构性修改,会抛 `ConcurrentModificationException`。这是 fail-fast。想在遍历时删除,要用 Iterator 的 remove() 或 CopyOnWriteArrayList。

---