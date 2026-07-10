
---

# PART 5: OOP 与核心语法 (OOP & Core)

## 5.1 面向对象四大特性

> 中文:封装(Encapsulation)、继承(Inheritance)、多态(Polymorphism)、抽象(Abstraction)。

## 5.2 重载 vs 重写 (Overload vs Override)

> 中文:
> - **重载 (Overload):** 同一个类里,方法名相同、参数不同。编译期决定。
> - **重写 (Override):** 子类重新实现父类方法,方法签名相同。运行期决定(多态)。

## 5.3 接口 vs 抽象类

| | 接口 (interface) | 抽象类 (abstract class) |
|---|---|---|
| 多继承 | 可以实现多个 | 只能继承一个 |
| 成员变量 | 只能 public static final | 可以普通成员变量 |
| 方法 | Java 8+ 可有 default 方法 | 可有普通方法 |
| 用途 | 定义能力/契约 | 复用代码 + 部分抽象 |

## 5.4 == vs equals

> 中文:`==` 比较引用地址(基本类型比值),`equals` 默认也比地址,但通常被重写成比内容(如 String)。

## 5.5 String vs StringBuilder vs StringBuffer

> 中文:
> - **String** 不可变,每次拼接创建新对象。
> - **StringBuilder** 可变,非线程安全,单线程拼接用它,快。
> - **StringBuffer** 可变,线程安全(方法加 synchronized),慢。

## 5.6 final / finally / finalize

> 中文:
> - **final:** 修饰变量(不可变)、方法(不可重写)、类(不可继承)。
> - **finally:** try-catch 中一定执行的块。
> - **finalize:** 对象被 GC 前调用的方法(已废弃,别用)。

---