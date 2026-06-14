---
# 前端架构方案摘要（供下游 skill / agent 快速读取）
framework: React
register: Desktop
complexity_level: 2
decisive_dimension: 业务复杂度
component_library: shadcn/ui + Radix UI
styling: Tailwind CSS + CSS Variables
state_management: Zustand 4 (persist middleware)
architecture_focus: Agent/Skill 双实体管理 + Tauri IPC 桥接 + Zustand 与文件系统双向同步
mvp_scope: P0 可运行闭环：Agent 侧栏 + Skill 网格/详情 + 新建 Skill + Tauri FS 读写持久化
created: 2026-06-13
updated: 2026-06-13
---

<!-- 下游 coding agent 使用指引：
     1. 先读 YAML frontmatter 了解全局决策
     2. 再读"架构约束"区了解不可违反的底线
     3. 按章节顺序理解架构方案
     4. 编码前检查 MVP 边界
-->

# SkillHub 前端架构设计方案

## 1. 项目理解摘要

**项目名称**：SkillHub
**项目类型**：桌面应用（Tauri 2.0）
**核心用户**：AI Agent 使用者 / 开发者——需要管理多个 Agent（如 Claude Code、Cursor、Codex）下各类 Skill 配置的人群
**核心场景**：在本地桌面环境中管理多个 AI Agent，为每个 Agent 创建、编辑、启用/禁用、测试运行各类 Skill，并将 Skill 配置持久化为本地 SKILL.md 文件，实现 Agent Skill 的可视化管理和文件系统级持久化
**前端形态**：工具型 / 管理型（三面板布局：AgentSidebar + SkillGrid + SkillDetail）
**技术约束**：
- 桌面框架已定：Tauri 2.0（Rust 后端 + WebView 前端）
- 前端框架已定：React 18 + TypeScript 5
- 构建工具已定：Vite 5
- 状态管理已定：Zustand 4（含 persist middleware）
- 样式方案调整：shadcn/ui + Radix UI + Tailwind CSS，保留 CSS Variables 作为主题 Token 来源
- 文件 I/O：Tauri FS Plugin（Rust 命令桥接）
- 设计书已完整定义类型、Store、组件、Tauri 命令

---

## 2. 复杂度判定

| 维度 | 判定 | 说明 |
|---|---|---|
| 团队规模 | Level 1 | 个人项目，1 人维护 |
| 生命周期 | Level 2 | 预期 > 3 个月持续迭代，P0-P3 四阶段交付 |
| 业务复杂度 | Level 2 | Agent + Skill 双实体管理，Zustand-to-文件系统双向同步，SKILL.md 序列化/反序列化，多状态联动（选中 Agent → 过滤 Skill → 展示详情） |

**最终 Level**：2（决定性维度：业务复杂度，架构侧重：领域拆分 + 状态架构 + 数据流设计）

---

## 3. 推荐技术栈

| 维度 | 选择 | 理由 |
|---|---|---|
| 框架 | React 18 | 设计书已定；组件生态丰富，Hooks 模型与 Zustand 配合简洁 |
| 语言 | TypeScript 5 | 设计书已定；类型安全，AI 生成代码可验证 |
| 构建工具 | Vite 5 | 设计书已定；Tauri 官方推荐，极速 HMR |
| 样式方案 | Tailwind CSS + CSS Variables | shadcn/ui 标准搭配；CSS Variables 承接 tokens.css 的主题变量，Tailwind 负责原子化样式和组件状态样式 |
| 组件库 | shadcn/ui + Radix UI | Desktop Register 的轻量、源码可控方案；保留设计书已定义的 Button/Badge/Modal/Input 语义，但实现落到 shadcn/ui 组件源码和 Radix primitives |
| 状态管理 | Zustand 4 + persist middleware | 设计书已定；轻量、TypeScript 友好；persist 仅缓存已提交成功的状态快照 |
| 数据获取 | Tauri IPC（invoke） | 桌面应用无 HTTP API，通过 Tauri invoke 调用 Rust 命令实现文件系统操作 |
| 图标库 | lucide-react | 设计书已定（Agent.icon 使用 lucide 图标名） |
| 路由 | 无 SPA 路由 | 单窗口三面板布局，通过 Zustand selectedAgentId / selectedSkillId 控制视图切换，不需要 react-router |

---

## 4. UI 组件库选择与理由

### 选择：shadcn/ui + Radix UI

### 选择理由

**设计书现状**：设计书已完整定义了 Button、Badge、Modal、Input 等基础组件语义和业务组件边界。架构方案保留这些组件接口和产品语义，但实现方式调整为 shadcn/ui + Radix UI + Tailwind CSS，避免后续再从 CSS Modules 自建体系迁移。

- **与 Register 匹配度**：Desktop Register 需要现代轻量、可高度定制的桌面工具 UI；shadcn/ui + Radix UI 适合 Tauri WebView 内的复杂交互组件
- **AI 友好度**：shadcn/ui 组件源码复制到项目内，命名规律、类型完整、可直接修改；Radix 提供稳定无障碍 primitive
- **定制能力**：通过 CSS Variables + Tailwind token 映射定制主题；组件源码本地可控，无黑盒
- **Bundle 影响**：按需复制组件，避免引入重型组件库运行时

### 与设计书自建组件的关系

设计书中的 Button / Badge / Modal / Input 作为**组件语义和 Props 需求来源**保留；实际实现优先使用 shadcn/ui 已有组件：

| 设计书组件 | shadcn/ui / Radix 映射 | 说明 |
|---|---|---|
| Button | `button` | variant / size 通过 `class-variance-authority` 管理 |
| Badge | `badge` | color 映射到 Agent / Skill 状态 token |
| Modal | `dialog` | 基于 Radix Dialog，覆盖 NewSkillModal / EditSkillModal |
| Input | `input` | 支持错误状态和前缀图标 |
| StatusDot | 自定义轻量组件 | 仅状态点，不需要外部 primitive |

### 排除方案

| 被排除的库 | 排除理由 |
|---|---|
| Ant Design | Desktop Register 反模式——风格偏传统后台，深度定制成本高，Bundle 体积大，桌面端体验差 |
| MUI | Material Design 风格较重，Tauri WebView 内渲染 MUI 组件性能不如轻量方案 |
| Chakra UI | 虽然主题系统友好，但运行时样式方案（CSS-in-JS）增加桌面端 Bundle 和渲染成本 |
| CSS Modules 全自建组件 | 设计书方案可行，但会让 Modal、焦点管理、键盘导航等交互组件重复造轮子；MVP 直接采用 shadcn/ui 更稳 |

---

## 5. 目录结构方案

按 Level 2 feature-based 结构，结合 Tauri 项目约定和设计书已有结构：

```
skillhub/
├── src-tauri/                       # Rust/Tauri 后端（设计书已定）
│   ├── src/
│   │   ├── main.rs                  # Tauri 入口，注册插件与命令
│   │   ├── commands/
│   │   │   ├── mod.rs
│   │   │   ├── skill.rs             # Skill 相关 Tauri 命令
│   │   │   └── agent.rs             # Agent 相关 Tauri 命令
│   │   └── lib.rs
│   ├── Cargo.toml
│   └── tauri.conf.json
│
├── src/                             # React 前端
│   ├── features/                    # 按功能模块组织
│   │   ├── agent/                   # Agent 领域
│   │   │   ├── components/
│   │   │   │   └── AgentSidebar.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useSelectedAgent.ts
│   │   │   └── index.ts
│   │   │
│   │   └── skill/                   # Skill 领域
│   │       ├── components/
│   │       │   ├── SkillGrid.tsx
│   │       │   ├── SkillCard.tsx
│   │       │   ├── SkillDetail.tsx
│   │       │   ├── NewSkillModal.tsx
│   │       │   └── EditSkillModal.tsx
│   │       ├── hooks/
│   │       │   ├── useFilteredSkills.ts
│   │       │   └── useSelectedSkill.ts
│   │       └── index.ts
│   │
│   ├── shared/                      # 跨功能共享
│   │   ├── components/              # 通用 UI 组件
│   │   │   ├── ui/                  # shadcn/ui 复制进项目的基础组件
│   │   │   │   ├── button.tsx
│   │   │   │   ├── badge.tsx
│   │   │   │   ├── dialog.tsx
│   │   │   │   └── input.tsx
│   │   │   └── StatusDot.tsx        # 项目自定义轻量组件
│   │   ├── hooks/                   # 通用 hooks
│   │   │   └── useTauriReady.ts
│   │   ├── lib/                     # 工具与桥接层
│   │   │   ├── tauri.ts             # Tauri invoke 封装
│   │   │   ├── skillParser.ts       # SKILL.md 序列化/反序列化
│   │   │   └── utils.ts             # 通用工具函数
│   │   └── types/                   # 共享类型
│   │       └── index.ts             # 全项目单一类型来源
│   │
│   ├── app/                         # 应用壳
│   │   ├── App.tsx                  # 根组件，三面板布局骨架
│   │   └── providers.tsx            # （预留）Context providers
│   │
│   ├── store/                       # Zustand Store
│   │   ├── index.ts                 # Store 定义（含 persist）
│   │   └── selectors.ts            # 派生查询 selectors
│   │
│   ├── styles/                      # Design Token / 全局样式
│   │   ├── tokens.css               # 设计 Token（颜色、间距、字体、布局）
│   │   └── globals.css              # CSS reset、滚动条、全局基础
│   │
│   ├── assets/
│   │   └── icon.svg
│   │
│   └── main.tsx                     # React 挂载入口
│
├── public/
├── index.html
├── docs/
│   └── frontend-architecture.md     # 本文件
├── vite.config.ts
├── tsconfig.json
└── package.json
```

### 设计书目录 → feature-based 映射

设计书的 `src/components/` 扁平结构映射为 feature-based：

| 设计书原路径 | 迁移目标 | 理由 |
|---|---|---|
| `src/components/AgentSidebar/` | `src/features/agent/components/` | Agent 领域专用组件 |
| `src/components/SkillGrid/` | `src/features/skill/components/` | Skill 领域专用组件 |
| `src/components/SkillDetail/` | `src/features/skill/components/` | Skill 领域专用组件 |
| `src/components/modals/` | `src/features/skill/components/` | Skill 弹窗归属 Skill 领域 |
| `src/components/common/` | `src/shared/components/ui/` + `src/shared/components/StatusDot.tsx` | 基础组件使用 shadcn/ui；项目特定轻量组件单独放置 |
| `src/hooks/` | `src/features/*/hooks/` + `src/shared/hooks/` | 按归属拆分 |
| `src/lib/` | `src/shared/lib/` | Tauri 桥接和工具函数 |
| `src/types/` | `src/shared/types/` | 全项目单一类型来源 |
| `src/store/` | `src/store/`（保持不变） | Store 是跨领域的基础设施，不归属任何 feature |

### 目录职责说明

| 目录 | 职责 | 放什么 | 不放什么 |
|---|---|---|---|
| `src/features/agent/` | Agent 领域模块 | AgentSidebar 组件、Agent 相关 hooks | Skill 相关组件或逻辑 |
| `src/features/skill/` | Skill 领域模块 | SkillGrid/SkillCard/SkillDetail/Modal 组件、Skill hooks | Agent 相关逻辑 |
| `src/shared/components/ui/` | shadcn/ui 基础组件 | button、badge、dialog、input 等源码复制组件 | 业务逻辑、API 调用 |
| `src/shared/components/` | 项目级通用轻量组件 | StatusDot、EmptyState、ErrorState | Agent / Skill 领域逻辑 |
| `src/shared/lib/` | 工具与桥接 | tauri.ts（IPC 封装）、skillParser.ts、utils.ts | React 组件 |
| `src/shared/types/` | 全局类型定义 | TypeScript interfaces、enums、type aliases | 运行时代码 |
| `src/shared/hooks/` | 通用 hooks | useTauriReady 等非领域 hooks | 业务逻辑 |
| `src/store/` | Zustand Store | Store 定义、selectors | React 组件 |
| `src/app/` | 应用壳 | App.tsx 根布局、providers | 业务组件 |
| `src/styles/` | Design Token + 全局样式 | tokens.css、globals.css | 组件 scoped 样式 |
| `src-tauri/` | Rust 后端 | Tauri 命令、权限配置、Cargo.toml | 前端代码 |

---

## 6. 模块边界与领域拆分

### 领域划分

| 域类型 | 模块 | 职责 | 依赖 |
|---|---|---|---|
| 核心域 | `features/agent` | Agent 实体的展示、选择、CRUD | shared |
| 核心域 | `features/skill` | Skill 实体的展示、过滤、详情、CRUD、测试运行 | shared、store |
| 通用域 | `shared/components` | 纯 UI 组件（Button、Badge、Modal、Input） | 无 |
| 通用域 | `shared/lib` | Tauri IPC 桥接、SKILL.md 解析、工具函数 | shared/types |
| 通用域 | `shared/types` | 全局 TypeScript 类型定义 | 无 |
| 基础设施 | `store` | Zustand Store + selectors + persist | shared/types |
| 基础设施 | `src-tauri` | Rust 后端文件系统操作 | 无 |

### 模块间依赖规则

```
核心域（features/agent, features/skill）
  → 通用域（shared/）
    → 基础设施（store, src-tauri）

依赖方向：从左到右，不可反向
- 核心域之间通过 store 解耦，不直接 import
- 通用域不依赖核心域
- shared/types 是最底层，不依赖任何其他模块
```

### 跨域数据流

```
Agent 侧栏选择 → store.selectAgent(id)
                  → Skill 领域响应 selectedAgentId 变化
                  → useFilteredSkills() 自动过滤
                  → SkillGrid 重新渲染
                  → SkillDetail 响应 selectedSkillId
```

---

## 7. 核心组件清单

### 应用壳组件

| 组件 | 职责 | 优先级 |
|---|---|---|
| App | 全局三面板布局骨架：AgentSidebar + SkillGrid + SkillDetail（条件渲染）+ Modal（条件渲染） | P0 |

### 业务组件

| 组件 | 职责 | 所属模块 | 优先级 |
|---|---|---|---|
| AgentSidebar | 左侧栏：Agent 列表 + 新建按钮 + 选中高亮 | features/agent | P0 |
| SkillGrid | 中间主区：Topbar + CategoryTabs + SkillCard 列表 | features/skill | P0 |
| SkillCard | 技能卡片：名称、描述、状态指示点、分类 Badge、操作按钮 | features/skill | P0 |
| SkillDetail | 右侧详情面板：Skill 全量信息 + 启用切换入口（测试运行可先禁用） | features/skill | P0 |
| NewSkillModal | 新增技能弹窗：Skill 表单（名称、描述、分类、参数配置） | features/skill | P1 |
| EditSkillModal | 编辑技能弹窗：复用 NewSkillModal 表单，预填数据 | features/skill | P2 |

### 通用组件

| 组件 | 职责 | 优先级 |
|---|---|---|
| Button | 通用按钮：4 variant（primary/secondary/ghost/danger）× 3 size + loading + icon | P0 |
| Badge | 标签徽章：5 color（purple/teal/amber/coral/gray） | P0 |
| Modal | 通用弹窗：可配宽度、title、footer | P0 |
| Input | 通用输入框：支持前缀图标、错误状态 | P0 |
| StatusDot | 状态指示点：active(绿)/draft(黄)/disabled(灰) + glow 效果 | P0 |
| EmptyState | 空状态展示：无 Agent / 无 Skill / 搜索无结果 | P1 |
| ErrorState | 错误状态展示 | P1 |

### 组件封装原则

1. 先用设计书已定义的组件（Button/Badge/Modal/Input），不重复造轮子
2. 业务组件通过 store selectors 获取数据，不直接调用 tauriApi
3. 组件样式优先使用 Tailwind class 与 CSS Variable Token；复杂组合样式抽到组件本地 helper，不再新增 CSS Module 文件
4. 组件命名遵循 AI 友好规则：可搜索（`AgentSidebar` 而非 `Sidebar`）、语义明确、遵循约定

---

## 8. Design Token 初稿

基于设计书 `src/styles/tokens.css` 已定义的 Token 体系，整理为规范格式：

### Spacing

```css
:root {
  --space-1: 4px;      /* 基础单元 */
  --space-2: 8px;      /* 组件内间距 */
  --space-3: 12px;     /* 紧凑区块间距 */
  --space-4: 16px;     /* 标准内边距 */
  --space-5: 20px;     /* 卡片内边距 */
  --space-6: 24px;     /* 区块间距 */
}
```

### Radius

```css
:root {
  --radius-sm: 6px;    /* Badge、小型 inline 元素 */
  --radius-md: 10px;   /* 按钮、输入框 */
  --radius-lg: 14px;   /* 卡片 */
  --radius-xl: 20px;   /* 弹窗 */
}
```

### Color

```css
:root {
  /* 背景层次（dark mode 默认） */
  --bg:        #0e0e10;              /* 页面底色 */
  --bg-2:      #18181b;              /* 侧栏、面板底色 */
  --bg-3:      #1f1f23;              /* 卡片、输入框底色 */
  --bg-4:      #27272a;              /* hover 状态底色 */

  /* 文字层次 */
  --text-1:    #f4f4f5;              /* 主要文字 */
  --text-2:    #a1a1aa;              /* 次要文字 */
  --text-3:    #71717a;              /* 提示文字 */

  /* 边框 */
  --border:    rgba(255,255,255,0.07);
  --border-2:  rgba(255,255,255,0.12);

  /* 品牌色（Agent 主题色 + 功能色） */
  --purple:    #534AB7;              /* Agent 默认主题色 */
  --purple-l:  #7F77DD;              /* Purple 亮色变体 */
  --purple-bg: rgba(83,74,183,0.15); /* Purple 背景色 */
  --teal:      #0F6E56;              /* Teal 主题色 */
  --teal-l:    #1D9E75;
  --teal-bg:   rgba(29,158,117,0.15);
  --amber:     #854F0B;              /* Amber 主题色 */
  --amber-l:   #BA7517;
  --amber-bg:  rgba(186,117,23,0.15);

  /* 状态色（Skill 状态指示点） */
  --status-active:   #22c55e;        /* 绿色：active */
  --status-draft:    #f59e0b;        /* 黄色：draft */
  --status-disabled: #71717a;        /* 灰色：disabled */
}
```

### Typography

```css
:root {
  --font-sans: 'DM Sans', system-ui, sans-serif;
  --font-mono: 'DM Mono', 'Fira Code', monospace;

  --text-xs:   0.75rem;    /* 12px */
  --text-sm:   0.875rem;   /* 14px */
  --text-base: 1rem;       /* 16px */
  --text-lg:   1.125rem;   /* 18px */
  --text-xl:   1.25rem;    /* 20px */
  --text-2xl:  1.5rem;     /* 24px */

  --leading-tight:  1.25;
  --leading-normal: 1.5;
}
```

### Layout

```css
:root {
  --sidebar-w:  220px;     /* AgentSidebar 宽度 */
  --detail-w:   280px;     /* SkillDetail 面板宽度 */
  --topbar-h:   56px;      /* SkillGrid 顶栏高度 */
}
```

### Token 使用规则

- 所有颜色、间距、圆角必须引用 token 变量，禁止写裸十六进制色值或 px 数字
- 新增 token 先在 `tokens.css` 定义，再使用
- 品牌色按 Agent.color 动态映射（purple/teal/amber/coral/blue），不硬编码
- 深色主题为默认；Light 主题在 P2/P3 阶段通过 `[data-theme="light"]` 覆盖

---

## 9. 数据与状态架构

### 状态分类

| 数据类型 | 方案 | 说明 |
|---|---|---|
| 文件系统状态（fs-state） | Tauri IPC invoke → Zustand store | SKILL.md 文件通过 Rust 命令读写，结果同步到 Zustand store；本地桌面应用的核心"数据源"是文件系统而非 HTTP API |
| 全局持久化状态（persisted-state） | Zustand 4 + persist middleware | 只持久化**已提交成功**的 agents[] + skills[] 快照；localStorage 是快速恢复缓存，不是 source of truth |
| 全局 UI 状态（client-state） | Zustand 4（不持久化） | selectedAgentId、selectedSkillId、searchQuery、activeCategory、弹窗开关；纯客户端交互状态 |
| 组件局部状态 | React useState | 表单输入值、弹窗内临时数据、hover 状态 |

### 双向同步架构：Zustand ↔ 文件系统

本项目数据架构的核心是 **Zustand Store 与磁盘 SKILL.md 文件的双向同步**。这不是典型的 server-state（无 HTTP API），而是桌面应用特有的 fs-state 模式：

```
┌──────────────────────────────────────────────────────────────────┐
│                        用户操作                                    │
│  （新建 Skill / 编辑 Skill / 删除 Skill / 切换状态）              │
└──────────┬───────────────────────────────────────┬───────────────┘
           │                                       │
           ▼                                       │
   Service Action                                  │
   createSkill / updateSkill / deleteSkill         │
           │                                       │
           ├─→ 校验 + serializeSkillMd             │
           │                                       │
           └─→ 调用 tauriApi.*                     │
               (异步 IPC)                           │
               │                                    │
               ▼                                    │
         Rust 后端                                  │
         save_skill / delete_skill_file             │
               │                                    │
               ▼                                    │
         磁盘 SKILL.md                              │
               │                                    │
               ▼                                    │
         commit Zustand Store                       │
         + persist 已提交快照                        │
                                                    │
                                                    ▼
                              ┌──────────────────────┐
                              │ App 启动时            │
                              │ tauriApi.loadSkills() │
                              │ → Rust 读磁盘        │
                              │ → Skill[] 返回前端    │
                              │ → 写入 Zustand store  │
                              └──────────────────────┘
```

**关键设计决策**：

1. **写入路径**：UI action → 校验/序列化 → Tauri IPC 写磁盘 → 成功后 commit 到 Zustand Store → persist middleware 写入 localStorage。不得在磁盘写入失败前把业务数据提交为已保存状态
2. **读取路径**：磁盘 → Tauri IPC → Store。应用启动时从磁盘加载所有 SKILL.md，解析后写入 store 作为初始状态
3. **冲突处理**：磁盘是 source of truth；Store 是已提交状态的内存镜像；localStorage 是上一次已提交状态的启动缓存
4. **persist 中间件**：只缓存已提交成功的状态。Tauri IPC 尚未就绪时可用缓存渲染只读 UI，但必须在磁盘加载完成后以磁盘结果覆盖缓存

### 事务规则（写入 / 删除 / 启动）

为避免“UI 显示已保存但磁盘未落盘”，所有会修改业务数据的操作必须遵循以下事务规则：

| 操作 | 事务步骤 | 失败处理 | Store 提交时机 |
|---|---|---|---|
| 新建 Skill | 表单校验 → 生成 Skill 草案 → `serializeSkillMd` → `tauriApi.saveSkill` → 返回 `filePath` → `store.addSkillCommitted` | 不写入 store；保留表单输入；toast / Modal 展示错误 | 磁盘写入成功后 |
| 编辑 Skill | 基于当前 Skill 生成 patch → `serializeSkillMd` → `tauriApi.saveSkill` → `store.updateSkillCommitted` | 不覆盖 store；保留编辑弹窗；提示可重试 | 磁盘写入成功后 |
| 删除 Skill | 记录待删除 id → `tauriApi.deleteSkillFile` → `store.deleteSkillCommitted` | 不从 store 移除；提示删除失败 | 磁盘删除成功后 |
| 切换状态 | 生成 status patch → `tauriApi.saveSkill` → `store.updateSkillCommitted` | 状态按钮回到原值；提示失败 | 磁盘写入成功后 |
| App 启动 | 先渲染 localStorage 快照（只读 / loading 标记）→ `tauriApi.loadSkills` → 以磁盘结果覆盖 store | 磁盘读取失败时保留缓存但标记“未同步”，禁止写操作直到用户选择目录或重试成功 | 磁盘读取成功后 |

Store action 命名建议区分“请求”和“提交”：

```typescript
// UI 层 / service 层调用，包含 IPC 事务
createSkill(data)
updateSkill(id, patch)
deleteSkill(id)
toggleSkillStatus(id, status)

// store 内部提交，仅在 IPC 成功后调用
addSkillCommitted(skill)
updateSkillCommitted(id, patch)
deleteSkillCommitted(id)
```

如果后续确实需要乐观 UI，必须先为 Skill 增加 `syncStatus: "synced" | "pending" | "error"`，并在架构变更记录中登记；MVP 不采用乐观提交。

### 数据获取层（Tauri IPC 桥接）

```
src/shared/lib/tauri.ts    ← 统一 IPC 封装入口
  ├─ tauriApi.saveSkill()       → invoke("save_skill", ...)
  ├─ tauriApi.loadSkills()      → invoke("load_skills", ...)
  ├─ tauriApi.deleteSkillFile() → invoke("delete_skill_file", ...)
  └─ tauriApi.runSkill()        → invoke("run_skill", ...)

src/shared/lib/skillParser.ts  ← SKILL.md 序列化/反序列化
  ├─ parseSkillMd(content: string): SkillFileContent
  └─ serializeSkillMd(skill: Skill): string
```

**封装原则**：
- 所有 Tauri 命令调用必须通过 `tauriApi.*` 封装，禁止直接 `invoke("xxx")`
- IPC 调用统一 try/catch，错误分类为 `tauri | parse | network` 三类
- `parseSkillMd` 和 `serializeSkillMd` 必须互为逆操作

### 全局状态边界

| 放全局（Zustand Store） | 不放全局 |
|---|---|
| agents[]、skills[]（持久化数据） | 表单输入值（Modal 内临时数据） |
| selectedAgentId、selectedSkillId（跨面板共享） | hover 状态、动画状态 |
| searchQuery、activeCategory（跨组件过滤） | 单个 SkillCard 的局部 UI 状态 |
| isNewSkillOpen、isEditSkillOpen（弹窗控制） | Modal 内的表单校验错误 |
| 侧栏折叠状态（如有） | tooltip 显隐 |

### Zustand Store Selector 策略

```typescript
// 组件中取数据时，使用 selectors，避免整棵 store 重渲染
const agents    = useStore((s) => s.agents);
const skills    = useFilteredSkills();      // 派生 selector
const selected  = useSelectedSkill();       // 派生 selector
const agent     = useSelectedAgent();       // 派生 selector
```

新增派生数据需求时，在 `store/selectors.ts` 添加新 selector，不在组件内写过滤逻辑。

---

## 10. 主题与国际化预留

本项目复杂度较低，此项暂不展开。

> 说明：设计书以深色主题为默认，采用 CSS Variables 管理颜色。后续如需 Light/Dark 切换，通过 `[data-theme="light"]` 覆盖 `tokens.css` 中的变量即可。项目为中文桌面工具，无需国际化。

---

## 11. MVP 第一阶段范围

### P0 可运行闭环定义

P0 的目标不是“完整管理后台”，而是验证 SkillHub 的核心桌面闭环：

```text
启动应用
  → 读取本地 Skill 数据
  → 选择 Agent
  → 查看 Skill 列表与详情
  → 新建一个 Skill
  → 写入磁盘 SKILL.md
  → 重启应用后从磁盘恢复
```

P0 必须满足：

- 有真实 Tauri FS 读写链路，不只依赖 localStorage
- 新建 Skill 成功后磁盘上能看到可读的 SKILL.md
- 重启后以磁盘数据恢复 store
- 写入失败不会污染 store 中的已提交业务数据

### 必须完成的页面/视图

| 页面/视图 | 核心功能 | 优先级 |
|---|---|---|
| App 三面板布局 | AgentSidebar + SkillGrid + SkillDetail 条件渲染 | P0 |
| AgentSidebar | Agent 列表展示 + 选中高亮 + 新建 Agent | P0 |
| SkillGrid + SkillCard | 按当前 Agent 过滤展示 Skill 卡片 + 状态指示点 | P0 |
| SkillDetail 面板 | Skill 详情展示 + 启用/禁用切换入口（可先禁用测试运行） | P0 |
| NewSkillModal | 创建 Skill 表单 + 按事务规则写入磁盘 + commit store | P0 |
| EditSkillModal | 编辑 Skill 表单 + 更新 store + 同步磁盘 | P1 |

### 必须封装的组件

| 组件 | 理由 | 优先级 |
|---|---|---|
| Button | 所有交互的基础，4 variant 覆盖全局按钮需求 | P0 |
| Badge | Skill 分类标签 + Agent 主题色标识 | P0 |
| Modal | NewSkillModal / EditSkillModal 的基础壳 | P0 |
| StatusDot | Skill 状态指示点，3 种状态 + glow 效果 | P0 |
| SkillCard | Skill 列表核心单元，跨列表/详情复用 | P0 |

### 必须实现的基础设施

| 基础设施 | 说明 | 优先级 |
|---|---|---|
| TypeScript 类型体系 | `src/shared/types/index.ts` 全部接口和类型 | P0 |
| Zustand Store + persist | `src/store/index.ts` 完整 store 定义 | P0 |
| Tauri IPC 封装 | `src/shared/lib/tauri.ts` 统一调用入口 | P0 |
| SKILL.md 解析器 | `src/shared/lib/skillParser.ts` 序列化/反序列化 | P0 |
| shadcn/ui 基础组件 | button、badge、dialog、input | P0 |
| Tailwind + CSS Variable Token 体系 | `src/styles/tokens.css` + Tailwind token 映射 | P0 |

### 明确延后的能力

| 能力 | 延后理由 | 预计引入阶段 |
|---|---|---|
| 搜索 + 分类过滤增强 | P0 只保留最小过滤能力；复杂搜索、排序和多条件组合延后 | P3 |
| 调用统计展示（callCount / avgLatencyMs） | 数据展示优先于统计分析 | P3 |
| Light/Dark 主题切换 | 深色主题足够 MVP | P2/P3 |
| 命令面板（Ctrl+K） | Desktop 高级交互 | P3 |
| 系统托盘集成 | 非核心功能 | P3 |
| 自动更新 | Tauri Updater 配置，非 MVP | P3 |
| Agent 拖拽排序 | skillIds 有序列表已有，UI 交互延后 | P2 |
| Skill 批量操作 | 多选 + 批量启用/禁用/删除 | P2 |

### 可接受的技术债

| 技术债 | 接受理由 | 清理时机 |
|---|---|---|
| 无测试覆盖 | MVP 先验证产品可行性 | P1：核心流程补测试 |
| SKILL.md 解析器错误边界简单 | MVP 阶段 SKILL.md 格式可控 | P2：补健壮性 |
| run_skill 为 mock 实现 | 真实执行需要 Agent 运行时集成 | P3 |
| localStorage 仅作为已提交快照缓存 | Tauri IPC 未就绪时可快速渲染只读 UI | P2：评估是否需要去掉业务数据缓存，只保留 UI 偏好 |
| 无骨架屏 loading | 功能优先 | P2 |
| 无 a11y 处理 | 产品验证优先 | P1：补基础 a11y（键盘导航、焦点管理） |
| Store 与磁盘同步无冲突检测 | 单用户桌面应用，无并发写入 | P3：如需多窗口再补 |

### 不可接受的技术债

- **不得绕过 `tauriApi.*` 直接调用 `invoke()`**——必须通过统一封装，保证类型安全和错误处理
- **不得在组件内直接操作 `localStorage`**——localStorage 只能由 Zustand persist middleware 缓存已提交快照
- **不得绕过 Store 直接管理业务数据**——组件只通过 `useStore(selector)` 取数据、store actions 改数据
- **必须按 feature-based 目录结构组织**——不回退到设计书的扁平 `components/` 结构
- **必须使用 shadcn/ui + Radix UI 作为基础组件实现**——不回退到 CSS Modules 全自建基础组件
- **必须实现全局错误兜底**——至少有 try/catch + toast 提示，不允许白屏
- **Tauri 命令入参/出参必须类型化**——复用 `src/shared/types/index.ts` 的接口定义

---

## 12. 后续编码建议

### 编码前

1. 按 feature-based 目录结构创建项目骨架（`features/agent/`、`features/skill/`、`shared/`、`app/`、`store/`、`styles/`）
2. 用 `npm create tauri-app@latest` 初始化 Tauri 2.0 + React + TypeScript + Vite 项目
3. 实现 `src/shared/types/index.ts`——全项目类型定义的单一来源
4. 配置 Tailwind CSS + shadcn/ui，并实现 `src/styles/tokens.css` + `globals.css`——Design Token 基础
5. 实现 `src/store/index.ts`——Zustand Store + persist + 所有 actions
6. 实现 `src/store/selectors.ts`——派生查询
7. 实现 `src/shared/lib/tauri.ts`——Tauri IPC 统一封装

### 编码中

1. 每个功能开发前检查 MVP 范围和优先级表
2. 新增组件遵循分层原则：`shared/components/ui/`（shadcn 基础 UI）→ `shared/components/`（项目级通用组件）→ `features/*/components/`（业务）
3. 样式使用 Token 变量，不硬编码颜色和间距
4. 组件取数据用 selectors，改数据用 store actions，不在组件内写过滤逻辑
5. 新增 Tauri 命令时：先 Rust → 注册 → 再 `tauri.ts` 封装
6. 每次修改后运行 `npx tsc --noEmit` 确认无类型错误

### 编码后

1. 运行 `npm run tauri dev` 验证完整用户旅程：启动 → 选 Agent → 看 Skill → 新建 Skill → 编辑 Skill → 切换状态
2. 验证持久化事务：新建 / 编辑 / 删除成功后磁盘和 store 一致；模拟写入失败时 store 不提交脏数据
3. 验证 SKILL.md 文件：检查磁盘上生成的 `.md` 文件格式正确、可人工阅读
4. 验证错误处理：模拟 Tauri 命令失败（如目录不存在），确认有 toast 提示而非白屏

### 衔接建议（可选）

| 后续步骤 | 推荐方式 | Fallback |
|---|---|---|
| 视觉设计 | Taste Skill（深色桌面工具风格） | 本方案中的 Design Token 基础 |
| 编码流程 | TDD：先类型 → 再 Store → 再组件 | 本方案的优先级表顺序 |
| 组件库演进 | P2/P3 补齐更多 shadcn/ui 组件和组合组件 | 保持当前 shadcn 基础组件集 |
| Rust 后端 | 按设计书 §6 实现 4 个命令 | — |

---

## 架构决策记录

| 日期 | 决策 | 理由 | 替代方案 | 触发方 |
|---|---|---|---|---|
| 2026-06-13 | 复杂度判定 Level 2 | Agent+Skill 双实体 + Zustand↔FS 双向同步 + 多状态联动 | Level 1（低估了同步复杂度） | 架构判定 |
| 2026-06-13 | feature-based 目录结构 | 双实体管理适合按领域拆分，设计书扁平结构在 Skill 种类增多后难以维护 | 设计书原扁平 `components/` 结构 | 架构判定 |
| 2026-06-13 | 使用 shadcn/ui + Radix UI + Tailwind CSS | Desktop Register 需要轻量、可控、AI 友好的桌面组件；用户明确要求使用 shadcn | CSS Modules 全自建组件 | 用户决策 |
| 2026-06-13 | Tauri FS 成功后再提交 Store | 避免 localStorage 缓存写入失败后的脏状态；磁盘是 source of truth | Store 先乐观提交再补偿 | 用户决策 |
| 2026-06-13 | 无 SPA 路由 | 三面板固定布局，视图切换由 store 状态驱动 | react-router（过度设计） | 架构判定 |
| 2026-06-13 | store/selectors.ts 独立文件 | 派生查询逻辑集中管理，组件零过滤逻辑 | selector 内联在 store/index.ts | 架构判定 |

---

## 架构约束（下游 skill 应遵守）

- **技术栈**：不得更改 Tauri 2.0、React 18、TypeScript 5、Zustand 4、Vite 5、shadcn/ui、Radix UI、Tailwind CSS 的选择
- **目录结构**：新增目录应遵循 feature-based 分层规则（features/agent、features/skill、shared/、app/、store/）
- **组件分层**：shadcn 基础 UI 组件放 `shared/components/ui/`，项目级通用组件放 `shared/components/`，业务组件放 `features/*/components/`，不得混放
- **Tauri 调用**：不得绕过 `tauriApi.*` 直接调用 `invoke()`，所有 IPC 调用通过 `src/shared/lib/tauri.ts`
- **状态管理**：不得绕过 Zustand Store 管理业务数据，组件通过 selectors 取数据、actions 改数据；业务数据必须按事务规则在磁盘写入成功后再 commit store
- **类型来源**：所有 TypeScript 类型定义在 `src/shared/types/index.ts`，不得在组件内定义内联类型
- **样式规则**：Tailwind class 应映射到 CSS Variable Token，不硬编码颜色、间距、圆角值
- **MVP 边界**：超出 MVP 范围的功能应先更新本文件，再开始编码
- **文件格式**：SKILL.md 的序列化/反序列化通过 `skillParser.ts`，不得在其他位置实现

## 架构变更记录

| 日期 | 变更内容 | 原因 | 触发方 |
|---|---|---|---|
| 2026-06-13 | 初版架构方案 | P3-2 试跑（Desktop Register） | 前端架构设计 Skill |
