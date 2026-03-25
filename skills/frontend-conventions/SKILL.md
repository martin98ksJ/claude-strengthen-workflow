---
name: frontend-conventions
description: 前端开发时的组件设计、状态管理和交互规范
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 前端代码文件（Vue/React/TS 等）
- 涉及的页面或组件描述

## 职责（必须做）

- [ ] 组件单一职责：一个组件只做一件事，超过 300 行考虑拆分
- [ ] Props/Emit 规范：props 必须定义类型，事件用 emit 声明，禁止直接改 props
- [ ] 四态处理：每个数据展示区域必须处理 加载态/空态/错误态/成功态
- [ ] 样式隔离：使用 scoped 或 CSS Modules，避免全局污染，禁止 !important
- [ ] i18n：用户可见文案走 locale 文件，不硬编码中文/英文字符串
- [ ] 性能基线：路由懒加载、图片懒加载、长列表虚拟滚动、避免不必要的重渲染

### 常见坑（AI 生成代码高频踩雷）

- [ ] 异步竞态：快速切换页面/Tab，旧请求晚于新请求返回，覆盖了正确数据；用 AbortController 取消旧请求或用请求 ID 校验
- [ ] 内存泄漏三件套：`setInterval` 未清除、`addEventListener` 未移除、WebSocket 未关闭；组件卸载时必须清理
- [ ] 深拷贝 JSON.parse(JSON.stringify)：丢失 Date/undefined/函数/循环引用；用 `structuredClone()`（现代浏览器）或 lodash `cloneDeep`
- [ ] 浮点数显示：`0.1 + 0.2` 显示 `0.30000000000000004`；展示用 `toFixed(2)`，计算用整数（分为单位）或 Decimal 库
- [ ] 时区问题：`new Date('2025-01-15')` 在不同时区解析结果不同（UTC vs 本地）；明确用 `new Date('2025-01-15T00:00:00')` 或 dayjs 处理
- [ ] CSS 样式穿透失败：scoped 样式无法影响子组件内部；Vue 用 `:deep()`，React CSS Modules 用 `:global()`
- [ ] 图片/资源路径：动态路径 `require(variable)` 或 `import(variable)` 打包后找不到；用 `new URL(path, import.meta.url)` 或静态 import
- [ ] localStorage 容量和类型：存储上限约 5MB，只能存字符串；存对象需 `JSON.stringify`，取出需 `JSON.parse`，注意 parse 失败兜底
- [ ] 事件冒泡与默认行为：表单内按钮默认 `type="submit"` 触发表单提交刷新页面；明确 `type="button"` 或 `e.preventDefault()`
- [ ] TypeScript any 传染：一个 `any` 类型会沿调用链传染，下游全部丢失类型检查；用 `unknown` + 类型收窄替代

## 边界（不做什么）

- 不涉及具体框架 API 细节（见 vue/react-conventions）
- 不涉及后端逻辑和 API 设计
- 不涉及 UI 视觉设计细节（配色、字体、间距等按仓库设计系统执行）
- 涉及布局、交互、可用性时联用 ui-ue-guidelines
- 不涉及构建工具配置（webpack/vite 等）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
