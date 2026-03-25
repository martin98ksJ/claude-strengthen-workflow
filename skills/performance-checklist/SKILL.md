---
name: performance-checklist
description: 前后端性能检查清单（慢查询、N+1、包体积、首屏加载、缓存策略）
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 性能问题描述或优化目标
- 涉及的代码文件/模块/接口
- 性能指标数据（如有：响应时间、QPS、包体积、首屏时间）

## 职责（必须做）

### 后端 — 数据库

- [ ] 慢查询识别：开启慢查询日志，关注 > 200ms 的查询；用 EXPLAIN 分析执行计划
- [ ] N+1 问题：循环内查询改为批量查询（IN 子句/JOIN），ORM 预加载关联数据
- [ ] 索引有效性：WHERE/JOIN/ORDER BY 字段有索引，复合索引遵循最左前缀，避免索引失效（函数包裹、隐式转换、LIKE '%xx'）
- [ ] 查询优化：只 SELECT 需要的列（不 SELECT *），大结果集分页/流式处理，避免全表扫描
- [ ] 连接池配置：最大连接数匹配并发量，空闲连接有回收策略，连接超时有合理值

### 后端 — 接口与计算

- [ ] 接口响应时间：P95 < 500ms（普通接口），P95 < 2s（复杂聚合），超标需优化或异步化
- [ ] 避免 O(n²)：嵌套循环改为 Map 查找，大集合操作注意算法复杂度
- [ ] 序列化开销：大对象避免深拷贝，JSON 序列化考虑字段裁剪，热路径避免反射
- [ ] 并发控制：耗时操作异步化（goroutine/线程池），并行调用无依赖的外部服务
- [ ] 资源释放：数据库连接、HTTP 客户端、文件句柄及时关闭，用 defer/try-finally 保证

### 后端 — 缓存策略

- [ ] 缓存分层：热数据内存缓存（LRU/本地 Map）→ 温数据分布式缓存（Redis/BadgerDB）→ 冷数据数据库
- [ ] 缓存键设计：`模块:资源:ID` 格式，包含版本号便于失效（如 `model:config:v2:123`）
- [ ] 失效策略：TTL 兜底 + 主动失效（写操作后删缓存），避免缓存与数据库不一致
- [ ] 防穿透：空值缓存（短 TTL）或布隆过滤器；防雪崩：TTL 加随机偏移，热 key 不同时过期
- [ ] 缓存预热：服务启动时加载高频数据，避免冷启动大量穿透

### 前端 — 加载性能

- [ ] 首屏指标：LCP < 2.5s、FID < 100ms、CLS < 0.1（Core Web Vitals 基线）
- [ ] 代码分割：路由级懒加载，重型库按需加载（`import()`），vendor 独立 chunk
- [ ] 包体积控制：定期分析（`vite-bundle-visualizer`/`webpack-bundle-analyzer`），单 chunk < 250KB（gzip 后）
- [ ] 资源优化：图片压缩 + WebP/AVIF + 懒加载，字体 subset + `font-display: swap`
- [ ] 预加载关键资源：`<link rel="preload">` 关键 CSS/字体，`<link rel="prefetch">` 下一页资源

### 前端 — 运行时性能

- [ ] 避免不必要的重渲染：React 用 memo/useMemo 有依据地优化，Vue 用 computed 代替 watch 同步
- [ ] 长列表虚拟滚动：> 100 条数据的列表用虚拟滚动（vue-virtual-scroller / react-window）
- [ ] 防抖节流：搜索输入 debounce（300ms），滚动/resize 事件 throttle
- [ ] 内存泄漏：组件卸载时清理定时器、事件监听、WebSocket 连接、AbortController
- [ ] Web Worker：CPU 密集计算（大数据处理/加密）移到 Worker，不阻塞主线程

### 前端 — 网络优化

- [ ] HTTP 缓存：静态资源 `Cache-Control: max-age=31536000` + 文件名 hash，API 按需 `ETag/Last-Modified`
- [ ] 请求合并：同页面多个接口考虑合并为一个聚合接口，减少请求数
- [ ] 压缩：服务端开启 gzip/brotli，API 响应大于 1KB 时压缩
- [ ] CDN：静态资源走 CDN，配置合理的缓存策略和回源规则

## 边界（不做什么）

- 不涉及具体性能测试工具使用（JMeter/k6/Lighthouse → 按需选型）
- 不涉及数据库调优细节（分库分表、读写分离、执行计划深度分析）
- 不涉及基础设施层优化（内核参数、网络调优、CDN 配置）
- 不涉及具体框架的性能 API（→ 见对应框架 skill）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
