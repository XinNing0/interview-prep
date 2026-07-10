*中英对照 | 详细讲解 | 面试可直接使用*

> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 10: 数据库 & JPA/Hibernate ⭐⭐ 2026 高频

> 搜索发现:N+1 查询问题是"最能区分真实生产经验和教程级知识"的问题。senior 岗位必问。

## 10.1 N+1 查询问题 ⭐⭐ 高频

> 中文:当你查询一个列表(1 次查询),然后对每个元素又触发一次关联查询(N 次),总共 N+1 次查询,性能灾难。

```java
// 问题:查 100 个 order,每个再查一次 items → 101 次查询
List<Order> orders = orderRepository.findAll();
for (Order o : orders) {
    o.getItems().size();  // 每次触发一条 SQL
}
```

**解决方法:**
```java
// 方法1:JOIN FETCH 一次查完
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();

// 方法2:@EntityGraph
@EntityGraph(attributePaths = {"items"})
List<Order> findAll();
```

## 10.2 Lazy vs Eager 加载

> 中文:
> - **Lazy(懒加载):** 关联数据用到时才查。默认 @OneToMany 是 lazy。省内存但可能触发 N+1。
> - **Eager(急加载):** 查主对象时立即把关联数据一起查出来。默认 @ManyToOne 是 eager。

## 10.3 乐观锁 vs 悲观锁

> 中文:
> - **乐观锁(Optimistic):** 假设冲突少,用版本号(@Version)。更新时检查版本,变了就重试。适合读多写少。
> - **悲观锁(Pessimistic):** 假设冲突多,直接锁行(SELECT ... FOR UPDATE)。适合写多。

```java
@Entity
public class Account {
    @Version
    private Long version;  // 乐观锁版本号
}
```

## 10.4 事务 ACID

> 中文:
> - **Atomicity(原子性):** 要么全成功要么全回滚。
> - **Consistency(一致性):** 事务前后数据合法。
> - **Isolation(隔离性):** 并发事务互不干扰。
> - **Durability(持久性):** 提交后永久保存。

## 10.5 事务隔离级别

| 级别 | 脏读 | 不可重复读 | 幻读 |
|------|------|-----------|------|
| Read Uncommitted | ❌可能 | ❌可能 | ❌可能 |
| Read Committed | ✅防止 | ❌可能 | ❌可能 |
| Repeatable Read | ✅防止 | ✅防止 | ❌可能 |
| Serializable | ✅防止 | ✅防止 | ✅防止 |

> MySQL 默认 Repeatable Read,Oracle/PostgreSQL 默认 Read Committed。

## 10.6 SQL vs NoSQL 怎么选

> 中文:
> - **SQL(关系型,如 Oracle/PostgreSQL):** 强一致性、复杂查询、事务。适合金融核心数据。
> - **NoSQL(如 MongoDB/Redis):** 高扩展、灵活 schema、高吞吐。适合缓存、日志、非结构化数据。
> - 常见组合:核心交易用 SQL 保证 ACID,读密集的查询用 NoSQL/缓存加速。

## 10.7 连接池 (Connection Pool)

> 中文:创建数据库连接很昂贵,连接池预先建好一批连接复用。Spring Boot 默认用 **HikariCP**。关键参数:maximumPoolSize(最大连接数),要和数据库的连接上限匹配,否则并发高时会耗尽。

---