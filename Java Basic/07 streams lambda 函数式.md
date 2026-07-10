*中英对照 | 详细讲解 | 面试可直接使用*

> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 7: Java 8 函数式编程 (Streams & Lambdas) ⭐⭐ 2026 必考

> 2026 面试中 Streams 是最高频的 Java 8 考点,几乎每场都问。

## 7.1 Lambda 表达式

> 中文:Lambda 是匿名函数,让你把"行为"当作参数传递。语法:`(参数) -> {函数体}`。

```java
// 传统匿名类
Runnable r1 = new Runnable() {
    public void run() { System.out.println("Hello"); }
};
// Lambda 简化
Runnable r2 = () -> System.out.println("Hello");
```

## 7.2 函数式接口 (Functional Interface)

> 中文:只有**一个抽象方法**的接口,可以用 Lambda 实现。四个核心内置接口:

| 接口 | 方法 | 用途 | 例子 |
|------|------|------|------|
| `Function<T,R>` | R apply(T) | 输入 T 返回 R | 转换 |
| `Predicate<T>` | boolean test(T) | 输入 T 返回布尔 | 过滤 |
| `Consumer<T>` | void accept(T) | 输入 T 无返回 | 消费/打印 |
| `Supplier<T>` | T get() | 无输入返回 T | 生产/工厂 |

## 7.3 Stream API ⭐⭐ 高频

> 中文:Stream 是对集合做**声明式**处理的管道,分为**中间操作**(懒执行,返回 Stream)和**终端操作**(触发执行,返回结果)。

**中间操作:** `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`
**终端操作:** `collect`, `forEach`, `reduce`, `count`, `anyMatch`, `findFirst`

```java
// 经典例子:筛选、转换、收集
List<String> result = transactions.stream()
    .filter(t -> t.getAmount() > 1000)          // 中间:过滤
    .map(Transaction::getBeneficiary)            // 中间:转换
    .distinct()                                  // 中间:去重
    .sorted()                                    // 中间:排序
    .collect(Collectors.toList());               // 终端:收集

// 分组(高频考点 groupingBy)
Map<String, List<Transaction>> byCurrency = transactions.stream()
    .collect(Collectors.groupingBy(Transaction::getCurrency));

// 统计每种货币的交易笔数
Map<String, Long> countByCurrency = transactions.stream()
    .collect(Collectors.groupingBy(Transaction::getCurrency, Collectors.counting()));

// 求和
double total = transactions.stream()
    .mapToDouble(Transaction::getAmount)
    .sum();

// 统计字符串里每个字符出现次数(常考编程题)
Map<Character, Long> charCount = "hello".chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(c -> c, Collectors.counting()));
```

## 7.4 map vs flatMap ⭐ 高频

> 中文:
> - **map:** 一对一转换。`Stream<String>` → `Stream<Integer>`。
> - **flatMap:** 一对多再压平。`Stream<List<String>>` → `Stream<String>`。把嵌套结构"拍扁"。

```java
// flatMap 例子:把 List<List<String>> 压平成 List<String>
List<List<String>> nested = Arrays.asList(
    Arrays.asList("a", "b"), Arrays.asList("c", "d"));
List<String> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [a, b, c, d]
```

## 7.5 Optional ⭐ 高频

> 中文:Optional 是一个容器,用来优雅处理 null,避免 NullPointerException。

```java
Optional<Transaction> tx = repository.findById(id);
// 不好:tx.get() 可能抛异常
// 好:
String beneficiary = tx.map(Transaction::getBeneficiary)
                       .orElse("UNKNOWN");
tx.ifPresent(t -> process(t));  // 存在才执行
Transaction result = tx.orElseThrow(() -> new NotFoundException());
```

---