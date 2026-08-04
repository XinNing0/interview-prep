# RESTful API 设计 — 面试问答

> 面试话题对应：`Backend: API design`、`networking fundamentals`
> 关联：`01 concurrency`（幂等）、`09 exceptions`（全局异常）、`06 springboot microservices`

---

## Q1. 什么是 REST？

**答：**

REST 是一种架构风格，不是协议。核心是**把一切抽象成资源（resource）**，用 URI 定位资源，用 HTTP 动词表达对资源的操作。

关键约束（面试常问的三条）：

| 约束 | 含义 | 实际意义 |
|---|---|---|
| **Client-Server** | 前后端分离 | 各自独立演进 |
| **Stateless** | 服务端不保存会话状态 | 每个请求自带全部信息 → 可水平扩展 |
| **Uniform Interface** | 统一接口 | 动词固定，语义靠 URI + method 表达 |

其余三条：Cacheable（可缓存）、Layered System（分层）、Code on Demand（可选）。

**加分回答 —— 为什么无状态重要：**

> 无状态意味着任何一台实例都能处理任何请求，这是能放到负载均衡后面、能水平扩容的前提。所以现在普遍用 JWT 而不是 server-side session：token 自带用户信息，服务端不用查 session 存储。如果必须存状态，就外置到 Redis，而不是放在应用内存里。

---

## Q2. HTTP 动词和 CRUD 怎么对应？

| 动词 | 操作 | 典型 URI | 成功状态码 |
|---|---|---|---|
| POST | Create | `POST /users` | **201** + `Location` header |
| GET | Read | `GET /users/1` | 200 |
| PUT | Update（全量替换） | `PUT /users/1` | 200 或 204 |
| PATCH | Update（部分更新） | `PATCH /users/1` | 200 |
| DELETE | Delete | `DELETE /users/1` | 204（或 200 带 body） |

**注意 POST 成功要返 201，不是 200**，并且在 `Location` header 里给出新资源的 URI：

```
HTTP/1.1 201 Created
Location: /api/v1/users/12345
```

---

## Q3. ⭐ Safe 和 Idempotent 的区别？（高频）

这题几乎必考，而且很多人答混。

- **Safe（安全）**：不修改服务端资源。只有读操作是 safe。
- **Idempotent（幂等）**：**执行一次和执行 N 次，服务端最终状态相同**。注意约束的是"最终状态"，不是"返回值相同"。

| 动词 | Safe | Idempotent |
|---|---|---|
| GET | ✅ | ✅ |
| HEAD | ✅ | ✅ |
| OPTIONS | ✅ | ✅ |
| **POST** | ❌ | **❌** |
| **PUT** | ❌ | **✅** |
| **DELETE** | ❌ | **✅** |
| PATCH | ❌ | ⚠️ 取决于实现 |

**为什么 PUT 幂等而 POST 不幂等：**

```
PUT /users/1  {"name": "Ann"}     → 执行 10 次，user 1 的 name 还是 Ann
POST /users   {"name": "Ann"}     → 执行 10 次，创建了 10 个用户
```

**追问：DELETE 第二次返回 404，还算幂等吗？**

> 算。幂等约束的是**资源的最终状态**，不是响应码。第一次删掉，之后每次调用资源都是"不存在"这个状态，没有变化。返 404 只是准确地报告了当前状态。

**追问：PATCH 为什么不一定幂等？**

> 取决于 payload 语义。`PATCH {"status": "PAID"}` 是幂等的（赋值）；但如果 patch 语义是 `{"op": "increment", "field": "count"}`，每次调用值都变，就不幂等了。

---

## Q4. ⭐ 幂等性在工程上怎么实现？（结合项目/MQ 必问）

理论说完，面试官一定会问"你实际怎么做的"。五种方案：

**1. 数据库唯一约束（最简单可靠）**

```sql
ALTER TABLE orders ADD UNIQUE KEY uk_order_no (order_no);
```
重复插入直接抛 `DuplicateKeyException`，捕获后当成功处理。

**2. 单条 SQL 原子操作（避免"读-改-写"）**

```sql
-- ❌ 错误：读出来判断再更新，并发下会超卖
-- ✅ 正确：判断和更新在同一条语句里，靠数据库行锁保证原子性
UPDATE commodity SET stock = stock - 1
WHERE id = #{id} AND stock > 0;
-- 检查影响行数：0 表示库存不足，无需回滚
```

**3. 乐观锁（version 字段）**

```sql
UPDATE orders SET status = 2, version = version + 1
WHERE id = #{id} AND version = #{version};
```
影响行数为 0 说明被别人改过，重试或报冲突（409 Conflict）。

**4. 状态机流转（业务幂等）**

```java
if (order.getStatus() != PENDING_PAYMENT) {
    return;  // 已支付过，直接返回，不重复处理
}
order.setStatus(PAID);
```
适合订单支付这类"只能从状态 A 到状态 B"的场景。

**5. 去重表 / Redis（消息消费、外部回调）**

```java
// 用业务唯一 ID 去重，SETNX 保证原子
Boolean first = redis.opsForValue()
        .setIfAbsent("msg:" + messageId, "1", Duration.ofHours(24));
if (!first) return;   // 已处理过，丢弃
```

**MQ 场景的标准回答：**

> 分布式系统里 exactly-once 代价极高（要跨系统的分布式事务），所以工程上普遍是 **at-least-once 投递 + 消费端幂等**，效果上等价于 exactly-once。消费端幂等靠业务唯一键去重：用消息 ID 或业务单号建唯一索引/去重表，重复消息直接丢弃。处理失败超过重试次数就进 DLQ，人工或定时任务捞出来重投。

---

## Q5. 状态码怎么选？

**2xx 成功**

| 码 | 用在哪 |
|---|---|
| 200 OK | 通用成功，有响应体 |
| 201 Created | 创建成功，配 `Location` header |
| 202 Accepted | 已接收，异步处理中（**MQ 场景常用**） |
| 204 No Content | 成功但无响应体，DELETE / PUT 常用 |

**4xx 客户端错误**

| 码 | 用在哪 |
|---|---|
| 400 Bad Request | 参数格式错误、校验失败 |
| **401 Unauthorized** | **未认证**——没登录 / token 无效过期 |
| **403 Forbidden** | **已认证但无权限**——登录了但不是管理员 |
| 404 Not Found | 资源不存在 |
| 405 Method Not Allowed | URI 存在但不支持这个动词 |
| 409 Conflict | 资源状态冲突（乐观锁失败、重复创建） |
| 422 Unprocessable Entity | 格式对但业务语义不合法 |
| 429 Too Many Requests | 限流触发（**对应限流章节**） |

⭐ **401 vs 403 是高频陷阱题**：401 = 你是谁我不知道（认证问题）；403 = 我知道你是谁，但你不能干这个（授权问题）。

**5xx 服务端错误**

500 内部错误、502 网关拿到上游坏响应、503 服务不可用（过载/维护）、504 网关超时。

⚠️ **原则：客户端的错不要返 5xx。** 参数错了返 400，不要因为没做校验导致 NPE 然后返 500——这是代码质量信号。

---

## Q6. URL 该怎么设计？

**核心原则：URI 用名词表示资源，动作由 HTTP 动词表达。**

```
❌ GET  /getUserById?id=1        动词在 URL 里
❌ POST /createUser              动词冗余
❌ GET  /addItemAction           用 GET 做写操作（危险，见下）
❌ POST /user/delete/1           动词 + 错误的动词选择

✅ GET    /users/1
✅ POST   /users
✅ PUT    /users/1
✅ DELETE /users/1
```

**其他规范：**

- **资源名用复数**：`/users` 而非 `/user`
- **嵌套表达从属关系**：`GET /users/1/orders`（1 号用户的所有订单）
- **嵌套不超过两层**：再深就用 query param —— `GET /orders?userId=1&status=PAID`
- **小写 + 连字符**：`/order-items`，不用 `/orderItems` 或 `/order_items`
- **过滤/分页/排序用 query param，不进路径**：

```
GET /users?page=2&size=20&sort=createTime,desc&status=ACTIVE
```

- **版本号**：`/api/v1/users`（放路径最常见），或放在 header 里 `Accept: application/vnd.api.v1+json`

---

## Q7. ⭐ 为什么不能用 GET 做写操作？

这题能体现工程判断力，答出三个理由就很出彩：

1. **HTTP 规范定义 GET 是 safe 的**，整条链路上的组件都按这个假设行事
2. **会被意外触发**：浏览器预加载、爬虫抓取、聊天软件生成链接预览、用户刷新页面 —— 每一次都会执行一遍写操作
3. **会被缓存**：浏览器、CDN、反向代理都可能缓存 GET 响应，导致请求根本没到服务端，或者旧数据被返回
4. **参数暴露在 URL 里**：会进浏览器历史、Nginx access log、Referer header —— 敏感数据泄露
5. **CSRF 风险**：一个 `<img src="/deleteUser/1">` 就能触发删除

**所以：`@RequestMapping` 不指定 `method` 是隐患**，因为它默认接受所有动词。生产代码应该用 `@GetMapping` / `@PostMapping` 这类明确的注解。

---

## Q8. POST vs PUT vs PATCH 怎么选？

| | POST | PUT | PATCH |
|---|---|---|---|
| 语义 | 创建（服务端定 ID） | 全量替换（客户端定 URI） | 部分更新 |
| 幂等 | ❌ | ✅ | ⚠️ 看实现 |
| 传什么 | 新资源数据 | **完整**资源表示 | **只传要改的字段** |

**PUT 的坑（面试爱问）：**

> PUT 是全量替换，没传的字段应该被置空/置默认值。如果只想改一个字段却用 PUT 且只传了那个字段，严格实现下其他字段会被清掉。所以**部分更新应该用 PATCH**。实践中很多团队用 PUT 做部分更新，这是不严格的，知道区别就好。

**主键该由谁生成？**

> 一般由服务端生成（数据库自增或雪花算法），用 POST。客户端指定 ID 的话可以用 PUT，但会带来 ID 冲突和可预测性问题（用自增 ID 还会有资源枚举的安全风险），所以不常用。

---

## Q9. 统一响应格式 + 全局异常处理

面试官看项目代码，这两个是"写过正经项目"的直接信号。

```java
// 统一响应包装
@Data
@AllArgsConstructor
public class Result<T> {
    private int code;
    private String message;
    private T data;

    public static <T> Result<T> ok(T data) {
        return new Result<>(200, "success", data);
    }
    public static <T> Result<T> fail(int code, String msg) {
        return new Result<>(code, msg, null);
    }
}
```

```java
// 全局异常处理
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 参数校验失败 → 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Result<Void> handleValidation(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldError().getDefaultMessage();
        return Result.fail(400, msg);
    }

    // 业务异常 → 自定义码
    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusiness(BusinessException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }

    // 兜底 → 500，注意日志要打，但不要把堆栈返给客户端
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public Result<Void> handleAll(Exception e) {
        log.error("unexpected error", e);
        return Result.fail(500, "服务器内部错误");
    }
}
```

**⭐ 追问：HTTP 状态码和 body 里的 code 重复了吗？**

> 不重复，是两层语义。HTTP 状态码是**传输层**的结果（网关、监控、重试策略都依赖它）；body 里的 code 是**业务层**的结果（比如"库存不足"、"余额不够"）。
>
> 一个常见的坏实践是所有响应都返 HTTP 200，错误只体现在 body 里——这会让 API 网关、监控告警、客户端重试逻辑全部失效，因为它们看不出这个请求失败了。

**追问：为什么不能把堆栈返给客户端？**

> 安全问题。堆栈会暴露框架版本、包结构、SQL 语句甚至数据库表名，是攻击者的信息来源。堆栈打进日志，返给客户端的是脱敏后的提示 + 一个 traceId 方便排查。

---

## Q10. 参数校验

```java
public class UserCreateDTO {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 2, max = 20, message = "用户名长度 2-20")
    private String name;

    @Email(message = "邮箱格式不正确")
    private String email;

    @NotNull
    @Min(value = 0, message = "价格不能为负")
    private BigDecimal price;
}

@PostMapping
public Result<Long> create(@Valid @RequestBody UserCreateDTO dto) { ... }
```

**⭐ 追问：为什么要 DTO，不直接用 Entity 接收？**

四个理由，答两三个就够：

1. **安全**：Entity 可能有 `password`、`isAdmin` 字段，直接绑定会被恶意注入（over-posting 攻击）
2. **解耦**：数据库表结构变了不会直接影响 API 契约
3. **字段不一致**：创建时不需要 `id`、`createTime`；返回时不该带 `password`
4. **校验规则不同**：创建和更新的必填项不一样，Entity 上挂不了两套注解

---

## Q11. REST 有什么局限？什么时候不用 REST？

这题答得好会显得你有真实判断，不是只会背规范。

| 方案 | 适合 | 不适合 |
|---|---|---|
| **REST** | 资源型 CRUD、公开 API、需要 HTTP 缓存 | 复杂查询、需要聚合多个资源 |
| **GraphQL** | 前端字段需求多变、避免 over/under-fetching | 缓存复杂、有 N+1 查询风险 |
| **gRPC** | **内部服务间调用**，低延迟高吞吐（Protobuf 二进制 + HTTP/2 多路复用） | 浏览器直连支持差、调试不直观 |

**REST 的两个典型痛点：**

1. **Over-fetching / Under-fetching**：返回的字段前端只用一半（浪费带宽），或者一个页面要发五个请求才能拼出数据
2. **不是所有操作都能自然映射成资源**：`POST /orders/1/cancel` 这种"动作型"接口，严格 REST 应该表达成 `PATCH /orders/1 {"status":"CANCELLED"}`，但前者可读性更好。**实践中允许这种妥协**，不用教条。

**关于 HATEOAS**：REST 成熟度最高的一级（Richardson 成熟度模型 Level 3），响应里带上相关操作的链接。理论上很美，实际用得少——**知道这个词、知道大部分 API 只做到 Level 2 就够了**。

---

## Q12. Spring 注解速查（live coding 会用到）

| 注解 | 作用 |
|---|---|
| `@RestController` | `@Controller` + `@ResponseBody`，返回值直接序列化成 JSON |
| `@RequestMapping` | 类级别定义公共前缀 |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` / `@PatchMapping` | 方法级别，**明确动词** |
| `@PathVariable` | 取路径里的占位符 `/users/{id}` |
| `@RequestParam` | 取 query param `?page=2`，支持 `required` / `defaultValue` |
| `@RequestBody` | 取请求体（JSON → 对象） |
| `@Valid` | 触发 DTO 上的校验注解 |
| `@RestControllerAdvice` | 全局异常处理 |
| `@ResponseStatus` | 指定响应的 HTTP 状态码 |

**一个能背下来的完整模板（面试手写用）：**

```java
@RestController
@RequestMapping("/api/v1/commodities")
public class CommodityController {

    private final CommodityService service;

    // 构造器注入：不可变、便于测试、循环依赖启动时就暴露
    public CommodityController(CommodityService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public Result<CommodityVO> getById(@PathVariable Long id) {
        return Result.ok(service.getById(id));
    }

    @GetMapping
    public Result<List<CommodityVO>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) Long sellerId) {
        return Result.ok(service.list(page, size, sellerId));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)          // 201
    public Result<Long> create(@Valid @RequestBody CommodityCreateDTO dto) {
        return Result.ok(service.create(dto));
    }

    @PatchMapping("/{id}")
    public Result<Void> update(@PathVariable Long id,
                               @Valid @RequestBody CommodityUpdateDTO dto) {
        service.update(id, dto);
        return Result.ok(null);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)       // 204
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}
```

⚠️ **关掉 IDE 自动补全，把这段练到能默写。** 面试要求关补全，`@RequestMapping` 拼错、`ResponseEntity` 忘了 import 都会卡住节奏。

---

## Q13. 反例分析（可以主动讲，很出彩）

如果面试官让你评审代码，或者问"你见过什么不好的 API 设计"：

```java
@Controller
public class CommodityController {
    @Resource
    OnlineShoppingCommodityDao dao;                    // ① 字段注入

    @RequestMapping("/addItemAction")                  // ② 动词进 URL
    public String addItemAction(                       // ③ 未指定 method → GET 可写
            @RequestParam("commodityId") long commodityId,   // ④ 参数散开，无校验
            @RequestParam("price") int price, ...) {
        dao.insertCommodity(...);                      // ⑤ Controller 直连 DAO，无 Service
        return "add_commodity_success";                // ⑥ 无统一响应 / 状态码
    }
}
```

**六个问题，改法：**

| 问题 | 改法 |
|---|---|
| ① `@Resource` 字段注入 | 构造器注入 —— 不可变、可测试、循环依赖启动即报错 |
| ② `/addItemAction` | `POST /api/v1/commodities` |
| ③ `@RequestMapping` 不带 method | `@PostMapping`，明确动词 |
| ④ 六个 `@RequestParam` 散开 | 收成一个 DTO + `@Valid` |
| ⑤ Controller 直连 DAO | 加 Service 层承载业务逻辑和事务边界 |
| ⑥ 返视图名，无状态码 | 返 `201 Created` + `Location` header |

**加一句判断力：**

> 不过要区分场景——如果这是服务端渲染的页面（Thymeleaf），返回视图名是对的，那本来就不是 REST API。REST 规范适用于给前端/第三方调用的 JSON 接口。混在一个 Controller 里才是真问题。

---

## 一分钟自测

答不上来的回去看对应章节：

1. PUT 和 POST 哪个幂等？为什么？
2. 401 和 403 的区别？
3. DELETE 第二次返 404 还算幂等吗？
4. 为什么不能用 GET 做写操作？（说三个理由）
5. 创建资源成功返什么状态码？带什么 header？
6. MQ 消费端怎么保证幂等？（说两种实现）
7. 为什么要 DTO 不直接用 Entity？
8. 所有接口都返 HTTP 200、错误只写在 body 里，有什么问题？
9. 什么场景下你会选 gRPC 而不是 REST？
10. 不看任何提示，手写一个带分页查询和创建的 RestController。