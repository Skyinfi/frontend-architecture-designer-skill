# 组件架构参考

> 本文件供 `core/workflow.md` §7（核心组件清单）和输出模板 §7 查表使用。
> 定义组件分层规则、命名规范、封装策略，确保 Agent 输出的组件体系可被后续 AI coding 稳定消费。

---

## 一、组件三层分离

前端组件应严格分为三层，职责不重叠：

| 层级 | 职责 | 标识特征 | 示例 |
|---|---|---|---|
| **UI 组件** | 纯视觉呈现，零业务逻辑 | props 驱动，无 API 调用，无路由感知 | Button、Modal、DataTable、FormatBadge |
| **业务组件** | 承载业务逻辑，组合 UI 组件 | 包含 API 调用、状态管理、业务规则 | ProductForm、OrderList、NoteCard |
| **页面组件** | 路由入口，组装业务组件 | 对应一个 URL，薄层，只做布局和组合 | ProductPage、DashboardPage、NoteDetailPage |

### 层级间依赖规则

```
页面组件 → 业务组件 → UI 组件
（依赖方向：从左到右，不可反向）
```

| 规则 | 说明 |
|---|---|
| 页面组件可依赖业务组件和 UI 组件 | 页面是组装层 |
| 业务组件可依赖 UI 组件 | 业务组件封装逻辑，用 UI 组件呈现 |
| UI 组件不依赖业务组件或页面组件 | UI 组件是纯展示，不应感知业务 |
| 业务组件之间尽量避免直接依赖 | 通过状态管理或事件解耦 |

---

## 二、各层详解

### 2.1 UI 组件

**定义**：不包含任何业务逻辑的可复用展示组件。

**判断标准**：
- 只通过 props 接收数据，不自己获取数据
- 不包含 API 调用
- 不引用全局状态（theme 等基础 UI 状态除外）
- 可在不同业务场景中复用

**文件结构**（React 示例）：

```
components/
├── Button/
│   ├── Button.tsx          # 组件实现
│   ├── Button.test.tsx     # 测试（如有）
│   └── index.ts            # 导出
├── DataTable/
│   ├── DataTable.tsx
│   ├── DataTable.types.ts
│   └── index.ts
└── Modal/
    ├── Modal.tsx
    └── index.ts
```

**Astro 组件**（Content Register）：

```
components/
├── NoteCard.astro          # 纯展示，props 驱动
├── FormatBadge.astro       # 纯展示
├── TagList.astro           # 纯展示
├── Sidebar.astro           # 导航展示
└── Search.astro            # 搜索 UI（含 island 交互）
```

### 2.2 业务组件

**定义**：承载特定业务逻辑的组件，组合 UI 组件并注入数据和交互。

**判断标准**：
- 包含 API 调用或数据获取逻辑
- 包含业务状态管理
- 包含业务校验规则
- 通常与特定功能模块绑定

**文件结构**（Level 2+ feature-based）：

```
features/product/
├── components/
│   ├── ProductForm.tsx         # 业务组件：表单提交 + 校验
│   ├── ProductList.tsx         # 业务组件：数据获取 + 列表渲染
│   └── ProductDetail.tsx       # 业务组件：详情获取 + 展示
├── hooks/
│   └── useProduct.ts           # 业务 hook：数据获取 + 缓存
├── api/
│   └── productApi.ts           # API 调用封装
├── types.ts
└── index.ts                    # 模块公共 API
```

### 2.3 页面组件

**定义**：对应一个路由 URL，负责组装布局和业务组件。

**判断标准**：
- 对应一个路由路径
- 不包含复杂逻辑，只做组合和布局
- 从路由参数获取必要信息

**示例**：

```tsx
// pages/ProductPage.tsx（React）
export function ProductPage() {
  const { id } = useParams()
  return (
    <AppShell>
      <ProductDetail productId={id} />
    </AppShell>
  )
}
```

```astro
---
// pages/notes/[...slug].astro（Astro）
import HtmlNoteLayout from '../../layouts/HtmlNoteLayout.astro'
import NoteLayout from '../../layouts/NoteLayout.astro'

const { slug } = Astro.params
const note = await getNoteBySlug(slug)
---
{note.format === 'html' ? (
  <HtmlNoteLayout note={note} />
) : (
  <NoteLayout note={note} />
)}
```

---

## 三、应用壳组件

应用壳是整个前端应用的骨架框架，所有页面共享。

### 必备应用壳组件

| 组件 | 职责 | 优先级 | 说明 |
|---|---|---|---|
| AppShell / Layout | 全局布局框架 | P0 | 侧栏 + 顶栏 + 主内容区 |
| Sidebar | 侧边导航 | P0/P1 | 按项目类型决定复杂度 |
| Topbar / Header | 顶部导航栏 | P0/P1 | 面包屑、搜索、用户信息 |
| RouterOutlet | 路由出口 | P0 | 框架自带或自定义 |

### 应用壳设计原则

1. **布局稳定**：应用壳在所有页面保持一致，不随路由变化
2. **内容区独立**：主内容区完全由页面组件控制，应用壳不干预
3. **响应式**：侧栏在移动端可折叠，应用壳处理这个逻辑
4. **零业务逻辑**：应用壳不知道具体业务，只提供导航和布局框架

### Register 差异

| Register | 应用壳特点 |
|---|---|
| Content | 极简：顶栏 + 内容区，可能无侧栏。Astro 用 Layout 组件 |
| Tool | 经典三栏：侧栏导航 + 顶栏 + 主内容。用组件库的 Layout |
| Desktop | 多面板：主面板 + 侧边栏 + 可选底部栏。关注本地集成入口 |
| Hybrid | 按页面类型切换布局：内容页用轻量壳，工具页用完整壳 |

---

## 四、高频业务组件模板

以下是常见项目类型中几乎必然出现的业务组件，可直接在核心组件清单中引用：

### Tool Register（后台管理）

| 组件 | 职责 | 依赖 |
|---|---|---|
| DataTable | 数据表格 + 分页 + 排序 + 筛选 | UI 组件库的 Table |
| SearchForm | 搜索条件表单 | UI 组件库的 Form |
| DetailDrawer | 侧边详情抽屉 | UI 组件库的 Drawer |
| StatusTag | 状态标签（颜色映射） | UI 组件库的 Tag |
| ActionBar | 表格操作栏（新建/批量/导出） | UI 组件库的 Button |
| ConfirmDialog | 删除/操作确认弹窗 | UI 组件库的 Modal |
| PermissionGuard | 权限控制包装器 | 认证上下文 |

### Content Register（内容展示）

| 组件 | 职责 | 依赖 |
|---|---|---|
| ContentCard | 内容卡片（标题 + 摘要 + 标签 + 时间） | 无 |
| TagFilter | 标签过滤导航 | 无 |
| Breadcrumb | 面包屑导航 | 无 |
| TOC | 页内目录（h2/h3 提取） | 无 |
| Pagination | 分页导航 | 无 |
| SearchBox | 搜索入口 | Pagefind / 静态索引 |

### Desktop Register（桌面应用）

| 组件 | 职责 | 依赖 |
|---|---|---|
| CommandPalette | 命令面板（Ctrl+K） | shadcn/ui Command |
| SystemTray | 系统托盘集成 | Tauri/Electron API |
| FileExplorer | 文件浏览器 | 本地文件系统 API |
| SettingsPanel | 设置面板 | 表单组件 |

---

## 五、AI 友好命名规则

组件命名直接影响 AI 搜索、理解和修改的效率。
命名四原则统一遵循 `core/workflow.md` §7.5（可搜索、语义明确、遵循约定、模式稳定）。本节只补充组件领域的命名模式。

### 组件命名规则

| 类型 | 命名模式 | 示例 |
|---|---|---|
| UI 组件 | PascalCase，功能 + 类型 | `Button`、`DataTable`、`Modal`、`FormatBadge` |
| 业务组件 | PascalCase，业务实体 + 功能 | `ProductForm`、`OrderList`、`NoteCard` |
| 页面组件 | PascalCase，业务名 + Page | `ProductPage`、`DashboardPage` |
| Layout | PascalCase，用途 + Layout | `BaseLayout`、`NoteLayout`、`HtmlNoteLayout` |
| Hook (React) | camelCase，use + 功能 | `useProduct`、`useAuth`、`useDebounce` |
| Composable (Vue) | camelCase，use + 功能 | `useProduct`、`useAuth` |
| API 模块 | camelCase，实体 + Api | `productApi`、`orderApi` |
| 类型文件 | camelCase，实体名 | `product.types.ts` 或 `types.ts`（在 feature 内） |
| 工具函数 | camelCase，动词/功能 | `resolveNoteLink`、`formatDate`、`clamp` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY`、`DEFAULT_PAGE_SIZE` |

### 文件命名规则

| 规则 | 说明 |
|---|---|
| 一个组件一个文件 | `Button.tsx` 只包含 Button 组件 |
| index.ts 作为模块入口 | feature 目录用 `index.ts` 暴露公共 API |
| 类型与实现同目录 | `ProductForm.tsx` 和 `ProductForm.types.ts` 放一起 |
| 测试文件就近放置 | `ProductForm.test.tsx` 与组件同目录 |

---

## 六、组件封装决策树

**核心原则：先封装稳定重复的结构，不为了"看起来高级"而提前抽象。**

```
是否在 3+ 个地方用到相同结构？
├── 是 → 封装为组件
│   └── 是否包含业务逻辑？
│       ├── 是 → 业务组件（features/[x]/components/）
│       └── 否 → UI 组件（shared/components/ 或 components/）
└── 否 → 不封装，内联在页面中
    └── 如果页面内出现第 2 次 → 标记 TODO，等第 3 次再封装
```

### 封装时机判断

| 时机 | 动作 |
|---|---|
| 同一结构出现 3 次 | 封装为组件 |
| 结构复杂且容易出错 | 封装为组件（即使只用 1 次，如复杂表单） |
| 需要统一交互行为 | 封装为组件（如确认弹窗、加载状态） |
| 只在 1 个页面用，且结构简单 | 不封装，内联 |
| 预感"将来会复用" | 不封装，等实际复用时再做（YAGNI） |

### 反过早抽象原则

1. **不为了代码复用率而封装**：封装是为了降低认知负担，不是为了数字好看
2. **不因为"应该有个组件"而封装**：要有真实的重复场景
3. **不因为 AI 生成方便而封装**：AI 可以生成重复代码，过早封装反而增加维护成本
4. **优先用组合而非继承**：组件组合 > HOC > render props > 继承

---

## 七、Register 差异化建议

### Content Register

| 应该 | 不应该 |
|---|---|
| 用 `.astro` 组件（零 JS） | 默认用 React/Vue 组件 |
| 只在需要交互时用 island 组件 | 给纯展示内容加客户端框架 |
| 组件极简：卡片、标签、导航 | 封装复杂的表单/表格组件 |
| Layout 组件处理三栏/iframe | 在 Layout 中放业务逻辑 |

### Tool Register

| 应该 | 不应该 |
|---|---|
| 充分利用组件库的表格、表单 | 自己造表格/表单轮子 |
| 业务组件按 feature 组织 | 所有业务组件平铺 |
| 权限控制做成通用组件 | 每个页面单独写权限判断 |
| 状态管理放在业务组件内 | UI 组件直接读全局状态 |

### Desktop Register

| 应该 | 不应该 |
|---|---|
| 组件关注本地集成能力 | 忽略文件系统/系统托盘 |
| 用 shadcn/ui 保持轻量 | 引入重型组件库 |
| 命令面板作为核心组件 | 每个功能都做独立菜单 |

### Hybrid Register

| 应该 | 不应该 |
|---|---|
| 内容页和工具页用不同组件策略 | 全站统一用重型组件 |
| 共享部分提取到通用 UI 组件 | 通用 UI 组件包含业务逻辑 |

---

## 八、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| UI 组件内包含 API 调用 | 违反分层原则，UI 组件无法复用 | API 调用放在业务组件或 hook 中 |
| 页面组件包含大量业务逻辑 | 页面应该薄，难以测试和复用 | 业务逻辑下沉到业务组件或 hook |
| 所有组件平铺在 `components/` | 文件 > 20 后难以查找 | 按分层（UI/业务/页面）或按 feature 组织 |
| 组件名不语义化（`Comp1`、`Wrapper`） | AI 搜索和理解效率低 | 用 `ProductForm`、`DataTable` 等语义名 |
| 过早抽象：只出现 1 次就封装 | 增加间接层，降低可读性 | 等出现 3 次再封装 |
| 组件之间互相 import 内部文件 | 破坏封装，改一处动全局 | 通过 index.ts 暴露公共 API |
| 在 Astro 项目中默认用 React 组件 | 违背零 JS 优先原则 | 优先 .astro 组件，交互用 island |
| 对 Content Register 封装复杂表格/表单 | 静态站不需要这些 | 只封装卡片、标签、导航等轻量组件 |
| 1000+ 行的组件 | 认知负担过重 | 拆分为子组件，保持单文件 < 300 行 |
| `any` 类型的组件 props | AI 无法推断正确用法 | 所有 props 定义 TypeScript 类型 |

---

## 九、与 output-template.md §7 的映射

本 reference 的内容与输出模板 §7（核心组件清单）直接对应：

| 本文件章节 | 输出模板对应 | 输出方式 |
|---|---|---|
| §三 应用壳组件 | 应用壳组件表格 | 按项目类型选择必备组件 |
| §四 高频业务组件 | 业务组件表格 | 按 Register 选择对应的常用组件 |
| §五 命名规则 | 组件命名贯穿全文 | 在组件清单中使用规范命名 |
| §六 封装决策树 | 组件封装原则 | 输出 4 条原则 |
