# RESTful API 设计 — 面试问答（中英对照）

> 面试话题对应：`Backend: API design`、`networking fundamentals`
> 关联：`01 concurrency`（幂等）、`09 exceptions`（全局异常）、`06 springboot microservices`
>
> **用法：中文段落用来理解，🗣️ English answer 用来开口。英文答案已按口语节奏写，可以直接背。**

---

## Q1. 什么是 REST？ / What is REST?

**中文理解：**

REST 是一种架构风格，不是协议。核心是**把一切抽象成资源（resource）**，用 URI 定位资源，用 HTTP 动词表达操作。

关键约束（常问三条）：

| 约束 | 含义 | 实际意义 |
|---|---|---|
| Client-Server | 前后端分离 | 各自独立演进 |
| **Stateless** | 服务端不保存会话状态 | 每个请求自带全部信息 → 可水平扩展 |
| Uniform Interface | 统一接口 | 动词固定，语义靠 URI + method 表达 |

其余三条：Cacheable、Layered System、Code on Demand（可选）。

> 🗣️ **English answer**
>
> "REST is an architectural style rather than a protocol. The core idea is that everything is modeled as a resource — you identify the resource with a URI, and you express the operation with an HTTP verb.
>
> The constraints I care about most in practice are three. First, client-server separation, so the frontend and backend can evolve independently. Second, statelessness — the server doesn't hold session state, so every request carries everything it needs. Third, a uniform interface — the set of verbs is fixed, so the meaning comes from the URI plus the method.
>
> Statelessness is the one that matters most operationally. If the server holds no state, any instance can serve any request, which is what lets you put the service behind a load balancer and scale horizontally. That's why most systems now use JWT instead of server-side sessions — the token carries the user context, so there's no session lookup. If you do need shared state, you externalize it to Redis rather than keeping it in application memory."

---

## Q2. HTTP 动词和 CRUD 的对应 / Mapping verbs to CRUD

| 动词 | 操作 | 典型 URI | 成功状态码 |
|---|---|---|---|
| POST | Create | `POST /users` | **201** + `Location` header |
| GET | Read | `GET /users/1` | 200 |
| PUT | Update（全量替换） | `PUT /users/1` | 200 或 204 |
| PATCH | Update（部分更新） | `PATCH /users/1` | 200 |
| DELETE | Delete | `DELETE /users/1` | 204（或 200 带 body） |

**注意 POST 成功要返 201，不是 200**，并在 `Location` header 给出新资源 URI：

```
HTTP/1.1 201 Created
Location: /api/v1/users/12345
```

> 🗣️ **English answer**
>
> "POST creates, GET reads, PUT replaces, PATCH partially updates, DELETE removes.
>
> The detail people often miss is the response. A successful POST should return 201 Created, not 200, and it should include a Location header pointing at the newly created resource, so the client knows where it lives without guessing. For DELETE, I'd return 204 No Content when there's nothing meaningful to send back."

---

## Q3. ⭐ Safe 和 Idempotent 的区别 / Safe vs idempotent

这题几乎必考，很多人答混。

- **Safe（安全）**：不修改服务端资源，只有读操作是 safe。
- **Idempotent（幂等）**：执行一次和执行 N 次，**服务端最终状态相同**。约束的是"最终状态"，不是"返回值"。

| 动词 | Safe | Idempotent |
|---|---|---|
| GET / HEAD / OPTIONS | ✅ | ✅ |
| **POST** | ❌ | **❌** |
| **PUT** | ❌ | **✅** |
| **DELETE** | ❌ | **✅** |
| PATCH | ❌ | ⚠️ 取决于实现 |

```
PUT /users/1  {"name": "Ann"}   → 执行 10 次，name 还是 Ann
POST /users   {"name": "Ann"}   → 执行 10 次，创建 10 个用户
```

> 🗣️ **English answer**
>
> "They're two different properties. Safe means the operation doesn't modify server state at all — only reads are safe. Idempotent means calling it once and calling it N times leave the server in the same final state. The key point is that idempotency constrains the *final state*, not the response.
>
> GET, HEAD and OPTIONS are both safe and idempotent. PUT and DELETE are idempotent but not safe. POST is neither — if you send the same POST ten times, you create ten resources. PATCH depends on the payload semantics.
>
> The clearest contrast is PUT versus POST. `PUT /users/1` with a name field sets that name — do it ten times and the name is still the same. `POST /users` with the same body creates ten separate users."

**追问 1：DELETE 第二次返回 404，还算幂等吗？**

> 🗣️ "Yes, it's still idempotent. Idempotency is about the state of the resource, not the status code. After the first call the resource is gone, and every call after that leaves it gone — the state never changes again. Returning 404 is just accurately reporting that the resource doesn't exist."

**追问 2：PATCH 为什么不一定幂等？**

> 🗣️ "It depends on what the patch means. If the payload is an assignment — say, set status to PAID — that's idempotent. But if the semantics are relative, like 'increment this counter by one', then every call changes the value, so it isn't idempotent anymore."

---

## Q4. ⭐⭐ 幂等性在工程上怎么实现 / How do you actually implement idempotency?

**这题和你简历上 MQ 那条 bullet 直接对应，最可能被深挖。**

**1. 数据库唯一约束（最简单可靠）**

```sql
ALTER TABLE orders ADD UNIQUE KEY uk_order_no (order_no);
```
重复插入抛 `DuplicateKeyException`，捕获后当成功处理。

**2. 单条 SQL 原子操作（避免"读-改-写"）**

```sql
-- ❌ 读出来判断再更新，并发下会超卖
-- ✅ 判断和更新在同一条语句里，靠行锁保证原子性
UPDATE commodity SET stock = stock - 1
WHERE id = #{id} AND stock > 0;
-- 检查影响行数：0 表示库存不足
```

**3. 乐观锁（version 字段）**

```sql
UPDATE orders SET status = 2, version = version + 1
WHERE id = #{id} AND version = #{version};
```
影响行数为 0 说明被改过 → 重试或返 409 Conflict。

**4. 状态机流转（业务幂等）**

```java
if (order.getStatus() != PENDING_PAYMENT) {
    return;  // 已支付过，直接返回
}
order.setStatus(PAID);
```

**5. 去重表 / Redis（消息消费、外部回调）**

```java
Boolean first = redis.opsForValue()
        .setIfAbsent("msg:" + messageId, "1", Duration.ofHours(24));
if (!first) return;   // 已处理过，丢弃
```

> 🗣️ **English answer（通用版）**
>
> "There are a few layers I'd reach for, depending on the operation.
>
> The simplest and most reliable is a unique constraint in the database — put a unique index on the business key, like the order number. A duplicate insert throws a duplicate-key exception, and I catch that and treat it as success.
>
> Second, push the check and the write into a single atomic statement instead of read-modify-write. So instead of selecting the stock, checking it in Java, and then updating, I do `UPDATE commodity SET stock = stock - 1 WHERE id = ? AND stock > 0` and look at the affected row count. Zero means there wasn't enough stock. That closes the race window entirely because the row lock does the work.
>
> Third, optimistic locking with a version column, when I need to detect concurrent modification and surface a conflict.
>
> Fourth, business-level idempotency through a state machine — if an order can only go from pending to paid, then checking the current state before transitioning makes a repeated call a no-op.
>
> And for message consumption or external callbacks, a dedup table or Redis with a set-if-absent on the message ID."

> 🗣️ **English answer（MQ 版 —— 被问到简历上那条 bullet 时用这个）**
>
> "In a distributed system, exactly-once delivery is extremely expensive — you'd need a distributed transaction across the broker and your database. So the practical pattern is at-least-once delivery on the broker side, plus idempotent consumers on our side. The combination gives you the effect of exactly-once.
>
> On the consumer side we deduplicated on a business unique key — the message ID or the business reference number — backed by a unique index, so a redelivered message was rejected as a duplicate and discarded rather than reprocessed.
>
> For failures, messages that exceeded the retry count went to a dead letter queue rather than blocking the main queue. From there they could be inspected and replayed once the underlying issue was fixed. That's how we got to zero data loss — nothing is silently dropped, it either succeeds, retries, or lands in the DLQ."

⚠️ **把这段的实现细节换成你在 BNY 实际用的方案再背** —— 面试官会顺着追问「what was the unique key」「how did you replay from the DLQ」，答案得是真的。

---

## Q5. 状态码怎么选 / Choosing status codes

**2xx**

| 码 | 用在哪 |
|---|---|
| 200 OK | 通用成功，有响应体 |
| 201 Created | 创建成功，配 `Location` |
| 202 Accepted | 已接收，异步处理中（**MQ 场景常用**） |
| 204 No Content | 成功但无响应体 |

**4xx**

| 码 | 用在哪 |
|---|---|
| 400 Bad Request | 参数格式错误、校验失败 |
| **401 Unauthorized** | **未认证** —— 没登录 / token 无效过期 |
| **403 Forbidden** | **已认证但无权限** |
| 404 Not Found | 资源不存在 |
| 405 Method Not Allowed | URI 存在但不支持这个动词 |
| 409 Conflict | 状态冲突（乐观锁失败、重复创建） |
| 422 Unprocessable Entity | 格式对但业务语义不合法 |
| 429 Too Many Requests | 限流触发 |

**5xx**：500 内部错误、502 网关收到坏响应、503 服务不可用、504 网关超时。

⚠️ **原则：客户端的错不要返 5xx。**

> 🗣️ **English answer（401 vs 403 —— 高频陷阱题）**
>
> "401 is about authentication — I don't know who you are. Either there's no token, or the token is invalid or expired. The right fix is to log in again.
>
> 403 is about authorization — I know exactly who you are, you're authenticated, but you don't have permission to do this. Logging in again won't help; you need a different role.
>
> So a regular user hitting an admin-only endpoint gets 403, not 401."

> 🗣️ **English answer（202 —— 异步场景，能顺带带到你的 MQ 经验）**
>
> "202 Accepted is the one I'd use for asynchronous work. If a request drops a message onto a queue and the actual processing happens later, returning 200 would be misleading — nothing is done yet. 202 says 'I've accepted this, it's in flight', and usually you return a resource URI or a job ID the client can poll for status."

> 🗣️ **English answer（客户端错误不该返 5xx）**
>
> "5xx should mean the server is at fault. If a client sends a malformed request and we return 500, we've made our problem out of their mistake — and it pollutes our error monitoring, because now genuine outages are buried in noise. Bad input should be validated up front and returned as 400. Seeing 500s caused by null pointers on unvalidated input is usually a sign validation is missing."

---

## Q6. URL 怎么设计 / URL design

**核心原则：URI 用名词表示资源，动作由 HTTP 动词表达。**

```
❌ GET  /getUserById?id=1        动词在 URL 里
❌ POST /createUser              动词冗余
❌ GET  /addItemAction           用 GET 做写操作
❌ POST /user/delete/1           动词 + 错误动词

✅ GET    /users/1
✅ POST   /users
✅ PUT    /users/1
✅ DELETE /users/1
```

- **资源名用复数**：`/users`
- **嵌套表达从属**：`GET /users/1/orders`
- **嵌套不超过两层**，再深用 query param：`GET /orders?userId=1&status=PAID`
- **小写 + 连字符**：`/order-items`
- **过滤分页排序用 query param**：`GET /users?page=2&size=20&sort=createTime,desc`
- **版本号**：`/api/v1/users`

> 🗣️ **English answer**
>
> "The rule I follow is that URIs are nouns and the verb lives in the HTTP method. So not `/getUserById` or `/createUser` — just `/users/1` with GET, and `/users` with POST. Resource names are plural, lowercase, hyphenated.
>
> Nesting expresses ownership — `/users/1/orders` is the orders belonging to user 1 — but I keep it to two levels. Anything deeper becomes a query parameter instead, so `/orders?userId=1&status=PAID`.
>
> Filtering, pagination and sorting all go in query parameters, never in the path. And I version the API in the path, `/api/v1/`, so breaking changes don't break existing clients."

---

## Q7. ⭐ 为什么不能用 GET 做写操作 / Why not GET for writes?

1. **HTTP 规范定义 GET 是 safe**，整条链路的组件都按这个假设行事
2. **会被意外触发**：浏览器预加载、爬虫、聊天软件链接预览、用户刷新页面
3. **会被缓存**：浏览器 / CDN / 反向代理都可能缓存，请求根本到不了服务端
4. **参数暴露**：进浏览器历史、Nginx access log、Referer header
5. **CSRF 风险**：一个 `<img src="/deleteUser/1">` 就能触发删除

**所以 `@RequestMapping` 不指定 method 是隐患** —— 它默认接受所有动词。

> 🗣️ **English answer**
>
> "Because the whole HTTP ecosystem assumes GET is safe, and a lot of things act on that assumption.
>
> First, it gets triggered accidentally. Browsers prefetch links, crawlers follow them, chat apps generate link previews, and users refresh pages. Every one of those would fire your write.
>
> Second, GET responses are cacheable. A browser, a CDN, or a reverse proxy can serve it from cache, which means your write silently never reaches the server.
>
> Third, the parameters end up in browser history, in access logs, and in the Referer header — so anything sensitive leaks.
>
> And it opens a CSRF hole: an image tag pointing at a delete endpoint is enough to trigger it.
>
> Related to that — I avoid a bare `@RequestMapping` without a method, because it accepts every verb by default. I always use the explicit annotations like `@PostMapping`."

---

## Q8. POST vs PUT vs PATCH

| | POST | PUT | PATCH |
|---|---|---|---|
| 语义 | 创建（服务端定 ID） | 全量替换 | 部分更新 |
| 幂等 | ❌ | ✅ | ⚠️ 看实现 |
| 传什么 | 新资源数据 | **完整**资源表示 | **只传要改的字段** |

> 🗣️ **English answer**
>
> "POST creates, and the server decides the ID. PUT is a full replacement at a known URI. PATCH is a partial update.
>
> The subtle one is PUT. Because it's a replacement, any field you don't send should be reset — so if you use PUT to change one field and only send that field, a strict implementation wipes the rest. That's why partial updates should use PATCH. A lot of teams use PUT loosely for partial updates; it works, but it's not what the spec says, and it's worth knowing the difference.
>
> On ID generation — I'd normally let the server generate it, either a database sequence or a snowflake ID, and use POST. Letting the client choose the ID means you have to handle collisions, and sequential guessable IDs also open up resource enumeration."

---

## Q9. 统一响应 + 全局异常处理 / Unified response and global exception handling

```java
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
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Result<Void> handleValidation(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldError().getDefaultMessage();
        return Result.fail(400, msg);
    }

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusiness(BusinessException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public Result<Void> handleAll(Exception e) {
        log.error("unexpected error", e);
        return Result.fail(500, "服务器内部错误");
    }
}
```

> 🗣️ **English answer**
>
> "I wrap every response in a consistent envelope — a code, a message, and a data payload — so the client always parses the same shape, and I centralize error handling in a `@RestControllerAdvice`. That way controllers only contain the happy path; validation failures, business exceptions, and unexpected errors are each mapped to the right status code in one place."

**⭐ 追问：HTTP 状态码和 body 里的 code 重复了吗？**

> 🗣️ "They're two different layers, not duplication. The HTTP status code is the transport-level outcome — API gateways, monitoring, and client retry logic all key off it. The code in the body is the business-level outcome, something like 'insufficient stock' or 'insufficient balance', which HTTP has no vocabulary for.
>
> The anti-pattern is returning HTTP 200 for everything and putting the real result only in the body. That breaks your gateway, your alerting, and your client's retry logic, because none of them can tell the request failed."

**追问：为什么不能把堆栈返给客户端？**

> 🗣️ "It's a security issue. A stack trace leaks your framework version, your package structure, sometimes SQL and table names — that's reconnaissance for an attacker. So the stack trace goes to the logs, and the client gets a sanitized message plus a trace ID they can quote when they report the problem."

---

## Q10. 参数校验和 DTO / Validation and DTOs

```java
public class UserCreateDTO {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 2, max = 20)
    private String name;

    @Email
    private String email;

    @NotNull
    @Min(0)
    private BigDecimal price;
}

@PostMapping
public Result<Long> create(@Valid @RequestBody UserCreateDTO dto) { ... }
```

> 🗣️ **English answer（为什么要 DTO，不直接用 Entity）**
>
> "Four reasons, and the first one is security. If I bind the request straight onto the entity, and the entity has fields like password or isAdmin, a client can set them just by including them in the JSON. That's an over-posting attack.
>
> Second, decoupling — the entity mirrors the table, and I don't want a schema change to silently break the API contract.
>
> Third, the field sets genuinely differ. On create there's no ID or createTime; on the response I must not return the password hash.
>
> Fourth, validation rules differ between create and update — required on one, optional on the other — and you can't hang two different rule sets off the same entity."

---

## Q11. REST 的局限 / When would you not use REST?

| 方案 | 适合 | 不适合 |
|---|---|---|
| REST | 资源型 CRUD、公开 API、需要 HTTP 缓存 | 复杂查询、聚合多资源 |
| GraphQL | 前端字段需求多变 | 缓存复杂、N+1 查询风险 |
| gRPC | **内部服务间调用**，低延迟高吞吐 | 浏览器直连支持差、调试不直观 |

**REST 的两个痛点**：over-fetching / under-fetching；动作型接口不好映射成资源。

**HATEOAS**：Richardson 成熟度模型 Level 3，响应带上相关操作链接。理论美，实践少 —— 知道这个词、知道大部分 API 停在 Level 2 就够。

> 🗣️ **English answer**
>
> "REST is a great fit for resource-shaped CRUD and public APIs, especially when you want HTTP caching to work for you. Where it struggles is over-fetching and under-fetching — either the response carries fields the client doesn't need, or the client has to make five calls to assemble one screen.
>
> For internal service-to-service calls where latency and throughput matter, I'd consider gRPC — Protobuf is a compact binary format and HTTP/2 gives you multiplexing, so it's meaningfully faster than JSON over HTTP/1.1. The trade-off is that it's not directly consumable from a browser and it's harder to debug by hand.
>
> GraphQL solves the fetching problem when frontend field requirements change a lot, but you give up straightforward HTTP caching and you have to actively manage N+1 query problems.
>
> One more practical point — not every operation maps cleanly onto a resource. Something like cancelling an order is arguably `PATCH /orders/1` with a status change, but `POST /orders/1/cancel` is far more readable. I don't think REST is worth being dogmatic about there."

---

## Q12. Spring 注解速查 + 手写模板 / Annotations and a template to memorize

| 注解 | 作用 |
|---|---|
| `@RestController` | `@Controller` + `@ResponseBody`，返回值序列化成 JSON |
| `@RequestMapping` | 类级别公共前缀 |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` / `@PatchMapping` | 方法级别，**明确动词** |
| `@PathVariable` | 取路径占位符 |
| `@RequestParam` | 取 query param，支持 `required` / `defaultValue` |
| `@RequestBody` | 取请求体（JSON → 对象） |
| `@Valid` | 触发 DTO 校验 |
| `@RestControllerAdvice` | 全局异常处理 |
| `@ResponseStatus` | 指定响应状态码 |

```java
@RestController
@RequestMapping("/api/v1/commodities")
public class CommodityController {

    private final CommodityService service;

    // 构造器注入：不可变、便于测试、循环依赖启动时暴露
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

⚠️ **关掉 IDE 自动补全，把这段练到能默写。**

> 🗣️ **边写边讲的英文口播（live coding 时用）**
>
> "I'll start with the controller. `@RestController` so the return values are serialized to JSON directly, and `@RequestMapping` at the class level for the shared prefix — I'll version it as `/api/v1/commodities`, plural noun.
>
> For the dependency I'm using constructor injection rather than field injection — it makes the field final, it's easier to test because I can pass a mock in directly, and circular dependencies fail at startup instead of at runtime.
>
> Then the read endpoint — `@GetMapping` with a path variable for the ID. For the list endpoint I'll take page and size as query parameters with defaults, so the client can omit them.
>
> For create I'll use `@PostMapping` with `@Valid @RequestBody` so the validation annotations on the DTO fire before my code runs, and I'll set the status to 201 Created rather than 200. And for delete, 204 No Content, since there's nothing meaningful to return."

---

## Q13. 反例分析 / Code review answer

面试官让你评审代码，或问"你见过什么不好的 API 设计"时：

```java
@Controller
public class CommodityController {
    @Resource
    OnlineShoppingCommodityDao dao;                    // ① 字段注入

    @RequestMapping("/addItemAction")                  // ② 动词进 URL
    public String addItemAction(                       // ③ 未指定 method → GET 可写
            @RequestParam("commodityId") long commodityId,   // ④ 参数散开，无校验
            @RequestParam("price") int price, ...) {
        dao.insertCommodity(...);                      // ⑤ Controller 直连 DAO
        return "add_commodity_success";                // ⑥ 无统一响应 / 状态码
    }
}
```

| 问题 | 改法 |
|---|---|
| ① 字段注入 | 构造器注入 |
| ② `/addItemAction` | `POST /api/v1/commodities` |
| ③ 不带 method | `@PostMapping` |
| ④ 六个 `@RequestParam` | 收成 DTO + `@Valid` |
| ⑤ 直连 DAO | 加 Service 层承载业务逻辑和事务边界 |
| ⑥ 返视图名 | 返 `201 Created` + `Location` |

> 🗣️ **English answer**
>
> "I reviewed some code recently that had a few of these problems together, and it's a good example.
>
> The endpoint was mapped as `/addItemAction` with a bare `@RequestMapping` and no method specified. So the verb was in the URL, and because no method was declared, a GET request could create a resource — which breaks the safety assumption the whole HTTP stack relies on.
>
> It took six separate request parameters with no validation, so any malformed input became a runtime exception rather than a 400.
>
> The controller talked to the DAO directly, so there was no service layer and therefore no clear transaction boundary.
>
> How I'd change it: `POST /api/v1/commodities`, an explicit `@PostMapping`, collapse the parameters into a DTO with validation annotations and `@Valid`, introduce a service layer for the business logic and the transaction, and return 201 with a Location header.
>
> One caveat though — if that's a server-rendered page rather than a JSON API, returning a view name is correct, and REST conventions don't really apply. The actual problem is mixing page rendering and API endpoints in the same controller."

---

## 一分钟自测 / Quick self-check

答不上来的回去看对应章节。**每题都用英文说一遍，不要只在心里过中文。**

1. PUT 和 POST 哪个幂等？为什么？
2. 401 和 403 的区别？
3. DELETE 第二次返 404 还算幂等吗？
4. 为什么不能用 GET 做写操作？（说三个理由）
5. 创建资源成功返什么状态码？带什么 header？
6. MQ 消费端怎么保证幂等？（说两种实现）
7. 为什么要 DTO 不直接用 Entity？
8. 所有接口都返 HTTP 200、错误只写在 body 里，有什么问题？
9. 什么场景下你会选 gRPC 而不是 REST？
10. 不看任何提示，手写一个带分页查询和创建的 RestController，**边写边用英文讲你在做什么**。