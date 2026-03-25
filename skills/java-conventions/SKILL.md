---
name: java-conventions
description: Java/Spring 项目的命名、异常、分层、并发和测试规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用后端架构规范见 backend-conventions

## 输入

- Java 源码文件
- 涉及的模块或 Spring 组件描述

## 职责（必须做）

### 命名

- [ ] 类大驼峰（`UserService`），方法小驼峰（`getUserById`），常量 UPPER_SNAKE（`MAX_RETRY_COUNT`），包名全小写
- [ ] Boolean 类型：字段/方法用 `is/has/can/should` 前缀（`isActive`、`hasPermission`）
- [ ] 集合变量用复数或 List/Map 后缀（`users`、`userMap`），不用 `list`、`data` 等无意义名
- [ ] 接口不加 `I` 前缀，实现类用有意义后缀（`JdbcUserRepository`，非 `UserRepositoryImpl`）

### 异常处理

- [ ] 业务异常（`BizException`）vs 系统异常（`SysException`）分离，业务异常携带错误码
- [ ] 不用 `catch(Exception)` 兜底，捕获最具体的异常类型
- [ ] 异常信息包含上下文（谁/做什么/为什么失败），不只抛 "操作失败"
- [ ] `@RestControllerAdvice` 统一异常处理，Controller 不 try-catch 业务异常
- [ ] 受检异常在 service 层转为非受检异常向上抛，不让受检异常污染调用链

### Spring 规范

- [ ] 构造器注入为主（`final` 字段 + `@RequiredArgsConstructor`），禁止字段注入
- [ ] 配置类 `@Configuration` 独立，`@Value` 集中到 `@ConfigurationProperties` 类
- [ ] Bean 作用域默认 singleton，有状态 Bean 必须显式声明 scope
- [ ] `@Async` 方法必须在独立类中（避免同类调用不走代理），返回 `CompletableFuture`

### 分层

- [ ] Controller → Service → Repository，禁止跨层调用（Controller 不直接访问 Repository）
- [ ] DTO（传输）/ VO（视图）/ Entity（持久化）分离，禁止 Entity 直接返回给前端
- [ ] 转换逻辑用 MapStruct 或手写静态方法，不在 Service 里逐字段赋值
- [ ] Service 接口 + 实现分离（便于 mock），简单 CRUD 可直接用实现类

### 事务

- [ ] `@Transactional` 只加在 Service 层公开方法上，明确 `rollbackFor = Exception.class`
- [ ] 只读操作用 `readOnly = true`，避免长事务（事务内不做 RPC/IO）
- [ ] 编程式事务（`TransactionTemplate`）用于需要细粒度控制的场景
- [ ] 事务方法不捕获异常后吞掉（会导致事务不回滚）

### 并发

- [ ] 线程池用 `ThreadPoolExecutor` 自定义（拒绝策略、队列大小），禁止 `Executors.newXxx()`
- [ ] 共享可变状态用 `ConcurrentHashMap`/`AtomicXxx`，不在同步块里做 IO
- [ ] `CompletableFuture` 链式调用指定线程池（`xxxAsync(task, executor)`），不用默认 ForkJoinPool

### 时间处理（Java 特有）

- [ ] 日期时间用 `LocalDateTime`，纯日期用 `LocalDate`，纯时间用 `LocalTime`；禁止使用 `Date`/`Calendar`
- [ ] 格式化用 `DateTimeFormatter`，定义为 `static final` 常量复用（线程安全），不每次 new
- [ ] JSON 序列化统一格式：`@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")`，或全局配置 `ObjectMapper`
- [ ] 数据库映射：JPA 自动映射 `LocalDateTime` ↔ `datetime`；MyBatis 用 `LocalDateTimeTypeHandler`
- [ ] 时区：服务端统一时区（`TimeZone.setDefault` 或 JVM 参数 `-Duser.timezone`），跨时区场景用 `ZonedDateTime`

### 测试

- [ ] JUnit5 + Mockito 单测为主，`@SpringBootTest` 仅集成测试用（启动慢）
- [ ] 测试方法名 `should_expectedBehavior_when_condition`（如 `should_throwBizException_when_userNotFound`）
- [ ] 覆盖核心业务路径 + 异常路径 + 边界值
- [ ] 测试数据用 Builder/Factory 构建，不硬编码大量字段

### 常见坑（AI 生成代码高频踩雷）

- [ ] NPE 三大重灾区：`map.get()` 返回 null 直接拆箱、`Optional.get()` 不判断 `isPresent`、集合流操作返回 null 元素
- [ ] equals 陷阱：`Integer` 缓存范围 -128~127，超出范围 `==` 比较为 false；对象比较一律用 `Objects.equals()`
- [ ] 日期格式化线程不安全：`SimpleDateFormat` 非线程安全，多线程共享会数据错乱；用 `DateTimeFormatter`（不可变线程安全）
- [ ] BigDecimal 构造陷阱：`new BigDecimal(0.1)` 精度丢失（0.10000000000000000555...），用 `BigDecimal.valueOf(0.1)` 或 `new BigDecimal("0.1")`
- [ ] ConcurrentModificationException：遍历集合时直接 `list.remove()` 抛异常；用 `Iterator.remove()`、`removeIf()` 或 `CopyOnWriteArrayList`
- [ ] Stream 重复消费：Stream 只能消费一次，第二次 `forEach/collect` 抛 `IllegalStateException`；需要多次消费用 `Supplier<Stream>`
- [ ] @Transactional 失效：同类内部方法调用不走代理，事务不生效；private 方法加 `@Transactional` 无效；catch 异常后没 rethrow 导致不回滚
- [ ] 泛型擦除：`List<String>` 和 `List<Integer>` 运行时是同一类型，不能用 `instanceof List<String>` 判断；反序列化泛型用 `TypeReference`
- [ ] HashMap 容量：默认初始容量 16，负载因子 0.75，预知大小时 `new HashMap<>(expectedSize * 4 / 3 + 1)` 避免反复扩容
- [ ] String 拼接：循环内 `+=` 拼接每次创建新对象，O(n²)；用 `StringBuilder` 或 `String.join()`/`Collectors.joining()`

## 边界（不做什么）

- 不涉及通用后端架构（分层原则、日志、安全 → 见 backend-conventions）
- 不涉及非 Spring 框架（Quarkus/Micronaut/Vert.x 等）
- 不涉及 Java 底层（JVM 调优、GC、字节码）
- 不涉及特定 ORM 框架用法（MyBatis/JPA 细节 → 按项目约定）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
