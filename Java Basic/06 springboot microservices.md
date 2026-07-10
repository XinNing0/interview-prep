
---

# PART 6: Spring Boot & 微服务 (Spring Boot & Microservices)

## 6.1 IoC 与 DI (控制反转 / 依赖注入)

> 中文:
> - **IoC (Inversion of Control):** 对象的创建和管理交给 Spring 容器,而不是你自己 new。
> - **DI (Dependency Injection):** 容器把依赖"注入"进来。三种方式:构造器注入(推荐)、setter 注入、字段注入(@Autowired)。

```java
@Service
public class ScreeningService {
    private final AlertRepository repo;
    // 构造器注入(推荐 —— 便于测试,保证不可变)
    public ScreeningService(AlertRepository repo) { this.repo = repo; }
}
```

## 6.2 Bean 的作用域 (Scope)

> 中文:singleton(默认,单例)、prototype(每次都新建)、request、session。

## 6.3 Spring Boot 核心注解

> 中文:
> - `@SpringBootApplication` —— 启动类(= @Configuration + @EnableAutoConfiguration + @ComponentScan)
> - `@RestController` —— REST 控制器(= @Controller + @ResponseBody)
> - `@Service` / `@Repository` / `@Component` —— 标记 Bean
> - `@Autowired` —— 自动注入
> - `@RequestMapping` / `@GetMapping` / `@PostMapping` —— 路由映射
> - `@Transactional` —— 声明式事务

## 6.4 @Transactional 事务传播

> 中文:最常用 `REQUIRED`(有事务就加入,没有就新建)。还有 `REQUIRES_NEW`(总是新建独立事务)、`NESTED`(嵌套)等。

## 6.5 REST API 设计原则

> 中文:用 HTTP 方法表达操作(GET 查、POST 增、PUT 改、DELETE 删),用状态码表达结果(200 成功、201 创建、400 参数错、401 未认证、404 找不到、500 服务器错),URL 用名词复数(/alerts/123)。

## 6.6 微服务核心概念

> 中文:
> - **服务发现:** Eureka / Consul —— 服务互相找到对方。
> - **API 网关:** 统一入口,路由、鉴权、限流。
> - **配置中心:** 集中管理配置。
> - **熔断:** 服务故障时快速失败,防止雪崩(Resilience4j)。
> - **分布式追踪:** 追踪一个请求跨多个服务的链路。

---

# 面试策略总结 (Interview Strategy)

## 今天(PNC/Ivan)的教训

- 这个岗位实质是 **Java 并发后端工程师**,FircoSoft 只是业务背景
- 卡住的三题:**PECS 通配符、race condition 四种解法、CompletableFuture vs ExecutorService** —— 现在都在上面了
- 并发是 PNC 这类岗位的核心,务必吃透 PART 1

## 通用答题技巧

- **不会就说不会,别硬编** —— "I'm not 100% sure on the deep internals, but at a high level..." 比编错强
- **答不上来先给定义** —— Ivan 提示过:先说"是什么",再说"怎么用"
- **关联实际经验** —— 你 List vs Set 那题主动关联到 V6 用 ArrayList/HashMap,这个做得好,继续
- **简洁** —— Ivan 说"don't be pressured to give me a bunch of stuff",很多题答案很简单,别绕

## 这份材料同时服务两个方向

1. **PNC 这类 FircoSoft + Java 岗位**
2. **你真正的目标:general SDE** —— 这些并发、集合、JVM 是所有 Java 后端面试的必考

> 建议:配合 LeetCode 练习,把这些理论和你 github.com/XinNing0/leetcode_notes 的刷题结合起来。