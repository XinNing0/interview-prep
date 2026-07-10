# Java SDE 面试笔记 (Java Interview Notes)

一套完整的 Java 后端/SDE 面试准备笔记,中英对照,按话题分成独立文件。
基于 2026 年主流 Java 面试趋势整理。

> ⭐ = 高频考点 | ⭐⭐ = 2026 必考/重点

## 目录

| # | 话题 | 重点 | 优先级 |
|---|------|------|--------|
| [01](01-Concurrency-并发编程.md) | 并发编程 Concurrency | race condition 四种解法、CAS、线程池、volatile、CompletableFuture | ⭐⭐ 最高频 |
| [02](02-Collections-集合框架.md) | 集合框架 Collections | List vs Set、equals/hashCode、HashMap 原理、ConcurrentHashMap | ⭐⭐ |
| [03](03-Generics-泛型.md) | 泛型 Generics | PECS 通配符、类型擦除 | ⭐ |
| [04](04-JVM-Memory-内存.md) | JVM & 内存 | JDK/JRE/JVM、内存区域、GC | ⭐ |
| [05](05-OOP-Core-核心语法.md) | OOP & 核心语法 | 四大特性、重载重写、接口抽象类、String | ⭐ |
| [06](06-SpringBoot-Microservices.md) | Spring Boot & 微服务 | IoC/DI、注解、事务、REST、微服务概念 | ⭐⭐ |
| [07](07-Streams-Lambda-函数式.md) | Streams & Lambda | Stream API、map/flatMap、groupingBy、Optional | ⭐⭐ 几乎必问 |
| [08](08-Java17-21-现代特性.md) | Java 17-21 新特性 | Record、Sealed、Pattern Matching、Virtual Threads | ⭐⭐ 加分项 |
| [09](09-Exceptions-异常处理.md) | 异常处理 | Checked vs Unchecked、try-with-resources | ⭐ |
| [10](10-Database-JPA-Hibernate.md) | 数据库 & JPA | N+1 问题、乐观悲观锁、事务隔离、SQL vs NoSQL | ⭐⭐ senior 试金石 |
| [11](11-Design-Patterns-设计模式.md) | 设计模式 | 单例、工厂、策略、建造者、观察者 | ⭐ |
| [12](12-System-Design-系统设计.md) | 系统设计 | 答题框架、CAP 定理、缓存、分片、消息队列 | ⭐⭐ senior 必考 |
| [13](13-Coding-Questions-编程题.md) | 常见编程题 | Stream 手写题、滑动窗口、groupingBy | ⭐ 现场手写 |

## 建议学习顺序(按考频)

1. **并发 (01)** — 最高频,大量 Java 后端岗位的核心
2. **Streams/Lambda (07)** — 几乎每场必问
3. **集合 (02)** — HashMap 原理必问
4. **数据库/JPA (10)** — N+1 是 senior 试金石
5. **Java 17-21 (08)** — 加分项,尤其 virtual threads
6. **JVM (04)** — GC、内存模型
7. **Spring Boot (06)** — 微服务岗位重点
8. **系统设计 (12)** — senior 岗位必考
9. 其余 (03, 05, 09, 11, 13) 按需

## 2026 年 Java 面试趋势

- 不再问"什么是 Java""解释 OOP"这种纯背诵题
- 转向真实场景:"walk me through a production issue you've owned"
- 自我评估准确性很关键 — 别把框架熟悉度当深度专长吹
- Java 8 Streams、Java 17-21 新特性越来越常问
- JPA/Hibernate N+1 问题是区分真实经验的试金石
- 系统设计要主动问澄清问题、讲权衡

## 配合使用

- 算法手写题 → LeetCode 练习
- 八股理论 → 本笔记
- 两者结合 = 完整的 Java SDE 准备