---
name: env-strategy
description: dev/test/pre/prod 四环境的分布定义、配置分层和回滚规则
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 项目的部署架构或配置文件
- 涉及的环境变更描述

## 职责（必须做）

- [ ] 环境分布定义：
  - **dev**：本地开发，mock 外部依赖，热重载，日志 DEBUG
  - **test**：CI 自动化测试，独立数据库，可随时重置，日志 INFO
  - **pre**：预发布，与 prod 同配置，脱敏数据副本，日志 INFO
  - **prod**：生产环境，最小权限原则，日志 WARN（关键路径 INFO）
- [ ] 配置分层：默认值 < config.{env}.yaml < 环境变量 < 密钥注入；敏感信息（密钥/token/密码）不入代码库，走密钥管理或环境变量
- [ ] 同构推进：同一构建产物（同镜像 digest）从 test → pre → prod 逐级推进，不重新 build
- [ ] 数据隔离：各环境独立数据库/缓存/存储，禁止 dev 连 prod，test 数据可随时重置
- [ ] 最小回滚：版本回滚（回退到上一 tag/digest）+ 配置回滚（git revert 环境配置），两条路径都必须可执行
- [ ] 最小兜底：每个环境定义超时默认值、熔断阈值、降级开关，prod 必须有降级预案

## 边界（不做什么）

- 不涉及完整发布审批流程和门禁矩阵
- 不涉及 SLO/SLA 定义和 oncall 体系
- 不涉及具体 CI/CD 工具配置（Jenkins/GitHub Actions 等）
- 不涉及监控平台搭建（Prometheus/Grafana 等）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
