---
name: rust-conventions
description: Rust 项目的所有权、错误处理、并发、模块组织和测试规范
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用后端架构规范见 backend-conventions

## 输入

- Rust 源码文件
- 涉及的 crate 或模块描述

## 职责（必须做）

### 命名

- [ ] crate/模块名蛇形（`user_service`），类型名大驼峰（`UserService`），函数/变量蛇形（`get_user_by_id`）
- [ ] 常量 UPPER_SNAKE（`MAX_RETRY_COUNT`），生命周期参数用短小写（`'a`、`'ctx`）
- [ ] trait 用形容词或动词能力命名（`Serialize`、`Readable`、`IntoIterator`），不加 `I` 前缀
- [ ] 构造函数 `new()`/`with_xxx()`，转换方法 `into_xxx()`/`as_xxx()`/`to_xxx()`（遵循 std 惯例）
- [ ] Builder 模式用于参数多的构造：`XxxBuilder::new().field(v).build()`

### 所有权与借用

- [ ] 默认借用（`&T`/`&mut T`），只在需要所有权转移时用 move
- [ ] 函数参数：只读用 `&T`，需修改用 `&mut T`，需要拥有用 `T`；返回值优先返回拥有的值
- [ ] 避免不必要的 `clone()`：先考虑能否用引用或生命周期解决，clone 需有注释说明理由
- [ ] 字符串参数：接收用 `&str`（通用），返回用 `String`（拥有所有权）；`impl AsRef<str>` 用于泛型接口
- [ ] 智能指针选择：`Box<T>`（堆分配）、`Rc<T>`/`Arc<T>`（共享所有权）、`Cow<'a, T>`（按需克隆）

### 错误处理

- [ ] 可恢复错误用 `Result<T, E>`，不可恢复用 `panic!`（仅限程序员错误/不变量违反）
- [ ] 自定义错误类型用 `thiserror` 派生，应用层顶级错误用 `anyhow::Result`（快速开发）或自定义枚举（库）
- [ ] 错误传播用 `?` 操作符，不手动 match + return Err；错误上下文用 `.context("doing xxx")`（anyhow）或 `map_err`
- [ ] 库 crate 不用 `anyhow`，定义具体错误枚举让调用方可以 match 处理
- [ ] `unwrap()`/`expect()` 仅限：测试代码、确定不会 None/Err 的场景（附注释说明为什么安全）

### 并发

- [ ] 异步运行时选型：tokio（通用）/ async-std，项目统一一个，不混用
- [ ] `async fn` 返回 `impl Future`，避免在 async 块中持有 `MutexGuard` 跨 `.await`（用 tokio::sync::Mutex 或提前释放）
- [ ] 共享状态：`Arc<Mutex<T>>`（简单场景）、`Arc<RwLock<T>>`（读多写少）、`DashMap`（高并发 Map）
- [ ] channel 选型：`tokio::sync::mpsc`（多生产者单消费者）、`broadcast`（多消费者）、`oneshot`（一次性响应）
- [ ] 任务生命周期：`tokio::spawn` 的任务必须有取消机制（`CancellationToken`/`select!`），优雅关闭时等待任务完成

### 模块与项目结构

- [ ] workspace 管理多 crate：`crates/` 目录下按功能拆分（`crates/core/`、`crates/api/`、`crates/cli/`）
- [ ] 模块可见性最小化：默认私有，`pub` 只暴露必要接口，`pub(crate)` 用于 crate 内共享
- [ ] `lib.rs` 作为 crate 公共 API 入口，re-export 关键类型，内部模块结构对外不可见
- [ ] 依赖管理：`Cargo.toml` 指定精确版本或兼容范围（`"1.2"`），workspace 用 `[workspace.dependencies]` 统一版本
- [ ] feature flag 控制可选功能，默认 feature 最小化，文档说明每个 feature 的作用

### Trait 与泛型

- [ ] trait 职责单一（小 trait 优于大 trait），组合用 `trait A: B + C`
- [ ] 泛型约束用 `where` 子句（参数多时），简单约束内联（`fn foo<T: Display>(t: T)`）
- [ ] 优先用 `impl Trait`（参数位置/返回位置）简化签名，需要动态分发时用 `dyn Trait`
- [ ] 为自定义类型实现标准 trait：`Debug`（必须）、`Clone`/`Display`/`Default`（按需）、`Serialize/Deserialize`（需序列化时）

### 测试

- [ ] 单元测试放模块内 `#[cfg(test)] mod tests`，集成测试放 `tests/` 目录
- [ ] 测试函数 `#[test] fn test_xxx_when_yyy()`，异步测试 `#[tokio::test]`
- [ ] 断言用 `assert_eq!/assert_ne!/assert!`，自定义错误信息 `assert!(cond, "expected {} got {}", a, b)`
- [ ] mock 用 `mockall` crate 或手写 trait 实现，测试 trait 不测具体类型
- [ ] 属性测试（property-based）用 `proptest`/`quickcheck` 覆盖边界组合

### 性能

- [ ] 零成本抽象：优先用迭代器链（`iter().filter().map().collect()`）而非手写循环，编译器会优化
- [ ] 预分配：`Vec::with_capacity(n)`、`String::with_capacity(n)`，已知大小时避免多次扩容
- [ ] 避免不必要的堆分配：小数据用栈（数组/元组），`SmallVec`/`ArrayVec` 用于通常很小但偶尔大的集合
- [ ] 热路径避免 `format!()`（会分配），用 `write!` 到预分配 buffer
- [ ] 编译优化：release 构建开 LTO（`lto = true`），`codegen-units = 1`（更好优化），按需 `opt-level`

### 常见坑（AI 生成代码高频踩雷）

- [ ] 所有权移动后使用：变量 move 后再访问编译报错，AI 容易生成 `let b = a; println!("{}", a)`；需要共享用 `clone()` 或引用
- [ ] 生命周期省略误判：函数返回引用时，编译器按省略规则推断生命周期，多参数时可能推断错误；复杂情况显式标注 `'a`
- [ ] async + Mutex 死锁：`tokio::sync::Mutex` 的 `MutexGuard` 跨 `.await` 持有，其他任务在同一线程等锁导致死锁；用 `{ let guard = lock.lock().await; ... }` 限制作用域
- [ ] unwrap 在生产代码：AI 爱生成 `.unwrap()`/`.expect()`，生产环境 panic 导致进程崩溃；改用 `?` 或 `match`/`if let`
- [ ] String vs &str 转换开销：频繁 `to_string()`/`clone()` 造成不必要堆分配；函数参数用 `&str`/`impl AsRef<str>` 避免调用方被迫分配
- [ ] 整数溢出：debug 模式 panic，release 模式静默 wrap；需要明确行为用 `checked_add()`/`saturating_add()`/`wrapping_add()`
- [ ] 闭包捕获所有权：`move || {}` 闭包会 move 所有引用的变量，可能意外拿走不该拿的；只 move 需要的变量，其余用 `let ref_x = &x` 预先借用
- [ ] Iterator 惰性求值：`iter().map().filter()` 不调用 `collect()`/`for_each()` 不执行任何操作；AI 生成的链式调用可能缺少终结操作
- [ ] Send/Sync 约束：`Rc<T>` 不能跨线程（不实现 Send），`Arc<T>` 可以；`tokio::spawn` 要求 `Send + 'static`，容易编译报错
- [ ] 模式匹配遗漏：`match` 非穷尽会编译错误，但 `if let` 会静默忽略其他分支；关键逻辑用 `match` 强制处理所有情况

## 边界（不做什么）

- 不涉及通用后端架构（分层、日志、安全 → 见 backend-conventions）
- 不涉及 unsafe Rust 和 FFI（除非明确需要，需额外审查）
- 不涉及嵌入式/no_std 场景
- 不涉及具体 Web 框架用法（Axum/Actix/Rocket → 按项目约定）
- 不涉及部署和环境配置（→ 见 env-strategy、docker-deploy）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
