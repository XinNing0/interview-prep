
> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 4: JVM & 内存 (JVM & Memory)

## 4.1 JDK vs JRE vs JVM ⭐ 被问到(答对了)

> 中文:
> - **JVM** (Java Virtual Machine):Java 虚拟机,真正执行字节码的地方。
> - **JRE** (Java Runtime Environment):运行时环境 = JVM + 核心类库。只能**运行**Java 程序。
> - **JDK** (Java Development Kit):开发工具包 = JRE + 编译器(javac)+ 调试工具。能**开发**Java 程序。

**English:** "JDK is the Java Development Kit — it's a superset of the JRE, adding the compiler and development tools on top. JRE is the runtime environment — it includes the JVM and the core class libraries needed to run Java programs."

> 关系:JDK ⊃ JRE ⊃ JVM

## 4.2 JVM 内存区域 (Runtime Data Areas)

> 中文:
> - **堆 (Heap):** 存对象实例,线程共享,GC 主战场。
> - **方法区 / 元空间 (Metaspace):** 存类信息、常量、静态变量。Java 8 后用元空间(在本地内存)取代永久代。
> - **虚拟机栈 (JVM Stack):** 每个线程私有,存局部变量、方法调用栈帧。
> - **程序计数器 (PC Register):** 记录当前线程执行到哪条字节码。
> - **本地方法栈 (Native Method Stack):** 给 native 方法用。

## 4.3 垃圾回收 (GC) 基础

> 中文:
> - **判断对象死亡:** 可达性分析 —— 从 GC Roots 出发能不能到达这个对象。
> - **分代收集:** 新生代(Young,存活时间短,Minor GC 频繁)+ 老年代(Old,存活久,Full GC)。
> - **常见 GC 器:** G1(Java 9+ 默认)、ZGC(低延迟)。

## 4.4 内存泄漏常见原因

> 中文:静态集合持有对象、未关闭的资源(连接/流)、ThreadLocal 未 remove、监听器未注销。

---