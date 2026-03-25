---
name: react-conventions
description: React Hooks、组件模式、状态管理和数据获取规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用前端规范见 frontend-conventions

## 输入

- React 组件文件（.tsx/.jsx）或 hook 文件
- 涉及的页面或组件描述

## 职责（必须做）

### 组件

- [ ] 函数组件为主，组件文件名大驼峰（`UserProfile.tsx`），一个文件一个导出组件
- [ ] props 用 TypeScript interface 定义，解构时设默认值；children 类型用 `React.ReactNode`
- [ ] 条件渲染用早返回（`if (!data) return <Empty />`），不在 JSX 里嵌套三元
- [ ] 组件职责单一，UI 组件（纯展示）和容器组件（数据获取+逻辑）分离

### Hooks

- [ ] 复用逻辑抽自定义 hook（`use` 前缀），hook 内部不处理 UI
- [ ] `useEffect` 依赖数组严格声明（开 exhaustive-deps），cleanup 函数必须处理（取消请求/清除定时器/解绑事件）
- [ ] 不在条件/循环中调用 hook，保持调用顺序稳定
- [ ] 自定义 hook 返回值：单值直接返回，多值用对象（非数组，便于按名解构）

### 状态管理

- [ ] `useState` 管局部状态，跨组件共享用 zustand/jotai，避免 Context 滥用（超过 3 层消费者换方案）
- [ ] 状态就近原则：状态放在最近的共同父组件，不无脑提升到顶层
- [ ] 派生状态直接计算（`const fullName = first + last`），不用 `useEffect` + `setState` 同步
- [ ] 复杂状态逻辑用 `useReducer`，action type 用字符串字面量联合类型

### 数据获取

- [ ] 统一在 `api/` 目录封装请求函数，组件不直接写 fetch/axios
- [ ] 服务端状态用 TanStack Query（React Query）/ SWR 管理（缓存、重试、失效刷新）
- [ ] loading/error/empty 三态必须处理，请求取消用 AbortController
- [ ] API 响应类型定义在 `types/` 目录，按模块分文件

### 性能

- [ ] `React.memo` 只在确认有性能问题时用，不预防式 memo
- [ ] `useMemo/useCallback` 需有明确依据（大计算 / 传给 memo 子组件的引用），不滥用
- [ ] 列表渲染 key 用稳定唯一值（id），不用 index（除非列表不会重排）
- [ ] 代码分割：路由级用 `React.lazy` + `Suspense`，重型组件按需加载

### 样式

- [ ] CSS Modules 或 Tailwind，避免 inline style（除动态计算值）
- [ ] className 组合用 `clsx/cn`，条件样式用对象语法 `cn({ active: isActive })`
- [ ] 响应式用 CSS 媒体查询/container query，不在 JS 里监听 resize

### TypeScript

- [ ] 组件 props 定义 interface（`interface Props {}`），不用 `type` 定义 props（interface 可扩展）
- [ ] 事件处理器类型用 React 提供的（`React.ChangeEvent<HTMLInputElement>`）
- [ ] 禁止 `any`，不确定类型用 `unknown` 后收窄；泛型组件用 `<T,>` 语法

### 常见坑（AI 生成代码高频踩雷）

- [ ] useEffect 无限循环：依赖数组里放对象/数组（每次渲染新引用），导致 effect 无限触发；用 `useMemo` 稳定引用或拆成基本类型依赖
- [ ] 闭包陷阱（stale closure）：`useEffect`/`useCallback` 里引用的 state 是创建时的快照，不是最新值；加入依赖数组或用 `useRef` 存最新值
- [ ] setState 异步批处理：`setCount(count + 1); setCount(count + 1)` 结果只 +1；连续更新用函数式 `setCount(prev => prev + 1)`
- [ ] key 导致状态丢失：列表 key 用 index，增删元素后组件状态错乱（输入框内容串行）；用稳定唯一 id 作 key
- [ ] useEffect 缺少 cleanup：订阅/定时器/事件监听不清理，组件卸载后仍执行导致内存泄漏或 setState on unmounted 警告
- [ ] 条件 Hook 调用：`if (cond) { useState() }` 违反 Hook 规则，渲染间调用顺序不一致导致状态错乱；Hook 必须在顶层无条件调用
- [ ] 对象/数组 state 直接修改：`state.items.push(x); setState(state)` 引用没变不触发重渲染；用展开 `setState({...state, items: [...state.items, x]})`
- [ ] memo 失效：`React.memo(Comp)` 但父组件每次传新的内联函数/对象 prop，memo 形同虚设；用 `useCallback`/`useMemo` 稳定 prop 引用
- [ ] Context 性能陷阱：Context value 变化导致所有消费者重渲染，即使消费者只用了 value 的一部分；拆分 Context 或用 zustand/jotai 替代
- [ ] dangerouslySetInnerHTML XSS：AI 生成代码渲染富文本时直接用 `dangerouslySetInnerHTML`，不做 sanitize；必须用 DOMPurify 等库清洗

## 边界（不做什么）

- 不涉及通用前端规范（组件原则、四态、i18n → 见 frontend-conventions）
- 不涉及 UI/UE 设计规范（布局、交互、反馈 → 见 ui-ue-guidelines）
- 不涉及 Vue 相关
- 不涉及 Class Component（已废弃模式）
- 不涉及 Next.js/Remix 等框架特定用法（Server Component/RSC → 按项目约定）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
