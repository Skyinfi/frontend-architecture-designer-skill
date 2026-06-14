# 数据与状态架构参考

> 本文件供 `output-template.md` §9（数据与状态架构）查表使用。
> 仅在 **Level 2+** 输出时参考。Level 1 项目通常不需要全局状态管理。

---

## 一、状态三分类

前端状态应先分类，再选方案。**不同类型的状态用不同工具管理**。

| 类型 | 定义 | 特征 | 管理方案 |
|---|---|---|---|
| **服务端状态（server-state）** | 来自后端 API 的数据 | 异步获取、需要缓存、可能过期、多组件共享 | TanStack Query / SWR / RTK Query |
| **全局 UI 状态（client-state）** | 客户端的全局交互状态 | 同步、即时、纯前端 | Zustand / Jotai / Pinia |
| **组件局部状态** | 单个组件内部的 UI 状态 | 同步、瞬时、不需要共享 | React useState / Vue ref |

### 判断流程

```
这个状态来自 API 吗？
├── 是 → server-state → 数据获取层（TanStack Query 等）
└── 否 → 多个组件需要访问吗？
    ├── 是 → 全局 UI 状态 → 全局状态管理（Zustand / Pinia）
    └── 否 → 组件局部状态 → useState / ref
```

---

## 二、数据获取层选型

### React 生态

| 方案 | 适合场景 | 核心优势 | 注意事项 |
|---|---|---|---|
| **TanStack Query** | 大多数项目（首选） | 缓存/失效/重试/分页内置，社区最大 | 学习 queryKey 设计 |
| **SWR** | 简单数据获取 | 极简 API，Vercel 维护 | 功能不如 TanStack Query 丰富 |
| **RTK Query** | 已用 Redux Toolkit 的项目 | 与 RTK 深度集成 | 不用 Redux 就不要为了它引入 |

### Vue 生态

| 方案 | 适合场景 | 核心优势 |
|---|---|---|
| **Pinia + 组合式请求** | 大多数 Vue 项目 | Pinia 做全局状态，数据获取用组合式函数封装 |
| **TanStack Query (Vue)** | 复杂数据需求 | 与 React 版共享核心能力 |
| **VueUse** | 轻量工具函数 | `useFetch`、`useStorage` 等现成组合式函数 |

### Astro / SSG

| 方案 | 适合场景 |
|---|---|
| **Content Collections** | 本地 Markdown/MDX 数据 |
| **构建时 fetch** | 构建时从 API 拉取数据 |
| **无运行时数据层** | 纯静态站不需要 |

### 请求层封装原则

无论选哪个方案，都应建立统一的请求层：

```
src/
├── shared/
│   └── request/
│       ├── client.ts          # HTTP 客户端实例（axios/fetch 封装）
│       ├── interceptors.ts    # 请求/响应拦截器（token、错误处理）
│       └── types.ts           # 通用响应类型
```

**封装要求**：
- 统一错误处理（网络错误、业务错误、权限错误）
- 统一 loading / error 状态
- 请求取消（组件卸载时自动取消）
- 重试策略（可配置）

---

## 三、全局状态选型边界

### React 生态

| 方案 | 适合场景 | 核心优势 | 何时不用 |
|---|---|---|---|
| **Zustand** | 大多数项目（首选） | 极简、无 boilerplate、TS 友好 | 需要极细粒度响应式更新 |
| **Jotai** | 原子化状态管理 | 按需渲染、组合灵活 | 状态间有复杂依赖关系 |
| **Redux Toolkit** | 大型团队、复杂业务 | DevTools 强大、中间件丰富 | 小项目嫌 boilerplate 多 |
| **Context + useReducer** | 极少量全局状态 | 零依赖 | 性能敏感场景（不必要的重渲染） |

### Vue 生态

| 方案 | 适合场景 | 核心优势 |
|---|---|---|
| **Pinia** | Vue 项目标准方案 | 官方推荐、TS 友好、DevTools 支持 |
| **VueUse** | 轻量组合式工具 | `useDark`、`useStorage` 等现成方案 |

### 何时不需要全局状态

| 场景 | 说明 |
|---|---|
| 纯静态站（Astro SSG） | 构建时数据全部内联，无运行时状态 |
| 页面数 < 5 | 组件间通信靠 props + events 足够 |
| 无跨页面共享数据 | 每个页面独立获取数据，不需要全局缓存 |
| 无主题切换 / 侧栏状态 | 缺少需要全局协调的 UI 状态 |

**如果不需要全局状态，在输出中写"未使用全局状态"，不要强行引入。**

---

## 四、全局状态边界判断

### 放全局

| 状态 | 理由 |
|---|---|
| 主题 / 暗色模式 | 所有页面都需要响应 |
| 侧边栏展开状态 | 多个布局组件需要读写 |
| 当前用户信息 | 多处使用（导航栏、权限判断、个人信息） |
| 全局通知 / Toast | 任何位置都可能触发 |
| 语言 / locale | 全站生效 |

### 不放全局

| 状态 | 正确做法 |
|---|---|
| 表单输入值 | 组件局部 useState / ref |
| 列表选中项 | 组件局部状态 |
| 弹窗开关 | 组件局部状态（或受控 prop） |
| 分页当前页 | 组件局部状态 + URL 参数 |
| 单次请求的 loading | 数据获取层自动管理 |

### 模糊地带

| 状态 | 判断方法 |
|---|---|
| 当前选中实体（如当前商品 ID） | 如果只在当前页面用 → URL 参数；如果跨页面用 → 全局 |
| 面包屑路径 | 如果从路由自动生成 → 不需要状态；如果需要手动配置 → 全局 |
| 搜索关键词 | 如果需要持久化到 URL → URL 参数；如果是临时搜索 → 局部状态 |

---

## 五、Register 差异化建议

### Content Register

| 应该 | 不应该 |
|---|---|
| 构建时获取所有数据 | 引入 TanStack Query / SWR |
| 零客户端状态（或极少） | 引入 Zustand / Redux |
| CSS Variables 管理主题状态 | 用 JS 管理主题切换 |

### Tool Register

| 应该 | 不应该 |
|---|---|
| TanStack Query 管理所有 API 数据 | 用 useEffect + useState 手动管理请求 |
| Zustand 管理全局 UI 状态 | 用 Context 传递所有状态 |
| 统一请求层封装 | 散落的 fetch / axios 调用 |
| Pinia（Vue）管理全局状态 | 在组件内直接读写 localStorage |

### Desktop Register

| 应该 | 不应该 |
|---|---|
| Zustand 管理面板/窗口状态 | Redux（过于重） |
| 本地数据优先，减少网络请求 | 每次操作都请求 API |
| 文件系统状态通过事件同步 | 轮询文件系统 |

### Hybrid Register

| 应该 | 不应该 |
|---|---|
| 内容页用 Content 的零状态策略 | 全站统一用重型状态管理 |
| 工具页用 Tool 的完整状态方案 | 内容页也加载 TanStack Query |

---

## 六、与既有数据架构对比

当存量项目已有数据/状态管理时，参考本文件的价值是**校验和补充**，而非全量替换。

### 校验清单

| 检查项 | 健康信号 | 问题信号 |
|---|---|---|
| 状态来源 | 明确区分 server-state / client-state / 局部 | 全部用 useState 模拟服务端数据 |
| 数据获取 | 统一数据获取层（TanStack Query 等） | 散落的 fetch / axios 调用 |
| 全局状态 | 仅放 UI 状态（主题/侧栏/通知等） | 把 API 数据放全局 store |
| 错误处理 | 统一拦截器 + 统一 loading / error 状态 | 每个组件各自 try-catch |
| 缓存策略 | queryKey 设计规范，失效策略统一 | 缓存键混乱，失效不一致 |
| 请求取消 | 组件卸载自动取消 | 切换页面仍发请求 |

### 补充建议

- 如果缺少统一请求层（`shared/request/`），建议新增并迁移散落的 fetch 调用
- 如果 Redux 项目复杂度不高，可考虑迁移到 Zustand 减少 boilerplate
- 如果用 useEffect + useState 模拟服务端数据，建议引入 TanStack Query 自动化 loading/error/缓存
- 如果没有数据失效策略，建议按业务定义 `staleTime` / `cacheTime` 等配置

---

## 七、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| 无脑上 Redux | 小项目用 Redux 增加大量 boilerplate | 小项目用 Zustand 或不需要全局状态 |
| 把 API 数据放全局状态 | 全局状态需要手动管理缓存/失效，数据获取层自动做 | API 数据用 TanStack Query / SWR |
| 散落的 fetch 调用 | 无统一错误处理、无缓存、无重试 | 统一请求层封装 |
| useEffect + useState 管理请求 | 手动处理 loading/error/race condition | 用数据获取层（TanStack Query 等） |
| 所有状态放一个 store | store 臃肿，难以维护 | 按功能拆分 store 或用原子化方案 |
| 组件间通过 localStorage 同步 | 性能差、时序不可靠、类型不安全 | 用全局状态管理 |
| 把 URL 可表达的状态放全局 | 刷新丢失、无法分享链接 | URL 参数（query / path） |
| 给静态站引入状态管理 | 纯静态内容无运行时状态需要管理 | 构建时处理一切 |

---

## 八、AI 友好命名约定

数据/状态相关命名直接影响 AI 搜索、理解和修改效率。
命名四原则统一遵循 `core/workflow.md` §7.5（可搜索、语义明确、遵循约定、模式稳定）。本节只补充数据与状态领域的命名模式。

### 命名规则

| 类型 | 命名模式 | 示例 |
|---|---|---|
| Store / State 文件 | camelCase，领域 + Store | `productStore.ts`、`authStore.ts` |
| Store 内部 slice | camelCase，领域 + Slice | `productSlice.ts` |
| Hook（数据获取） | camelCase，use + 领域 + Query | `useProductQuery.ts`、`useUserQuery.ts` |
| Hook（业务逻辑） | camelCase，use + 动词 | `useProductSearch.ts`、`useDebounce.ts` |
| Query Key | 数组路径，便于失效 | `['product', id]`、`['product', 'list', filter]` |
| Mutation Key | 同上，按操作命名 | `['product', 'create']` |
| API 模块 | camelCase，领域 + Api | `productApi.ts`、`orderApi.ts` |
| 类型文件 | camelCase，领域 + .types | `product.types.ts` |
| 请求层 | `shared/request/` 下分类 | `client.ts` / `interceptors.ts` / `types.ts` |

---

## 九、Level 与数据架构选择

| Level | 数据架构策略 | 输出内容 |
|---|---|---|
| **Level 1** | 不需要全局状态 | 标注"本项目复杂度较低，此项暂不展开"，说明无运行时数据需求 |
| **Level 2** | 基础数据架构 | 状态三分类表 + 数据获取层 + 全局状态边界 |
| **Level 3** | 完整数据架构 | 上述 + 请求层封装 + 缓存策略 + 错误处理统一方案 |
