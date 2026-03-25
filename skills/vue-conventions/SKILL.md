---
name: vue-conventions
description: Vue3 组合式 API、组件设计、状态管理和 Element Plus 使用规范
version: "1.1"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用前端规范见 frontend-conventions

## 输入

- Vue3 SFC 文件（.vue）或 composable 文件
- 涉及的页面或组件描述

## 职责（必须做）

### 组合式 API

- [ ] 使用 `<script setup lang="ts">`，复用逻辑抽 composables（`use` 前缀）
- [ ] `ref` 用于基本类型和需要替换整体的引用，`reactive` 用于不会整体替换的对象
- [ ] `computed` 处理派生状态，避免在 watch 里手动同步派生值
- [ ] `watch/watchEffect` 必须明确清理副作用（返回清理函数），组件卸载时自动停止

### 组件规范

- [ ] SFC 单文件，`<script setup>` → `<template>` → `<style scoped>` 顺序
- [ ] props 用 `defineProps<T>()` + TS 类型，设默认值用 `withDefaults`
- [ ] 事件用 `defineEmits<T>()` 声明，v-model 用 `defineModel()`
- [ ] `defineExpose` 只暴露必要方法，不暴露内部状态
- [ ] 组件通信：父子 props/emit，跨层 provide/inject，全局状态用 Pinia

### API 调用

- [ ] 统一在 `api/` 目录封装请求函数，一个模块一个文件（如 `api/user.ts`）
- [ ] 组件不直接写请求，通过 store action 或 composable 调用 api 层
- [ ] 请求/响应类型定义在 `types/` 目录，按模块分文件
- [ ] 错误统一在请求拦截器处理（toast 提示），业务层只处理特殊错误

### Element Plus

- [ ] 表单：`el-form` + `rules` 校验，校验规则可复用的抽到公共 validators
- [ ] 表格：`el-table` + `el-pagination` 分页，分页参数统一命名（`page/pageSize`）
- [ ] 弹窗：`el-dialog` 用 `v-model` 控制显隐，弹窗内表单提交后关闭并刷新列表
- [ ] 消息：操作反馈用 `ElMessage`，确认操作用 `ElMessageBox.confirm`
- [ ] 按需导入组件和样式，不全量引入

### 路由与状态

- [ ] 路由组件懒加载 `() => import()`，嵌套路由不超过 3 层
- [ ] 路由守卫处理鉴权和权限，未登录重定向到登录页
- [ ] Pinia store 按功能模块拆分，store 之间不互相引用，异步操作放 action
- [ ] store 只存需要跨组件共享的状态，组件局部状态用 `ref/reactive`

### i18n

- [ ] 用户可见文案用 `useI18n()` 的 `t('key')` 或模板中 `$t('key')`
- [ ] key 按 `模块.页面.字段` 层级命名（如 `user.list.deleteConfirm`）
- [ ] 新增文案同步更新所有语言文件，不留缺失 key

### TypeScript

- [ ] 组件 props/emit/expose 必须有类型定义
- [ ] API 响应定义 interface，不用 `any`；不确定类型用 `unknown` 后收窄
- [ ] 枚举值用 `const enum` 或字面量联合类型，不用魔法字符串

### 常见坑（AI 生成代码高频踩雷）

- [ ] reactive 解构丢失响应性：`const { name } = reactive(obj)` 解构后 name 是普通值，不再响应；用 `toRefs(obj)` 解构或直接 `obj.name` 访问
- [ ] ref 忘记 .value：`<script setup>` 中 `ref` 变量必须 `.value` 访问，template 中自动解包不需要；AI 经常在 JS 逻辑里漏掉 `.value`
- [ ] watch 立即执行陷阱：`watch(source, cb)` 默认不立即执行，初始值不触发；需要初始执行加 `{ immediate: true }`
- [ ] v-for 与 v-if 优先级：Vue 3 中 `v-if` 优先于 `v-for`（与 Vue 2 相反），同一元素上同时用会导致 `v-if` 访问不到 `v-for` 的变量；用 `<template v-for>` 包裹
- [ ] computed 有副作用：`computed` 里发请求/修改状态，导致无限循环或不可预测行为；computed 必须是纯函数，副作用放 `watch`
- [ ] 组件 v-model 双向绑定：自定义组件 `v-model` 需要 `defineModel()` 或 `modelValue` prop + `update:modelValue` emit，AI 经常只写一半
- [ ] nextTick 时机：DOM 更新是异步的，修改数据后立即读 DOM 拿到旧值；需要等 DOM 更新用 `await nextTick()`
- [ ] provide/inject 响应性：`provide(key, value)` 如果 value 不是 ref/reactive，inject 方拿到的是静态值不响应；provide ref 对象保持响应性
- [ ] 路由组件缓存：`<KeepAlive>` 缓存的组件不触发 `onMounted`，用 `onActivated`/`onDeactivated` 处理进入/离开逻辑
- [ ] Pinia store 解构：`const { count } = useStore()` 丢失响应性；用 `storeToRefs(store)` 解构响应式属性，方法直接解构

## 边界（不做什么）

- 不涉及通用前端规范（组件原则、四态、样式隔离 → 见 frontend-conventions）
- 不涉及 UI/UE 设计规范（布局、交互、反馈 → 见 ui-ue-guidelines）
- 不涉及 React 相关
- 不涉及 Vue2 Options API
- 不涉及构建工具配置（Vite/Webpack → 按项目约定）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
