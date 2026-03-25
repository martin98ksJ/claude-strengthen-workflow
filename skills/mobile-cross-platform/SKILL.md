---
name: mobile-cross-platform
description: Flutter/React Native/小程序等跨端开发的架构、导航、状态管理和平台适配规范
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 目标平台（iOS/Android/Web/小程序/鸿蒙）
- 技术栈选型（Flutter/React Native/Taro/uni-app/原生）
- 涉及的页面或功能模块描述

## 职责（必须做）

### 技术选型决策

- [ ] 选型依据：团队技术栈 > 性能要求 > 平台覆盖范围 > 生态成熟度
- [ ] Flutter：高性能自绘、双端一致性强、适合中大型 App
- [ ] React Native：Web 团队友好、热更新能力、适合快速迭代
- [ ] Taro/uni-app：小程序为主 + H5 兼顾、适合轻量级多端覆盖
- [ ] 原生（Swift/Kotlin）：极致性能和平台特性需求、核心模块可混合开发
- [ ] 不盲目跨端：性能敏感模块（音视频/地图/复杂动画）评估是否需要原生桥接

### 项目结构

- [ ] 分层清晰：UI 层（页面/组件）→ 业务层（状态/逻辑）→ 数据层（API/本地存储），跨层通过接口通信
- [ ] 按功能模块组织（`feature/user/`、`feature/order/`），不按技术层平铺（不建议顶层 `components/`、`services/`）
- [ ] 平台差异代码隔离：`.ios.tsx`/`.android.tsx`（RN）、`Platform.isIOS`（Flutter）、条件编译（小程序），不在业务代码里散落 if-else 判断平台
- [ ] 共享代码最大化：业务逻辑、数据模型、工具函数、API 层跨平台复用，只有 UI 和平台 API 调用允许差异

### 导航与路由

- [ ] 统一路由管理：集中定义路由表，支持深链接（Deep Link）和路由参数传递
- [ ] 导航模式：Tab 导航（主入口 3-5 个）+ Stack 导航（页面栈），避免导航层级过深（> 5 层提供快捷返回）
- [ ] 路由守卫：未登录拦截跳登录页，无权限页面给提示，路由切换时取消未完成的请求
- [ ] 页面生命周期：进入时加载数据、离开时清理资源（定时器/监听器/控制器）、返回时按需刷新

### 状态管理

- [ ] 局部状态：组件内 `useState`/`StatefulWidget`/`ref`，不上提到全局
- [ ] 共享状态：跨页面数据用状态管理方案（Provider/Riverpod/GetX/Zustand/Pinia），按模块拆分 store
- [ ] 服务端状态：API 数据用请求缓存方案（React Query/SWR/Dio 缓存），与 UI 状态分离
- [ ] 持久化状态：用户偏好/token/草稿用本地存储（SharedPreferences/AsyncStorage/MMKV），敏感数据加密存储
- [ ] 状态不冗余：同一数据单一数据源，派生数据用 computed/selector 计算，不手动同步

### 网络与数据

- [ ] 统一请求层：封装 HTTP 客户端（Dio/Axios/fetch），统一处理 baseURL、超时、重试、token 注入
- [ ] 请求拦截：自动附加 token、请求/响应日志（开发环境）、401 自动刷新 token 或跳登录
- [ ] 离线策略：关键数据本地缓存，断网时展示缓存数据 + 离线提示，恢复后自动同步
- [ ] 文件上传/下载：大文件分片上传、断点续传、下载进度展示、后台下载（如需）

### 平台适配

- [ ] 屏幕适配：使用相对单位（dp/rpx/rem），适配刘海屏/折叠屏/平板，安全区域（SafeArea）处理
- [ ] 手势与交互：遵循平台惯例（iOS 右滑返回、Android 物理返回键），触摸目标 ≥ 44pt/48dp
- [ ] 权限管理：运行时权限按需申请（相机/定位/存储），被拒后引导到设置页，不阻塞非相关功能
- [ ] 推送通知：统一推送通道（FCM/APNs/厂商通道），前台/后台/杀死状态分别处理，通知点击跳转到对应页面
- [ ] 系统特性：深色模式适配、多语言（i18n）、动态字体大小、无障碍标签（accessibilityLabel）

### 性能与体验

- [ ] 启动优化：减少启动时同步操作，闪屏页过渡，首页数据预加载
- [ ] 列表性能：长列表用 `ListView.builder`（Flutter）/ `FlatList`（RN）/ 虚拟列表，复用 item 组件
- [ ] 图片优化：按需加载合适尺寸、内存缓存 + 磁盘缓存、占位图/渐进加载
- [ ] 包体积：按需引入依赖，移除未使用资源，开启代码混淆和 Tree Shaking
- [ ] 内存管理：页面销毁时释放大对象（图片/视频/WebView），监控内存泄漏

### 发布与更新

- [ ] 版本号规范：`major.minor.patch`（语义化），内部构建号递增
- [ ] 热更新（如支持）：RN CodePush / Flutter Shorebird / 小程序自带，灰度发布 + 强制更新策略
- [ ] 应用商店：遵循 App Store / Google Play 审核指南，隐私政策、权限说明、截图准备
- [ ] 崩溃监控：集成 Crashlytics/Sentry/Bugly，崩溃率 < 0.1%，ANR 率 < 0.5%

## 边界（不做什么）

- 不涉及原生 iOS/Android 开发细节（Swift/Kotlin 语法、原生 SDK 用法）
- 不涉及具体框架 API 教程（Flutter Widget/RN Component → 查官方文档）
- 不涉及 UI 视觉设计（配色/字体/图标 → 按设计系统执行，交互规范见 ui-ue-guidelines）
- 不涉及后端 API 设计（→ 见 db-api-design + backend-conventions）
- 不涉及 CI/CD 和自动化构建流水线配置

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
