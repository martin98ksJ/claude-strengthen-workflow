# Skill Packs — 按技术栈分组

根据变更文件后缀或项目类型，加载对应的 skill pack。**只加载命中的 pack，不全量加载。**

## 使用方式

1. 检测变更文件后缀（`git diff --name-only` 或用户指定）
2. 按下表匹配 pack
3. 用 Read 工具读取 pack 中列出的 skill 文件

## Pack 定义

### Go 后端

触发：`.go` 文件变更 或 `go.mod` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| go-conventions | `~/.claude/skills/go-conventions/SKILL.md` | 命名/错误处理/并发/常见坑 |
| backend-conventions | `~/.claude/skills/backend-conventions/SKILL.md` | 分层/时间/DTO/错误 |

### Java 后端

触发：`.java` 文件变更 或 `pom.xml`/`build.gradle` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| java-conventions | `~/.claude/skills/java-conventions/SKILL.md` | 命名/异常/Spring/事务/常见坑 |
| backend-conventions | `~/.claude/skills/backend-conventions/SKILL.md` | 分层/时间/DTO/错误 |

### Python 后端

触发：`.py` 文件变更 或 `pyproject.toml`/`requirements.txt` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| python-conventions | `~/.claude/skills/python-conventions/SKILL.md` | 命名/类型提示/异步/常见坑 |
| backend-conventions | `~/.claude/skills/backend-conventions/SKILL.md` | 分层/时间/DTO/错误 |

### Rust 后端

触发：`.rs` 文件变更 或 `Cargo.toml` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| rust-conventions | `~/.claude/skills/rust-conventions/SKILL.md` | 所有权/错误处理/并发/常见坑 |
| backend-conventions | `~/.claude/skills/backend-conventions/SKILL.md` | 分层/时间/DTO/错误 |

### Vue 前端

触发：`.vue` 文件变更 或 `vite.config`/`vue.config` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| vue-conventions | `~/.claude/skills/vue-conventions/SKILL.md` | 组合式API/组件/Element Plus/常见坑 |
| frontend-conventions | `~/.claude/skills/frontend-conventions/SKILL.md` | 四态/样式/i18n/常见坑 |
| ui-ue-guidelines | `~/.claude/skills/ui-ue-guidelines/SKILL.md` | 常见坑/实战模式/四态/表单/弹窗 |
| frontend-ui-design | `~/.claude/skills/frontend-ui-design/SKILL.md` | 新页面时：线框图/组件清单/交互/i18n |

### React 前端

触发：`.tsx`/`.jsx` 文件变更 或 `next.config` 存在

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| react-conventions | `~/.claude/skills/react-conventions/SKILL.md` | Hooks/组件/状态管理/常见坑 |
| frontend-conventions | `~/.claude/skills/frontend-conventions/SKILL.md` | 四态/样式/i18n/常见坑 |
| ui-ue-guidelines | `~/.claude/skills/ui-ue-guidelines/SKILL.md` | 常见坑/实战模式/四态/表单/弹窗 |
| frontend-ui-design | `~/.claude/skills/frontend-ui-design/SKILL.md` | 新页面时：线框图/组件清单/交互/i18n |

### 移动端/跨端

触发：`.dart`/`.wxml`/`.swift`/`.kt` 文件变更

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| mobile-cross-platform | `~/.claude/skills/mobile-cross-platform/SKILL.md` | 架构/导航/状态/平台适配 |

### TypeScript 后端

触发：`.ts` 文件变更（排除 `.tsx`）且项目无 Vue/React 前端框架标志

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| backend-conventions | `~/.claude/skills/backend-conventions/SKILL.md` | 分层/时间/DTO/错误 |

### SQL 脚本

触发：`.sql` 文件变更

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| db-api-design | `~/.claude/skills/db-api-design/SKILL.md` | 表设计/索引/迁移 |

### Docker 部署

触发：`Dockerfile` 或 `.dockerfile` 或 `docker-compose*.yml` 变更

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| docker-deploy | `~/.claude/skills/docker-deploy/SKILL.md` | 多阶段构建/健康检查/安全 |

### 环境配置

触发：`config.*.yaml`/`config.*.yml`/`.env.*` 文件变更

| Skill | 路径 | 重点章节 |
|-------|------|---------|
| env-strategy | `~/.claude/skills/env-strategy/SKILL.md` | 四环境/配置分层/数据隔离 |

### 通用（所有 pack 自动附加）

| Skill | 路径 | 何时读取 |
|-------|------|---------|
| code-review | `~/.claude/skills/code-review/SKILL.md` | 审查时 |
| performance-checklist | `~/.claude/skills/performance-checklist/SKILL.md` | 性能检查时 |
| db-api-design | `~/.claude/skills/db-api-design/SKILL.md` | 涉及表设计/API 设计时 |
| error-handling | `~/.claude/skills/error-handling/SKILL.md` | 涉及错误体系设计时 |
| testing-strategy | `~/.claude/skills/testing-strategy/SKILL.md` | 涉及测试时 |
| ui-ue-guidelines | `~/.claude/skills/ui-ue-guidelines/SKILL.md` | 涉及 UI 交互设计时 |
| design-first | `~/.claude/skills/design-first/SKILL.md` | 新功能设计时 |
| frontend-ui-design | `~/.claude/skills/frontend-ui-design/SKILL.md` | 新前端页面设计时 |
| docker-deploy | `~/.claude/skills/docker-deploy/SKILL.md` | 涉及容器化部署时 |
| env-strategy | `~/.claude/skills/env-strategy/SKILL.md` | 涉及环境配置时 |
