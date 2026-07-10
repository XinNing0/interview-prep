*中英对照 | 详细讲解 | 面试可直接使用*

> ⭐ 标记 = 高频考点 | ⭐⭐ = 2026 必考/重点

---

# PART 9: 异常处理 (Exception Handling) ⭐ 高频

## 9.1 Checked vs Unchecked 异常

> 中文:
> - **Checked(受检)异常:** 编译期强制处理(try-catch 或 throws)。如 IOException、SQLException。
> - **Unchecked(非受检)异常:** 运行时异常,编译器不强制。如 NullPointerException、IllegalArgumentException。继承自 RuntimeException。
> - **Error:** 严重的系统级错误,不该 catch。如 OutOfMemoryError、StackOverflowError。

```
Throwable
├── Error (不该处理:OOM、StackOverflow)
└── Exception
    ├── Checked (强制处理:IOException, SQLException)
    └── RuntimeException (不强制:NPE, IllegalArgument)
```

## 9.2 try-with-resources ⭐

> 中文:自动关闭实现了 AutoCloseable 的资源,不用手动 finally 关闭。

```java
// 旧写法要 finally 关闭
// 新写法自动关闭
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.executeQuery();
} // conn 和 ps 自动关闭,即使抛异常
```

## 9.3 finally 与 return

> 中文:finally 一定执行(除非 System.exit)。如果 finally 里有 return,会覆盖 try 里的 return——这是坑,面试常问。

---