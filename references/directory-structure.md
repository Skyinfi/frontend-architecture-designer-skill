# 目录结构参考

> 本文件供 `core/workflow.md` §5（目录结构方案）查表使用。
> 按复杂度 Level 和框架类型给出目录模板，含职责说明和反模式。

---

## 一、Level 1：扁平结构（小型项目）

适用于：个人工具、静态站、原型、页面数 < 10。

### React

```
project/
├── src/
│   ├── components/       # 通用 UI 组件
│   ├── pages/            # 页面组件（或用文件路由）
│   ├── hooks/            # 自定义 hooks（如有）
│   ├── utils/            # 工具函数
│   ├── styles/           # 全局样式 / CSS Variables
│   ├── App.tsx
│   └── main.tsx
├── public/
├── docs/
│   └── frontend-architecture.md
├── package.json
└── [配置文件]
```

### Vue

```
project/
├── src/
│   ├── components/       # 通用 UI 组件
│   ├── views/            # 页面组件
│   ├── composables/      # 组合式函数（如有）
│   ├── utils/            # 工具函数
│   ├── styles/           # 全局样式
│   ├── App.vue
│   └── main.ts
├── public/
├── docs/
├── package.json
└── [配置文件]
```

### Astro SSG

```
project/
├── scripts/              # 构建前脚本（sync、预处理）
├── src/
│   ├── content/          # Content Collections（MD/MDX 数据源）
│   │   └── config.ts     #   Collection schema
│   ├── data/             # 构建生成的 JSON 数据
│   ├── pages/            # 文件路由（.astro / .mdx）
│   ├── layouts/          # 页面布局模板
│   ├── components/       # 可复用 UI 组件
│   └── utils/            # 工具函数
├── public/               # 静态资源（不经过构建）
│   └── html-notes/       #   外部 HTML 文件（如有）
├── docs/
├── astro.config.mjs
└── package.json
```

### scripts/ 职责说明（构建管线项目）

当项目包含构建前处理（如内容同步、元数据提取）时：

| 目录/文件 | 职责 | 触发时机 |
|---|---|---|
| `scripts/sync-*.ts` | 从外部源同步内容到 `src/content/` 和 `public/` | `prebuild` 钩子 |
| `scripts/extract-*.ts` | 从原始文件提取元数据，生成 JSON | 被 sync 脚本调用 |
| `package.json#scripts.prebuild` | 自动触发 sync | `npm run build` 前 |

---

## 二、Level 2：Feature-based 结构（中型项目）

适用于：SaaS 后台、工具型产品、页面数 10–50、团队 2–5 人。

### React

```
project/
├── src/
│   ├── features/               # 按功能模块组织
│   │   ├── auth/
│   │   │   ├── components/     #   该功能的 UI 组件
│   │   │   ├── hooks/          #   该功能的 hooks
│   │   │   ├── api/            #   该功能的 API 调用
│   │   │   ├── types.ts        #   该功能的类型
│   │   │   └── index.ts        #   对外导出（公共 API）
│   │   ├── dashboard/
│   │   └── [feature]/
│   ├── shared/                 # 跨功能共享
│   │   ├── components/         #   通用 UI 组件
│   │   ├── hooks/              #   通用 hooks
│   │   ├── utils/              #   工具函数
│   │   └── types/              #   共享类型
│   ├── app/                    # 应用壳
│   │   ├── routes.tsx          #   路由定义
│   │   ├── providers.tsx       #   Context providers
│   │   └── layout.tsx          #   全局布局
│   ├── styles/                 # Design Token / 全局样式
│   └── main.tsx
├── public/
├── docs/
│   └── frontend-architecture.md
├── package.json
└── [配置文件]
```

### Vue

```
project/
├── src/
│   ├── features/               # 按功能模块组织
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── composables/
│   │   │   ├── api/
│   │   │   ├── types.ts
│   │   │   └── index.ts
│   │   └── [feature]/
│   ├── shared/                 # 跨功能共享
│   │   ├── components/
│   │   ├── composables/
│   │   ├── utils/
│   │   └── types/
│   ├── app/                    # 应用壳
│   │   ├── router.ts
│   │   ├── providers.vue
│   │   └── layout.vue
│   ├── styles/
│   └── main.ts
├── public/
├── docs/
├── package.json
└── [配置文件]
```

---

## 三、Level 3：领域化结构（大型项目）

适用于：复杂业务系统、多团队协作、页面数 > 50、需要 DDD。

```
project/
├── src/
│   ├── domains/                    # 领域模块
│   │   ├── core/                   # 核心域（业务主流程）
│   │   │   └── [domain]/
│   │   │       ├── components/
│   │   │       ├── hooks/          # React / composables (Vue)
│   │   │       ├── api/
│   │   │       ├── types/
│   │   │       ├── constants.ts
│   │   │       └── index.ts
│   │   ├── support/                # 支撑域（辅助业务）
│   │   └── shared/                 # 通用域（跨域共享）
│   │       ├── auth/               # 认证授权
│   │       ├── theme/              # 主题系统
│   │       ├── i18n/               # 国际化
│   │       ├── request/            # 请求层封装
│   │       └── ui/                 # 通用 UI 组件
│   ├── app/                        # 应用壳
│   │   ├── routes.tsx
│   │   ├── providers.tsx
│   │   └── layout.tsx
│   ├── styles/                     # Design Token / 全局样式
│   └── main.tsx
├── public/
├── docs/
│   └── frontend-architecture.md
├── package.json
└── [配置文件]
```

---

## 四、目录职责对照表

### 通用职责

| 目录 | 职责 | 放什么 | 不放什么 |
|---|---|---|---|
| `src/components/` (L1) / `src/shared/components/` (L2+) | 通用 UI 组件 | Button、Modal、Toast 等纯 UI 组件 | 业务逻辑、API 调用 |
| `src/features/` (L2+) | 功能模块 | 按业务功能组织的组件、hooks、API、类型 | 跨功能的共享代码 |
| `src/app/` (L2+) | 应用壳 | 路由、全局 providers、全局布局 | 业务组件 |
| `src/styles/` | 全局样式 | CSS Variables、Design Token、全局 CSS | 组件 scoped 样式 |
| `src/utils/` (L1) | 工具函数 | 纯函数、格式化、常量 | 依赖业务上下文的逻辑 |
| `public/` | 静态资源 | 不经构建的文件（图片、favicon、raw HTML） | JS/CSS 源码 |
| `scripts/` | 构建脚本 | 同步、预处理、代码生成 | 运行时代码 |
| `docs/` | 文档 | 架构方案、API 文档 | 源码 |

### Astro 专用

| 目录 | 职责 |
|---|---|
| `src/content/` | Content Collections 数据源（.md/.mdx），由 schema 约束 |
| `src/data/` | 构建生成的 JSON 数据（如 html-notes.json） |
| `src/layouts/` | 页面布局模板（BaseLayout、NoteLayout 等） |
| `src/pages/` | 文件路由，每个文件对应一个 URL |
| `public/html-notes/` | 外部 HTML 原文件，不经过 Astro 管线 |

---

## 五、与已有目录结构对比

当存量项目已有目录结构时，参考本文件的价值是**校验和补充**，而非全量替换。

### 校验清单

| 检查项 | 健康信号 | 问题信号 |
|---|---|---|
| 页面/路由位置 | 统一在 `pages/` 或 `views/` | 页面散落在多个目录 |
| 组件分类 | 通用组件与业务组件分开 | 所有组件混在一个大 `components/` |
| 工具函数 | 纯函数无副作用 | 工具函数里包含 API 调用 |
| 样式管理 | 全局 Token + 组件 scoped | 全局 CSS 到处 import |
| 类型定义 | 集中管理或就近放置 | 类型散落在各处或大量 `any` |

### 补充建议

- 如果缺少 `docs/` 目录，建议新增并放入 `frontend-architecture.md`
- 如果缺少 `scripts/` 目录但构建前需要处理，建议新增
- 如果 `components/` 文件数 > 20 且无子分类，考虑按 feature 拆分（升级到 L2）

---

## 六、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| 业务组件和通用 UI 组件混放 | 职责不清，复用困难 | 分层：`shared/components/`（通用）+ `features/[x]/components/`（业务） |
| 一个 `components/` 目录放 50+ 文件 | 找不到东西，AI 搜索效率低 | 按功能或类型分子目录 |
| 在 `utils/` 中放 API 调用 | `utils` 应该是纯函数 | API 调用放 `api/` 或 `features/[x]/api/` |
| 页面组件直接包含业务逻辑 | 页面应该薄，只做路由和布局 | 业务逻辑下沉到 hooks/composables 或 service 层 |
| 按文件类型而非功能分目录（`all-components/`、`all-hooks/`） | 修改一个功能要跨多个目录 | 按功能模块组织（feature-based），同一功能的组件/hooks/API 放一起 |
| 给 Level 1 项目用领域化结构 | 过度设计，增加认知负担 | 小项目用扁平结构，够用就好 |
| 给 Level 3 项目用扁平结构 | 业务复杂度无法在扁平结构中管理 | 升级到 feature-based 或领域化 |
| Astro 项目把所有内容放在 `public/` | 绕过 Content Collection，失去 schema 校验和类型安全 | MD 内容放 `src/content/`，只有原始静态文件放 `public/` |

---

## 七、Level 与目录结构选择

| Level | 目录策略 | 判断依据 |
|---|---|---|
| **Level 1** | 扁平结构 | 页面 < 10，1 人维护，无复杂业务逻辑 |
| **Level 2** | Feature-based | 页面 10–50，2–5 人，有多条业务线 |
| **Level 3** | 领域化 | 页面 > 50，多团队，需要 DDD 方法论 |

**核心原则**：目录结构服务于**可维护性**，不服务于"看起来专业"。小项目用扁平结构是正确的工程决策，不是偷懒。
