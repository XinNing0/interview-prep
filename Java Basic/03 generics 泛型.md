
> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 3: 泛型 (Generics) ⭐ PNC 重点

## 3.1 Upper/Lower Bounded Wildcards ⭐ 被问到(卡住了)

> 这是今天卡最惨的一题。核心是 **PECS 原则**:Producer Extends, Consumer Super。

**Upper Bounded (上界通配符) `<? extends T>`:**
> 中文:表示"T 或 T 的子类型"。用于**只读取(生产)**数据的场景。
> - 你可以安全地**读**出 T 类型(因为不管具体是什么子类,都是 T)。
> - 你**不能写入**(除了 null),因为编译器不知道具体是哪个子类。

```java
// Producer —— 从集合里读数据
public double sum(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) total += n.doubleValue();  // 读 OK
    return total;
    // list.add(1);  // ❌ 编译错误,不能写
}
// 可以传 List<Integer>, List<Double>, List<Number>
```

**Lower Bounded (下界通配符) `<? super T>`:**
> 中文:表示"T 或 T 的父类型"。用于**只写入(消费)**数据的场景。
> - 你可以安全地**写入** T 类型(因为 T 一定是这个类型或其子类)。
> - 你**读**出来只能当 Object,因为不知道具体的父类是哪个。

```java
// Consumer —— 往集合里写数据
public void addNumbers(List<? super Integer> list) {
    list.add(1);   // 写 OK
    list.add(2);   // 写 OK
    // Integer i = list.get(0);  // ❌ 只能读成 Object
}
// 可以传 List<Integer>, List<Number>, List<Object>
```

**PECS 记忆法:**
> - **P**roducer **E**xtends —— 集合是数据的**生产者**(你从里面读),用 `extends`
> - **C**onsumer **S**uper —— 集合是数据的**消费者**(你往里面写),用 `super`

**English answer (今天应该这样答):**
"Upper bounded wildcard `? extends T` means T or any subtype — you use it when you only READ from the collection, like a producer. You can read elements as type T but you can't add. Lower bounded `? super T` means T or any supertype — you use it when you only WRITE to the collection, like a consumer. The rule is PECS: Producer Extends, Consumer Super."

## 3.2 泛型擦除 (Type Erasure)

> 中文:Java 泛型是**编译期**的语法糖,运行时会被"擦除"成原始类型(Object 或上界)。所以 `List<String>` 和 `List<Integer>` 运行时是同一个类 `List`。这就是为什么不能 `new T()`、不能 `instanceof List<String>`。

---