# Java Code Review Guide

> Java 代码审查指南。**金融 / 支付场景优先**过 [金融红线](custom/finance-java-block.md)，再按 [阿里巴巴 P3C](custom/ali-java/) 基础与专题规约；并覆盖 Java 17/21、Spring Boot 3、虚拟线程、JPA 性能与可维护性。文末有 [Review Checklist](#review-checklist) 快速对照。

## 目录

- [金融红线（优先）](#金融红线优先)
- [阿里巴巴 P3C 精简规范 (Tier2)](#阿里巴巴-p3c-精简规范-tier2)
- [现代 Java 特性 (17/21+)](#现代-java-特性-1721)
- [Stream API & Optional](#stream-api--optional)
- [Spring Boot 最佳实践](#spring-boot-最佳实践)
- [JPA 与 数据库性能](#jpa-与-数据库性能)
- [并发与虚拟线程](#并发与虚拟线程)
- [Lombok 使用规范](#lombok-使用规范)
- [异常处理](#异常处理)
- [测试规范](#测试规范)
- [Review Checklist](#review-checklist)

---

## 金融红线（优先）

金融 / 支付 / 账务相关代码 **必须先加载**，违反项直接 `blocking`。与 P3C 冲突时以本文件为准。

| 文件 | 覆盖范围 |
|------|----------|
| [finance-java-block.md](custom/finance-java-block.md) | 金额计算、账务事务、脱敏、加密验签、资金表、合规对照 |

---

## 阿里巴巴 P3C 精简规范 (Tier2)

通用 Java 业务代码在过金融红线后，按变更范围加载子规范（来源：[阿里巴巴 Java 开发手册](https://alibaba.github.io/p3c/)）。违反【强制】项建议标记 `blocking`。

### 基础规约（优先加载）

| 主题 | 文件 | 覆盖范围 |
|------|------|----------|
| 基础 | [ali-base.md](custom/ali-java/ali-base.md) | 命名、常量、格式、OOP、控制语句、注释 |
| 集合 | [ali-collection.md](custom/ali-java/ali-collection.md) | hashCode/equals、转换、遍历、Collectors |
| 并发 | [ali-concurrent.md](custom/ali-java/ali-concurrent.md) | 线程池、锁、ThreadLocal、定时任务 |
| 日期时间 | [ali-datetime.md](custom/ali-java/ali-datetime.md) | 格式化、闰年、sql 日期类型（嵩山版+） |
| 异常日志 | [ali-exception.md](custom/ali-java/ali-exception.md) | 异常处理、SLF4J 日志 |
| IO/文件 | [ali-io-file.md](custom/ali-java/ali-io-file.md) | 流关闭、编码、日志文件 |

### 专题规约（按 diff 加载）

| 主题 | 文件 | 覆盖范围 |
|------|------|----------|
| MySQL/ORM | [ali-sql-mysql.md](custom/ali-java/ali-sql-mysql.md) | 建表、索引、SQL、MyBatis |
| 错误码 | [ali-error-code.md](custom/ali-java/ali-error-code.md) | A/B/C 五位错误码（泰山版+） |
| 前后端 API | [ali-api.md](custom/ali-java/ali-api.md) | REST、JSON、Long 精度（嵩山版+） |
| 安全 | [ali-security.md](custom/ali-java/ali-security.md) | 鉴权、脱敏、CSRF、防刷 |
| 工程结构 | [ali-project.md](custom/ali-java/ali-project.md) | 分层、Maven 二方库、服务器 |
| 单元测试 | [ali-unit-test.md](custom/ali-java/ali-unit-test.md) | AIR 原则、覆盖率、Mock |
| 设计规约 | [ali-design.md](custom/ali-java/ali-design.md) | 方案/架构评审（设计向） |

**推荐加载顺序：** `finance-java-block`（金融场景）→ 基础规约表 → 与本次 diff 相关的专题表。

---

## 现代 Java 特性 (17/21+)

### Record (记录类)

```java
// ❌ 传统的 POJO/DTO：样板代码多
public class UserDto {
    private final String name;
    private final int age;

    public UserDto(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // getters, equals, hashCode, toString...
}

// ✅ 使用 Record：简洁、不可变、语义清晰
public record UserDto(String name, int age) {
    // 紧凑构造函数进行验证
    public UserDto {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }
}
```

### Switch 表达式与模式匹配

```java
// ❌ 传统的 Switch：容易漏掉 break，不仅冗长且易错
String type = "";
switch (obj) {
    case Integer i: // Java 16+
        type = String.format("int %d", i);
        break;
    case String s:
        type = String.format("string %s", s);
        break;
    default:
        type = "unknown";
}

// ✅ Switch 表达式：无穿透风险，强制返回值
String type = switch (obj) {
    case Integer i -> "int %d".formatted(i);
    case String s  -> "string %s".formatted(s);
    case null      -> "null value"; // Java 21 处理 null
    default        -> "unknown";
};
```

### 文本块 (Text Blocks)

```java
// ❌ 拼接 SQL/JSON 字符串
String json = "{\n" +
              "  \"name\": \"Alice\",\n" +
              "  \"age\": 20\n" +
              "}";

// ✅ 使用文本块：所见即所得
String json = """
    {
      "name": "Alice",
      "age": 20
    }
    """;
```

---

## Stream API & Optional

### 避免滥用 Stream

```java
// ❌ 简单的循环不需要 Stream（性能开销 + 可读性差）
items.stream().forEach(item -> {
    process(item);
});

// ✅ 简单场景直接用 for-each
for (var item : items) {
    process(item);
}

// ❌ 极其复杂的 Stream 链
List<Dto> result = list.stream()
    .filter(...)
    .map(...)
    .peek(...)
    .sorted(...)
    .collect(...); // 难以调试

// ✅ 拆分为有意义的步骤
var filtered = list.stream().filter(...).toList();
// ...
```

### Optional 正确用法

```java
// ❌ 将 Optional 用作参数或字段（序列化问题，增加调用复杂度）
public void process(Optional<String> name) { ... }
public class User {
    private Optional<String> email; // 不推荐
}

// ✅ Optional 仅用于返回值
public Optional<User> findUser(String id) { ... }

// ❌ 既然用了 Optional 还在用 isPresent() + get()
Optional<User> userOpt = findUser(id);
if (userOpt.isPresent()) {
    return userOpt.get().getName();
} else {
    return "Unknown";
}

// ✅ 使用函数式 API
return findUser(id)
    .map(User::getName)
    .orElse("Unknown");
```

---

## Spring Boot 最佳实践

### 依赖注入 (DI)

```java
// ❌ 字段注入 (@Autowired)
// 缺点：难以测试（需要反射注入），掩盖了依赖过多的问题，且不可变性差
@Service
public class UserService {
    @Autowired
    private UserRepository userRepo;
}

// ✅ 构造器注入 (Constructor Injection)
// 优点：依赖明确，易于单元测试 (Mock)，字段可为 final
@Service
public class UserService {
    private final UserRepository userRepo;

    public UserService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }
}
// 💡 提示：结合 Lombok @RequiredArgsConstructor 可简化代码，但要小心循环依赖
```

### 配置管理

```java
// ❌ 硬编码配置值
@Service
public class PaymentService {
    private String apiKey = "sk_live_12345";
}

// ❌ 直接使用 @Value 散落在代码中
@Value("${app.payment.api-key}")
private String apiKey;

// ✅ 使用 @ConfigurationProperties 类型安全配置
@ConfigurationProperties(prefix = "app.payment")
public record PaymentProperties(String apiKey, int timeout, String url) {}
```

---

## JPA 与 数据库性能

### N+1 查询问题

> 📖 通用原理和跨语言方案详见 [N+1 查询跨语言指南](cross-cutting/n-plus-one-queries.md)

```java
// ❌ FetchType.EAGER 或 循环中触发懒加载
// Entity 定义
@Entity
public class User {
    @OneToMany(fetch = FetchType.EAGER) // 危险！
    private List<Order> orders;
}

// 业务代码
List<User> users = userRepo.findAll(); // 1 条 SQL
for (User user : users) {
    // 如果是 Lazy，这里会触发 N 条 SQL
    System.out.println(user.getOrders().size());
}

// ✅ 使用 @EntityGraph 或 JOIN FETCH
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

### 事务管理

```java
// ❌ 在 Controller 层开启事务（数据库连接占用时间过长）
// ❌ 在 private 方法上加 @Transactional（AOP 不生效）
@Transactional
private void saveInternal() { ... }

// ✅ 在 Service 层公共方法加 @Transactional
// ✅ 读操作显式标记 readOnly = true (性能优化)
@Service
public class UserService {
    @Transactional(readOnly = true)
    public User getUser(Long id) { ... }

    @Transactional
    public void createUser(UserDto dto) { ... }
}
```

### Entity 设计

```java
// ❌ 在 Entity 中使用 Lombok @Data
// @Data 生成的 equals/hashCode 包含所有字段，可能触发懒加载导致性能问题或异常
@Entity
@Data
public class User { ... }

// ✅ 仅使用 @Getter, @Setter
// ✅ 自定义 equals/hashCode (通常基于 ID)
@Entity
@Getter
@Setter
public class User {
    @Id
    private Long id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        return id != null && id.equals(((User) o).id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

---

## 并发与虚拟线程

### 虚拟线程 (Java 21+)

```java
// ❌ 传统线程池处理大量 I/O 阻塞任务（资源耗尽）
ExecutorService executor = Executors.newFixedThreadPool(100);

// ✅ 使用虚拟线程处理 I/O 密集型任务（高吞吐量）
// Spring Boot 3.2+ 开启：spring.threads.virtual.enabled=true
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// 在虚拟线程中，阻塞操作（如 DB 查询、HTTP 请求）几乎不消耗 OS 线程资源
```

### 线程安全

```java
// ❌ SimpleDateFormat 是线程不安全的
private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

// ✅ 使用 DateTimeFormatter (Java 8+)
private static final DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// ❌ HashMap 在多线程环境会数据丢失（Java 7 及之前 resize 还可能死循环，Java 8 修复了死循环但仍非线程安全）
// ✅ 使用 ConcurrentHashMap
Map<String, String> cache = new ConcurrentHashMap<>();
```

---

## Lombok 使用规范

```java
// ❌ 滥用 @Builder 导致无法强制校验必填字段
@Builder
public class Order {
    private String id; // 必填
    private String note; // 选填
}
// 调用者可能漏掉 id: Order.builder().note("hi").build();

// ✅ 关键业务对象建议手动编写 Builder 或构造函数以确保不变量
// 或者在 build() 方法中添加校验逻辑 (Lombok @Builder.Default 等)
```

---

## 异常处理

### 全局异常处理

```java
// ❌ 到处 try-catch 吞掉异常或只打印日志
try {
    userService.create(user);
} catch (Exception e) {
    e.printStackTrace(); // 不应该在生产环境使用
    // return null; // 吞掉异常，上层不知道发生了什么
}

// ✅ 自定义异常 + @ControllerAdvice (Spring Boot 3 ProblemDetail)
public class UserNotFoundException extends RuntimeException { ... }

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ProblemDetail handleNotFound(UserNotFoundException e) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
    }
}
```

---

## 测试规范

### 单元测试 vs 集成测试

```java
// ❌ 单元测试依赖真实数据库或外部服务
@SpringBootTest // 启动整个 Context，慢
public class UserServiceTest { ... }

// ✅ 单元测试使用 Mockito
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock UserRepository repo;
    @InjectMocks UserService service;

    @Test
    void shouldCreateUser() { ... }
}

// ✅ 集成测试使用 Testcontainers
@Testcontainers
@SpringBootTest
class UserRepositoryTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    // ...
}
```

---

## Review Checklist

> 详细规则见上文 [金融红线](#金融红线优先)、[P3C 索引](#阿里巴巴-p3c-精简规范-tier2)。涉及资金时金融项全部必查；P3C【强制】违反标 `blocking`。

### 金融红线（支付 / 账务 / 资金，优先）

- [ ] 金额用 `BigDecimal` + `decimal`，禁止 `double`/`float`；字符串构造、显式 `scale`/`RoundingMode`
- [ ] 支付/转账/回调有幂等键与唯一约束，余额变更有乐观锁或行锁
- [ ] 资金操作有完整审计字段；跨服务有冲正/补偿；禁止删流水
- [ ] 日志/接口未泄露卡号、身份证、CVV、完整手机号
- [ ] 密钥走 KMS/配置中心；回调验签；CVV 不存库
- [ ] 资金表查询走索引，无全表扫/循环查流水

→ 完整清单：[finance-java-block.md](custom/finance-java-block.md)

### 阿里巴巴 P3C · 基础

- [ ] 命名/常量/OOP 符合规约（`@Override`、POJO 无默认值、布尔不加 `is`）
- [ ] 无魔法值；`equals` 用常量调用；包装类比较用 `equals`
- [ ] 集合：`equals`/`hashCode` 成对；禁止 foreach 中 remove；`Collectors.toMap` 有 merge 函数
- [ ] 线程池用 `ThreadPoolExecutor`，禁止 `Executors`/`new Thread`；`ThreadLocal` 已 `remove`
- [ ] 异常不吞、不在 `finally` 返回；日志用 SLF4J + 占位符
- [ ] 流/连接 try-with-resources 或 `finally` 关闭

→ 基础文件：[ali-base](custom/ali-java/ali-base.md) · [collection](custom/ali-java/ali-collection.md) · [concurrent](custom/ali-java/ali-concurrent.md) · [exception](custom/ali-java/ali-exception.md) · [io-file](custom/ali-java/ali-io-file.md)

### 阿里巴巴 P3C · 专题（按 diff 勾选）

- [ ] **DB/MyBatis**：无 `SELECT *`；`#{}` 非 `${}`；金额 `decimal`；订正前先 `SELECT`
- [ ] **API**：Long 大整数用 String 返回；错误含 `errorCode`/`errorMessage`；列表空返 `[]`
- [ ] **安全**：水平越权校验；参数校验；CSRF；敏感数据脱敏
- [ ] **单测**：在 `src/test/java`；AIR 原则；核心增量有覆盖

### 现代 Java 与 Spring

- [ ] 遵循 Java 17/21 新特性（Switch 表达式, Records, 文本块）
- [ ] 避免 `Date`/`Calendar`/`SimpleDateFormat`（见 [ali-datetime](custom/ali-java/ali-datetime.md)）
- [ ] Optional 仅用于返回值；集合优先 Stream / Collections API
- [ ] 构造器注入；`@ConfigurationProperties`；Controller 轻薄
- [ ] `@ControllerAdvice` / ProblemDetail 统一异常

### 数据库 & 事务

- [ ] 读操作 `@Transactional(readOnly = true)`
- [ ] 无 N+1（EAGER fetch 或循环调用）
- [ ] Entity 未滥用 `@Data`；`equals`/`hashCode` 正确
- [ ] 查询条件有索引覆盖；事务内无长耗时 RPC

### 并发与性能

- [ ] I/O 密集型考虑虚拟线程
- [ ] `ConcurrentHashMap` vs `HashMap` 使用正确
- [ ] 锁粒度合理；锁内无 I/O

### 可维护性

- [ ] 关键逻辑有单测（见 [ali-unit-test](custom/ali-java/ali-unit-test.md)）
- [ ] Slf4j 日志，无 `System.out`；无魔法值
