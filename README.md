# Claude Strengthen Workflow

> 语言 / Language：**中文**｜[English](./README_EN.md)

面向全栈开发的 Claude Code 增强工作流，覆盖设计 → 编码 → 审查 → 调试完整生命周期。

通过 3 个 Subagents + 19 个 Skills + CLAUDE.md 规则，让 Claude 在写代码时就符合规范，而不是写完再靠审查补漏洞。

---

## 包含内容

### 3 个 Subagents

| Agent | 职责 | 触发时机 |
|-------|------|---------|
| `designer` | 技术方案 + UI/UX 设计文档 | 新功能/大改动，涉及 3+ 文件时自动触发 |
| `reviewer` | 代码审查并直接修复问题 | 写完代码后，或主动说"检查一下" |
| `debugger` | 定位根因并最小化修复 | 构建失败、测试报错、API 异常等 |

**核心设计**：发现问题直接修复，不只是报告问题。

---

### 19 个 Skills（按技术栈分组）

**前端**
| Skill | 覆盖内容 |
|-------|---------|
| `vue-conventions` | Vue3 组合式 API、Element Plus、常见坑 |
| `react-conventions` | Hooks、组件模式、状态管理 |
| `frontend-conventions` | 组件设计、四态处理、i18n、样式隔离 |
| `ui-ue-guidelines` | 布局、表单、弹窗、用户反馈、无障碍 |
| `frontend-ui-design` | 新页面开发前生成 ASCII 线框图 + 组件规格 + 交互说明 + i18n key 清单 |
| `mobile-cross-platform` | Flutter/RN/小程序架构、导航、平台适配 |

**后端**
| Skill | 覆盖内容 |
|-------|---------|
| `go-conventions` | 命名、错误处理、并发、常见坑 |
| `java-conventions` | Spring、异常、事务、命名规范 |
| `python-conventions` | 类型提示、异步、项目结构 |
| `rust-conventions` | 所有权、错误处理、并发 |
| `backend-conventions` | 分层架构、时间处理、DTO、日志安全 |

**通用**
| Skill | 覆盖内容 |
|-------|---------|
| `code-review` | 质量/安全/性能审查清单 |
| `testing-strategy` | 单元/集成/E2E 分层、覆盖率、mock |
| `performance-checklist` | 慢查询、N+1、首屏、缓存策略 |
| `db-api-design` | 表设计、RESTful 接口规范 |
| `error-handling` | 统一错误码、前后端错误传递链路 |
| `design-first` | 新功能先写设计文档再实现的工作流 |
| `docker-deploy` | Dockerfile、镜像构建、健康检查 |
| `env-strategy` | dev/test/pre/prod 四环境配置分层 |

---

### CLAUDE.md 新增 3 条规则

**1. 编码规范自动加载**

写代码前自动按文件类型加载对应 skill，同会话不重复加载：
- `.go` → go-conventions + backend-conventions
- `.vue` → vue-conventions + frontend-conventions + ui-ue-guidelines
- `.tsx/.jsx` → react-conventions + frontend-conventions + ui-ue-guidelines
- `.java / .py / .rs` → 对应语言规范 + backend-conventions
- 涉及表设计/API/测试/Docker → 按需追加对应 skill

**2. 变更影响范围**

每次改动前自动评估跨端影响：
- 改后端接口 → 检查前端调用方是否需要联动
- 改数据库字段 → 检查 ORM、API 响应、前端绑定
- 改公共模块 → grep 所有调用方再动手

**3. 任务并行执行**

多个独立任务时自动判断并行机会：
- 列出每个任务涉及的文件清单
- 无重叠 → 同时启动多个 agent 并行执行
- 有共享文件 → 降级串行，共享文件任务最后执行

---

## 完整工作流

```
你说需求（涉及 3+ 文件）
  └→ designer agent 先出设计文档
       ├─ 技术方案（数据模型 + 接口设计）
       └─ UI/UX 四态（加载/空/成功/失败）

你开始写代码
  └→ 自动加载对应语言规范（写时就符合规范，不靠事后审查）
  └→ 自动检查跨端影响范围
  └→ 多个独立任务自动并行执行

出错 / 构建失败 / 测试报错
  └→ debugger agent 定位根因 + 最小化修复 + 预防建议

写完后
  └→ reviewer agent 按技术栈加载规范，审查并直接修复
       ├─ Critical/Warning 问题直接改
       └─ Suggestion 只报告不强改
```

---

## 并行执行说明

| 场景 | 执行方式 |
|------|---------|
| 后端模块 A + 后端模块 B（文件不重叠） | 并行 ✅ |
| 前端页面 A + 前端页面 B（文件不重叠） | 并行 ✅ |
| 后端接口 + 前端页面（接口约定确定后） | 并行 ✅ |
| 多任务共享同一文件（如 gateway.go） | 串行 ⚠️ |
| 设计文档未完成就开始编码 | 串行 ⚠️ |

实测：后端 `tag.go`（22s）+ 前端 `Tags.vue`（67s）同时启动，总耗时 67s，比串行（89s）节省约 25%。

---

## 安装

**macOS / Linux：**
```bash
git clone git@github.com:martin98ksJ/claude-strengthen-workflow.git
cd claude-strengthen-workflow
bash install.sh
```

**Windows (PowerShell)：**
```powershell
git clone git@github.com:martin98ksJ/claude-strengthen-workflow.git
cd claude-strengthen-workflow
.\install.ps1
```

重新打开 Claude Code 即可生效。

> 安装脚本会先自动备份已有的 `~/.claude/` 文件（agents/、skills/、CLAUDE.md）到 `~/.claude/.backup/<时间戳>/`，然后覆盖安装。agents 和 skills 同名文件会提示是否覆盖。

---

## 验证安装效果

| 测试方式 | 预期行为 |
|---------|---------|
| 说"帮我写一个 Go 接口" | 写代码前自动读取 go-conventions + backend-conventions |
| 说"帮我加一个涉及多文件的新功能" | 自动触发 designer agent 先出设计文档 |
| 说"帮我检查一下改动" | 自动触发 reviewer agent 审查并修复 |
| 构建/测试失败 | 自动触发 debugger agent 定位根因修复 |
| 同时提两个独立任务 | 自动判断文件重叠，无重叠则并行执行 |

---

## 卸载

**macOS / Linux：**
```bash
bash uninstall.sh
```

**Windows (PowerShell)：**
```powershell
.\uninstall.ps1
```
