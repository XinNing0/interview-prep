*中英对照 | 详细讲解 | 面试可直接使用*

> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 11: 设计模式 (Design Patterns) ⭐ 常考

## 11.1 单例模式 (Singleton) ⭐

```java
// 双重检查锁(线程安全 + 懒加载)
public class Config {
    private static volatile Config instance;
    private Config() {}
    public static Config getInstance() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) {
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}
// 更推荐:枚举单例(天然线程安全,防反射)
public enum ConfigEnum { INSTANCE; }
```

## 11.2 工厂模式 (Factory)

> 中文:把对象创建逻辑封装起来,调用者不用关心具体怎么 new。

## 11.3 策略模式 (Strategy)

> 中文:定义一系列算法,封装起来可互换。比如不同的筛查规则策略。

## 11.4 建造者模式 (Builder)

```java
Transaction tx = Transaction.builder()
    .id("T1").amount(5000).currency("USD").build();
```

## 11.5 观察者模式 (Observer)

> 中文:一对多依赖,对象状态变化时通知所有观察者。事件驱动的基础。

---