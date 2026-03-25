---
name: go-conventions
description: Go 项目的命名、错误处理、并发、接口设计和测试规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用后端架构规范见 backend-conventions

## 输入

- Go 源码文件
- 涉及的包或模块描述

## 职责（必须做）

### 命名

- [ ] 包名小写单词不用下划线，目录名即包名，不用 common/util/misc 等无意义包名
- [ ] 接口用 -er 后缀（Reader/Writer/Handler），只定义使用方需要的方法（小接口优于大接口）
- [ ] 未导出小驼峰，导出大驼峰；缩写词全大写或全小写（`HTTPClient` / `httpClient`，不写 `HttpClient`）
- [ ] 变量名长度与作用域成正比：循环变量 `i/v`，局部变量简短，包级变量完整描述

### 错误处理

- [ ] `fmt.Errorf("xxx: %w", err)` 包装上下文，`errors.Is/As` 判断
- [ ] 不用 panic 做流控，panic 仅限程序员错误（不可恢复的前置条件违反）
- [ ] 默认不忽略 error，例外需注释说明（如 `_ = f.Close() // best-effort cleanup`）
- [ ] 自定义错误类型实现 `Error()` 和 `Unwrap()`，区分业务错误和系统错误
- [ ] 错误信息小写开头、不以标点结尾，便于 wrap 后拼接可读

### 并发

- [ ] goroutine 必须通过 context 或 done channel 可退出，禁止裸 `go func()` 无退出机制
- [ ] 用 `sync.WaitGroup` 或 `errgroup.Group` 管理 goroutine 生命周期，确保优雅退出
- [ ] 共享状态优先 channel，需要保护临界区时用 mutex；mutex 保护的字段紧跟 mutex 声明
- [ ] channel 有明确的 owner（谁创建谁关闭），不在接收方关闭 channel

### 接口与依赖

- [ ] 接口定义在使用方，不在实现方（Go 隐式接口的正确用法）
- [ ] 依赖通过构造函数注入（`func NewService(repo Repository) *Service`），不在函数内部直接构造
- [ ] context.Context 作为函数第一个参数传递，不存到 struct 里

### 项目结构

- [ ] `cmd/`（入口）`internal/`（私有）`pkg/`（公共库），不在 main 包写业务逻辑
- [ ] 按功能域划分包（`user/`、`order/`），不按技术层划分（不建议顶层 `models/`、`controllers/`）
- [ ] `internal/` 内的包可以有自己的分层（handler/service/store），但包间依赖单向

### 测试

- [ ] table-driven tests，子测试用 `t.Run(name, func(t *testing.T){...})`
- [ ] mock 接口不 mock 实现，测试文件 `_test.go` 同目录
- [ ] 测试函数名 `Test<Function>_<scenario>`（如 `TestCreateUser_DuplicateEmail`）
- [ ] 测试辅助函数用 `t.Helper()` 标记，错误信息包含足够上下文

### 时间处理（Go 特有）

- [ ] 格式化/解析用 `time.Format` / `time.Parse`，layout 基于参考时间 `2006-01-02 15:04:05`（不是 yyyy-MM-dd）
- [ ] 统一格式常量：`const TimeFormat = "2006-01-02 15:04:05"`、`const DateFormat = "2006-01-02"`，不在业务代码里硬编码 layout
- [ ] 时间类型用 `time.Time`，不用 string 存时间；JSON 序列化需要自定义格式时实现 `MarshalJSON/UnmarshalJSON`
- [ ] 时间比较用 `time.Before/After/Equal`，不用 `==`（`==` 会比较 monotonic clock 和 Location）
- [ ] 获取当前时间统一入口（便于测试 mock），不在业务代码里散落 `time.Now()`

### 性能

- [ ] 字符串拼接用 `strings.Builder`，slice 已知容量时 `make([]T, 0, cap)` 预分配
- [ ] 热路径考虑 `sync.Pool` 复用临时对象，defer 在热路径中注意性能开销
- [ ] 大 struct 传指针，小 struct（≤3 字段且无引用语义）传值

### 常见坑（AI 生成代码高频踩雷）

- [ ] goroutine 泄漏：`go func()` 里的 channel 读写/HTTP 请求没有 timeout 或 context 取消，goroutine 永远阻塞不退出
- [ ] slice 陷阱：`append` 可能共享底层数组，子 slice 修改影响原 slice；需要独立副本时用 `copy` 或 `slices.Clone`
- [ ] nil slice vs empty slice：`var s []int`（nil）和 `s := []int{}`（empty）JSON 序列化结果不同（`null` vs `[]`），对外接口统一用 `make([]T, 0)`
- [ ] nil map 写入 panic：`var m map[string]int` 后直接 `m["key"] = 1` 会 panic，必须 `make(map[string]int)`
- [ ] defer 循环陷阱：`for { f := open(); defer f.Close() }` 文件句柄不会及时释放，应提取为函数或手动 close
- [ ] 循环变量捕获（Go < 1.22）：goroutine/闭包里引用循环变量，拿到的是最后一个值；Go 1.22+ 已修复，低版本需 `v := v` 拷贝
- [ ] interface nil 陷阱：`var err *MyError = nil; var e error = err; e != nil` 为 true，因为 interface 持有类型信息；返回 error 时直接 `return nil`
- [ ] time.After 泄漏：`select { case <-time.After(5s): }` 在循环中每次创建新 timer 不回收，用 `time.NewTimer` + `Reset`
- [ ] 并发 map 读写 panic：`map` 不是并发安全的，多 goroutine 读写会 `fatal error`；用 `sync.Map` 或 `sync.RWMutex` 保护
- [ ] string 遍历是 byte 不是 rune：`for i, c := range s` 的 c 是 rune，但 `s[i]` 是 byte；中文/emoji 处理用 `[]rune(s)` 转换

## 边界（不做什么）

- 不涉及通用后端架构（分层、日志、安全 → 见 backend-conventions）
- 不涉及部署和环境配置（见 env-strategy）
- 不涉及 CGO 和底层系统编程
- 不涉及特定框架用法（Gin/Echo/GORM 等 → 按项目 CLAUDE.md 约定）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
