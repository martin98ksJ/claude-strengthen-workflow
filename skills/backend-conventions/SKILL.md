---
name: backend-conventions
description: 后端开发时的架构分层、时间处理、数据传输对象、错误处理、日志和安全规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 后端代码文件（Go/Java/Python/Node 等）
- 涉及的模块或功能描述

## 职责（必须做）

### 架构分层

- [ ] controller/handler → service → repository/store，禁止跨层调用
- [ ] controller 只做参数绑定、校验、调用 service、组装响应，不写业务逻辑

### 时间处理

- [ ] 日期时间格式统一 `yyyy-MM-dd HH:mm:ss`（如 `2025-01-15 09:30:00`），纯日期 `yyyy-MM-dd`
- [ ] 数据库 datetime 列存本地时间（或 UTC，项目统一选一种，不混用）
- [ ] 时间范围筛选参数统一命名 `start_time` / `end_time`：
  - `start_time` 未指定时间部分时，默认补 `00:00:00`（当天开始）
  - `end_time` 未指定时间部分时，默认补 `23:59:59`（当天结束）
  - 前端传日期选择器的值时，后端负责补全时分秒
- [ ] 时间比较用时间类型（不用字符串比较），序列化/反序列化统一格式
- [ ] 项目内封装时间工具函数（获取当前时间、格式化、解析），不在业务代码里直接硬编码格式字符串

### 数据传输对象（DTO/VO）

- [ ] 入参统一用 DTO/Request 对象接收，不用 Map/散参数；字段加校验注解/tag
- [ ] 出参按需选择：
  - 简单场景（字段与 Entity 基本一致）：可直接返回 Entity（隐藏敏感字段）
  - 需要裁剪/聚合/脱敏：用 VO/Response 对象包装
- [ ] DTO 和 VO 不复用同一个类，入参和出参职责分离
- [ ] 通用字段抽基类/嵌套结构：`BasePageRequest`（page/pageSize）、`BaseResponse`（code/message/data）
- [ ] 列表接口统一返回 `{ list, total, page, pageSize }`，不返回裸数组

### HTTP 方法选择

- [ ] GET：无副作用的查询（单条详情、列表查询、简单筛选）
- [ ] POST：创建资源、复杂查询（参数多/有嵌套/含敏感信息不宜放 URL）、执行操作（导入/导出/触发任务）
- [ ] PUT：全量更新（传完整对象）
- [ ] PATCH：部分更新（只传变更字段）
- [ ] DELETE：删除资源
- [ ] 实际项目中大部分列表查询带筛选条件时用 POST（参数多、可能含数组/嵌套），简单的 ID 查询和无参列表用 GET

### 错误处理

- [ ] 错误必须包装上下文后向上传递，禁止吞异常，业务错误和系统错误分离
- [ ] 对外 API 使用统一错误码体系，内部用 error wrap

### 日志与可观测

- [ ] 结构化日志（JSON/KV），关键路径必须有日志，含 trace id
- [ ] 关键操作埋点（耗时、成功/失败计数），错误日志包含足够定位信息

### 输入校验与安全

- [ ] 所有外部输入（API 参数、配置、文件）在入口处校验
- [ ] 配置优先级：环境变量 > 配置文件 > 默认值，敏感信息不入代码库

## 边界（不做什么）

- 不涉及具体语言语法和惯用法（见对应语言 skill：go/java-conventions）
- 不涉及部署运维和环境管理（见 env-strategy）
- 不涉及前端代码
- 不涉及数据库表设计和 API 路径/响应设计（见 db-api-design）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
