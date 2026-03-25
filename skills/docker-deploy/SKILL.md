---
name: docker-deploy
description: Dockerfile 规范、镜像构建、docker-compose 编排、健康检查
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 项目的技术栈和构建方式
- 现有 Dockerfile / docker-compose 文件（如有）
- 部署目标环境描述

## 职责（必须做）

### Dockerfile 规范

- [ ] 多阶段构建：build 阶段用完整 SDK 镜像，runtime 阶段用最小基础镜像（alpine/distroless/scratch）
- [ ] 层缓存优化：依赖安装（go.mod/package.json）在代码 COPY 之前，变更频率低的层在上
- [ ] 不在镜像中包含：源码（runtime 阶段）、.git、测试文件、开发依赖、密钥/配置文件
- [ ] 使用 .dockerignore 排除无关文件，减小构建上下文
- [ ] 非 root 用户运行：`RUN adduser` + `USER appuser`，应用监听非特权端口
- [ ] 固定基础镜像版本（`golang:1.24-alpine`，不用 `latest`），定期更新基础镜像修复 CVE

### 镜像构建

- [ ] 镜像标签策略：`<registry>/<project>:<git-sha-short>`（可追溯），同时打 `latest` 和语义版本 tag
- [ ] 构建参数通过 `ARG` 传入（版本号、构建时间），运行时配置通过 `ENV` 或挂载
- [ ] 镜像体积控制：Go 项目 < 30MB（scratch/静态编译），Node 项目 < 200MB（alpine + 生产依赖）
- [ ] CI 中构建：利用 BuildKit 缓存（`--mount=type=cache`），多平台构建用 `buildx`

### docker-compose 编排

- [ ] 服务命名清晰（`gateway`、`db`、`redis`），不用 `app1`/`service1`
- [ ] 环境变量：compose 文件只放非敏感默认值，敏感信息通过 `.env` 文件或 secrets 注入
- [ ] 数据持久化：数据库/存储目录用 named volume 或 bind mount，明确挂载路径
- [ ] 网络隔离：前端/后端/数据库分网络，只暴露必要端口
- [ ] 依赖顺序：`depends_on` + 健康检查条件（`condition: service_healthy`），不依赖启动顺序
- [ ] 资源限制：设置 `mem_limit`/`cpus`，防止单容器耗尽宿主资源

### 健康检查

- [ ] HEALTHCHECK 指令：HTTP 服务用 `curl/wget` 探测健康端点，非 HTTP 用进程/端口检查
- [ ] 健康端点（`/health` 或 `/healthz`）返回：服务状态 + 关键依赖状态（DB/缓存连通性）
- [ ] 参数合理：`interval=30s`、`timeout=5s`、`retries=3`、`start_period=10s`（按服务启动时间调整）
- [ ] 编排层（compose/k8s）根据健康状态决定流量路由和重启策略

### 运维与安全

- [ ] 日志输出到 stdout/stderr（不写容器内文件），由 Docker/编排层收集
- [ ] 优雅停机：应用处理 SIGTERM，完成在途请求后退出；`stop_grace_period` 设合理值
- [ ] 重启策略：`restart: unless-stopped`（开发）或 `restart: always`（生产）
- [ ] 镜像扫描：CI 中集成漏洞扫描（Trivy/Snyk），高危漏洞阻断发布

## 边界（不做什么）

- 不涉及 Kubernetes/Helm/Terraform 等编排平台（容器化到 compose 级别）
- 不涉及 CI/CD pipeline 配置（Jenkins/GitHub Actions 等具体工具）
- 不涉及环境策略和发布流程（→ 见 env-strategy）
- 不涉及应用代码逻辑和架构设计

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
