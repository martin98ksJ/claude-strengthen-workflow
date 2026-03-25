---
name: db-api-design
description: 数据库表设计和 RESTful API 接口设计规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 需求描述或功能设计文档
- 现有数据库 schema（如有）
- 现有 API 接口定义（如有）

## 职责（必须做）

### 表设计

- [ ] 表名蛇形复数（`user_accounts`），列名蛇形（`created_at`）；关联表用 `主表_从表` 命名（`user_roles`）
- [ ] 必备字段：`id`（主键）、`created_at`、`updated_at`；软删除用 `deleted_at`（nullable）
- [ ] 主键策略：自增 ID 适合单体，分布式用雪花 ID/UUID；对外暴露的 ID 考虑不可猜测性
- [ ] 字段类型精确：金额用 DECIMAL 不用 FLOAT，状态用 TINYINT + 注释枚举值，变长文本用 VARCHAR 设合理长度
- [ ] NOT NULL 为默认，允许 NULL 需有明确理由（如"尚未设置"语义区别于零值）
- [ ] 外键：逻辑外键（应用层维护）为主，物理外键按项目约定；关联字段命名 `<关联表单数>_id`（`user_id`）

### 索引

- [ ] 查询驱动建索引：WHERE/JOIN/ORDER BY 高频字段建索引
- [ ] 复合索引遵循最左前缀原则，区分度高的字段在前
- [ ] 唯一约束用唯一索引实现，不依赖应用层去重
- [ ] 避免过度索引：单表索引不超过 5-6 个，写多读少的表更要克制

### 迁移

- [ ] 先加后删：新增列 → 代码适配 → 确认无引用 → 删除旧列
- [ ] 不直接改列类型：新增列 → 迁移数据 → 切换引用 → 删旧列
- [ ] 每次迁移可回滚，迁移脚本和回滚脚本成对出现
- [ ] 大表加列/加索引评估锁表影响，必要时用 Online DDL / pt-osc

### API 路径与方法

- [ ] RESTful 风格：资源名复数名词（`/users`），嵌套不超过 2 层（`/users/{id}/orders`）
- [ ] 方法语义：GET 查询、POST 创建、PUT 全量更新、PATCH 部分更新、DELETE 删除
- [ ] 版本控制：路径前缀 `/api/v1/`，大版本变更才升级
- [ ] 非 CRUD 操作用动词子资源（`POST /orders/{id}/cancel`），不在 URL 里放动词

### API 响应

- [ ] 默认响应体：`{ code, data, message }`，成功 code=0，错误时 code 非零 + message 描述原因
- [ ] 流式（SSE/WebSocket）、代理透传、第三方兼容接口例外需在文档中声明格式
- [ ] 错误响应包含 error code（机器可读）+ message（人可读），敏感信息不暴露（不返回堆栈/SQL）
- [ ] 空集合返回 `[]` 不返回 `null`，缺失资源返回 404

### 分页与过滤

- [ ] 分页参数统一 `page/pageSize`（偏移分页）或 `cursor/limit`（游标分页），与 `BasePageRequest` 对齐
- [ ] 设最大 `pageSize` 限制（如 100），未传时用默认值（如 20），防止一次拉全表
- [ ] 分页响应统一 `{ list, total, page, pageSize }`，空结果 list 返回 `[]`
- [ ] 简单筛选用 GET query string（`?status=active&sort=created_at:desc`），复杂筛选（多条件/数组/嵌套）用 POST body
- [ ] 时间范围筛选：参数命名 `start_time/end_time`
  > 时间格式规范详见 backend-conventions
- [ ] 排序参数：`sort=field:asc|desc`，默认按 `created_at:desc`，支持的排序字段需白名单校验（防注入）

### 幂等与安全

- [ ] POST 创建用幂等键（`Idempotency-Key` header 或业务唯一字段）防重复提交
- [ ] PUT/DELETE 天然幂等，重复调用结果一致
- [ ] 批量操作设上限（如单次最多 100 条），超限返回明确错误
- [ ] 敏感操作（删除/状态变更）记录操作日志（who/when/what）

## 边界（不做什么）

- 不涉及具体 ORM 框架用法（GORM/MyBatis/Prisma 等 → 按项目约定）
- 不涉及数据库性能调优（慢查询分析、执行计划、分库分表）
- 不涉及认证鉴权实现细节（→ 见 auth 相关设计）
- 不涉及 GraphQL/gRPC 等非 REST 协议

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
