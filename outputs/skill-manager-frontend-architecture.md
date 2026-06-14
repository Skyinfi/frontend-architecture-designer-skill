---
# 前端架构方案摘要（供下游 skill / agent 快速读取）
framework: React
register: Desktop
complexity_level: 2
decisive_dimension: 业务复杂度
component_library: 无（现有全局 CSS + CSS Variables 方案；建议演进为 shadcn/ui + Tailwind）
styling: CSS Variables（全局）；建议迁移为 CSS Modules 或 Tailwind
state_management: Zustand 5 (persist middleware)
architecture_focus: App.tsx 解耦为 feature-based 结构 + 全局 CSS 拆分 + Rust 后端模块化
mvp_scope: 保持现有功能不变，增量改进目录结构和样式隔离
created: 2026-06-13
updated: 2026-06-13
---

<!-- 下游 coding agent 使用指引：
     1. 先读 YAML frontmatter 了解全局决策
     2. 再读"架构约束"区了解不可违反的底线
     3. 按章节顺序理解架构方案
     4. 编码前检查 MVP 边界
-->

# Skill Manager 前端架构设计方案

## 1. 项目理解摘要

**项目名称**：Skill Manager
**项目类型**：桌面应用（Tauri 2 桌面端）
**核心用户**：使用 AI Agent（Codex / Claude Code / Cursor 等）进行开发的开发者
**核心场景**：管理多个 AI Agent 的 Skill 配置文件——浏览、新建、编辑、删除、导入 Skill，管理 Agent 列表及其关联的 Skill
**前端形态**：工具型（以数据管理和交互操作为主）
**技术约束**：已有 Tauri 2 + React 19 + TypeScript + Zustand 5 代码库，为存量项目增量改造

---

## 2. 复杂度判定

| 维度 | 判定 | 说明 |
|---|---|---|
| 团队规模 | Level 1 | 1 人开发 |
| 生命周期 | Level 2 | 持续迭代的个人工具，预期 > 3 个月 |
| 业务复杂度 | Level 2 | 多实体（Agent / Skill / Param）的 CRUD + 关联关系 + 文件系统读写 + Skill 解析引擎 |

**最终 Level**：2（决定性维度：业务复杂度，架构侧重：领域拆分 + 状态架构 + 数据流设计）

---

## 3. 推荐技术栈

| 维度 | 选择 | 理由 |
|---|---|---|
| 框架 | **React 19**（已采用） | 项目已使用 React 19.1，保持不变 |
| 语言 | **TypeScript**（已采用） | 严格模式已开启，保持不变 |
| 构建工具 | **Vite 7**（已采用） | 项目已使用 Vite 7，Tauri 2 官方支持 |
| 样式方案 | **CSS Variables + CSS Modules**（渐进迁移） | 现有 tokens.css（54 个语义变量）保留，新增样式用 CSS Modules 做作用域隔离 |
| 组件库 | **暂不引入**（后续可考虑 shadcn/ui） | 当前规模无需重型组件库；如后续组件增多，shadcn/ui 是 Desktop Register 的首选 |
| 状态管理 | **Zustand 5**（已采用） | 已使用 Zustand 5 + persist middleware，保持不变 |
| 数据获取 | **Tauri invoke**（已采用） | 桌面应用通过 Tauri IPC 调用 Rust 后端，不涉及 HTTP API |
| 图标库 | **lucide-react**（建议引入） | 轻量、tree-shakable、AI 友好，与 shadcn/ui 生态一致 |
| 路由 | **暂不需要** | 单页应用，无多路由场景；如后续扩展可引入 React Router |

---

## 4. UI 组件库选择与理由

### 选择：暂不引入主组件库

### 选择理由

- **与 Register 匹配度**：Desktop Register 推荐轻量方案。当前项目已有完整的通用组件（Button / Badge / Modal / Input / ParamEditor / EmptyState / ErrorDisplay / LoadingSpinner），足以支撑现有功能
- **AI 友好度**：现有组件均为自研，代码可控、类型完整、命名语义明确
- **生态成熟度**：不引入额外依赖，降低维护负担
- **定制能力**：自研组件完全可控，通过 tokens.css 统一视觉风格
- **Bundle 影响**：零额外引入

### 后续演进路径

| 阶段 | 触发条件 | 方案 |
|---|---|---|
| 当前 | 组件数 < 15，功能稳定 | 保持自研 |
| 增长期 | 组件数 > 20，需要复杂表格/表单/命令面板 | 引入 shadcn/ui + Tailwind |

### 排除方案

| 被排除的库 | 排除理由 |
|---|---|
| Ant Design | 重型组件库，bundle 大，与桌面应用轻量定位不符 |
| MUI | 样式体系（emotion/styled-components）与现有 CSS Variables 冲突 |
| Headless UI | 当前不需要，且现有组件已满足需求 |

---

## 当前结构诊断（存量项目工作流 B）

> 本节为存量项目追加内容，记录现有状态诊断和增量改造方案。

### 现有技术栈

| 维度 | 当前方案 | 版本 |
|---|---|---|
| 框架 | React | 19.1 |
| 组件库 | 自研（无第三方组件库） | — |
| 样式方案 | 全局 CSS + CSS Variables（tokens.css 54 个变量） | — |
| 状态管理 | Zustand + persist middleware | 5.x |
| 构建工具 | Vite + Tauri 2 | Vite 7 |
| 桌面框架 | Tauri 2（插件：fs, dialog, opener） | 2.x |
| 后端语言 | Rust（monolithic lib.rs ~800 行） | — |
| 测试（前端） | 无 | — |
| 测试（后端） | Rust 单元测试（7 个函数） | — |

### 问题分类

| 类别 | 问题 | 严重度 |
|---|---|---|
| 高风险 | **App.tsx 单体文件**（~213 行）：抽取 ~45 个 Zustand selector，通过 props drilling 向所有子组件传递状态和回调 | 🔴 |
| 高风险 | **无 CSS 作用域隔离**：所有样式在 App.css 中，使用泛化类名（.card, .panel, .hero），无 CSS Modules，存在全局污染风险 | 🔴 |
| 高风险 | **Rust 后端单体**：lib.rs ~800 行，未拆分为 commands/skill.rs 和 commands/agent.rs，维护和测试困难 | 🔴 |
| 低风险 | **未使用的 greet 命令**：脚手架残留，应清理 | 🟡 |
| 低风险 | **syncSkill 与 upsertSkill 函数体完全相同**：重复代码 | 🟡 |
| 低风险 | **前端无测试基础设施**：无 Vitest / Playwright 配置 | 🟡 |
| 低风险 | **store 中部分方法静默吞错**：catch 空块或仅 console.error，错误不可见 | 🟡 |
| 优势保留 | **严格的 TypeScript 配置**：tsconfig 严格模式，完整类型定义 | ✅ |
| 优势保留 | **竞态条件保护**：store 中使用 syncSeq 计数器防止异步竞态 | ✅ |
| 优势保留 | **Error Boundary**：main.tsx 中已设置错误边界 | ✅ |
| 优势保留 | **优雅的 Tauri 检测**：useTauriReady hook 处理 Web/桌面双环境 | ✅ |
| 优势保留 | **Seed Data 回退**：Tauri 不可用时使用种子数据，确保开发和演示可用 | ✅ |
| 优势保留 | **54 个语义 CSS 变量**：tokens.css 建立了完善的 Design Token 体系 | ✅ |
| 优势保留 | **Serde camelCase 重命名**：Rust 侧 serde 配置确保 JS 互操作的命名一致性 | ✅ |
| 优势保留 | **超越设计文档的功能**：Agent 排序、Skill 导入、Param 编辑器、删除确认等 | ✅ |

### 保留项

| 保留内容 | 保留理由 |
|---|---|
| Zustand 5 + persist middleware | 满足当前状态管理需求，无替换必要 |
| tokens.css（54 个语义 CSS 变量） | 已建立完善的 Design Token 体系，作为后续样式演进的基础 |
| useTauriReady hook | 优雅处理 Web/桌面双环境，设计模式正确 |
| syncSeq 竞态保护 | 正确的异步竞态处理方案，不可丢弃 |
| Error Boundary（main.tsx） | 全局错误兜底，应保留并增强 |
| Seed Data 回退机制 | 保证开发体验和可演示性 |
| Rust 单元测试（7 个函数） | 已有的测试覆盖，应保留并扩展 |
| Serde camelCase 重命名策略 | Rust/JS 互操作最佳实践 |
| 所有超越设计文档的功能增强 | Agent 排序、Skill 导入、Param 编辑器等，均为有价值的增量功能 |

### 调整项

| 调整内容 | 预期收益 | 成本 |
|---|---|---|
| **App.tsx 拆分为 feature hooks**：将 ~45 个 selector 和 props drilling 替换为按功能模块的 custom hooks，各组件自行订阅所需状态 | 降低组件耦合度，提升可维护性；消除 App.tsx 的"上帝组件"反模式 | 中（需逐组件重构） |
| **CSS Modules 渐进迁移**：新组件使用 CSS Modules，现有组件逐步迁移 | 消除全局样式污染风险，实现样式作用域隔离 | 低（逐文件迁移，无破坏性） |
| **App.css 拆分为组件级样式文件**：按组件维度拆分，配合 CSS Modules | 降低单文件复杂度，提升样式可定位性 | 低（机械性拆分） |
| **Rust lib.rs 模块化**：拆分为 commands/skill.rs、commands/agent.rs、commands/storage.rs | 降低单文件复杂度，提升 Rust 代码可测试性 | 中（需要调整 mod 引用） |
| **清理脚手架残留**：删除 unused greet 命令 | 减少死代码 | 极低 |
| **合并 syncSkill / upsertSkill**：消除重复函数 | 减少维护负担 | 极低 |
| **store 错误处理增强**：catch 块中添加用户可见的错误提示 | 提升调试体验和问题可见性 | 低 |
| **引入 Vitest 测试基础设施**：配置 Vitest，编写 store 和工具函数的单元测试 | 建立测试安全网，支撑后续重构信心 | 中（初始配置 + 首批用例） |

### 迁移步骤

| 步骤 | 操作 | 验证方式 | 风险 | 回滚方案 |
|---|---|---|---|---|
| **M1. 清理脚手架残留** | 删除 Rust 侧 unused `greet` 命令及前端调用 | `cargo build` 通过 + 全功能回归测试 | 极低 | git revert |
| **M2. 合并 syncSkill / upsertSkill** | 保留 `upsertSkill`，将 `syncSkill` 的调用点改为引用 `upsertSkill` | 全局搜索 `syncSkill` 无残留 + Skill 增删改功能正常 | 极低 | git revert |
| **M3. store 错误处理增强** | 在 catch 块中添加 Zustand `setError` 调用或 Toast 通知 | 故意触发错误场景（如读写不存在的路径），确认错误可见 | 低 | 移除新增的错误通知代码 |
| **M4. 引入 CSS Modules 基础设施** | 配置 Vite CSS Modules（Vite 原生支持，零配置），新建一个组件的 `.module.css` 作为试点 | 试点组件样式正常渲染，无全局泄漏 | 极低 | 删除 `.module.css` 文件，恢复原引用 |
| **M5. App.css 拆分**：common 组件 | 将 common/ 下组件的样式从 App.css 提取为各组件的 `.module.css` | 每个组件样式独立，App.css 行数减少 | 低 | 将样式合并回 App.css |
| **M6. App.css 拆分**：业务组件 | 将 AgentSidebar / SkillGrid / SkillDetail / HeroPanel / ToolbarPanel / StoragePanel 的样式提取为 `.module.css` | 所有业务组件样式正常，无类名冲突 | 低 | 同上 |
| **M7. App.tsx 解耦 — 提取 feature hooks** | 新建 `hooks/useAgentManager.ts` 和 `hooks/useSkillManager.ts`，从 App.tsx 抽取各自的 selector 和 action 调用；子组件改为直接使用对应 hook 而非接收 props | App.tsx 行数 < 80 行，各子组件自行订阅状态，全功能回归通过 | 中（涉及多个组件改动） | 恢复 props drilling 方式 |
| **M8. App.tsx 解耦 — 应用壳提取** | 将 App.tsx 的布局结构提取为 `components/AppShell/index.tsx`，App.tsx 只做 Provider 组合 | App.tsx < 30 行，AppShell 组件独立可测试 | 低 | 将布局合并回 App.tsx |
| **M9. Rust lib.rs 模块化** | 创建 `src-tauri/src/commands/` 目录，拆分为 `skill.rs`、`agent.rs`、`storage.rs`，lib.rs 只做 mod 声明和注册 | `cargo build` + `cargo test` 通过 + 前端全功能回归 | 中（涉及 Rust 模块重构） | 将模块合并回 lib.rs |
| **M10. 引入 Vitest** | 安装 vitest + @testing-library/react，配置 vite，编写 store slice 和 skillParser 的首批测试 | `npm run test` 通过，覆盖核心解析和状态逻辑 | 低 | 移除 vitest 配置和测试文件 |
| **M11. 目录结构演进** | 创建 `features/` 目录骨架，将现有组件按功能归入 `features/skill/` 和 `features/agent/` | 构建通过，全功能回归，目录结构清晰 | 中（大量文件移动） | 恢复原始目录结构 |

> **迁移原则**：每步独立可验证、可回滚。建议按 M1→M11 顺序执行，高风险步骤（M7、M9、M11）安排在低风险步骤之后，确保基础改进先到位。

### 技术债记录

| 技术债 | 暂不处理理由 | 预计清理时机 |
|---|---|---|
| 未引入路由库 | 当前为单页应用，无多路由需求 | 功能扩展需多页面时引入 React Router |
| 未引入 TanStack Query | 数据来源于 Tauri IPC 而非 HTTP API，Zustand + invoke 模式足够 | 如引入文件监听或实时同步，可能需要 |
| 未引入 shadcn/ui + Tailwind | 当前组件数 < 15，自研组件足够 | 组件数 > 20 或需要复杂 UI 组件时 |
| 无 E2E 测试 | Vitest 单元测试优先建立 | MVP 稳定后引入 Playwright |
| 无暗色模式 | 当前仅 light mode，功能稳定 | 视觉设计阶段一并处理 |
| 无国际化 | 用户群体为中文开发者 | 有国际用户需求时 |

---

## 5. 目录结构方案

### 当前结构（扁平，按类型组织）

```
skill-manager/
├── src/
│   ├── components/
│   │   ├── AgentSidebar/index.tsx
│   │   ├── SkillGrid/ (index.tsx, SkillCard.tsx)
│   │   ├── SkillDetail/index.tsx
│   │   ├── HeroPanel.tsx          ← 平铺，无目录
│   │   ├── ToolbarPanel.tsx       ← 平铺，无目录
│   │   ├── StoragePanel.tsx       ← 平铺，无目录
│   │   ├── modals/ (NewSkillModal, EditSkillModal, AgentEditorModal, DeleteConfirmModal)
│   │   └── common/ (Button, Badge, Modal, Input, ParamEditor, EmptyState, ErrorDisplay, LoadingSpinner)
│   ├── hooks/ (useSkills.ts, useTauriReady.ts)
│   ├── store/index.ts
│   ├── types/index.ts
│   ├── lib/ (tauri.ts, skillParser.ts, skillParam.ts, constants.ts)
│   ├── styles/ (tokens.css, globals.css)
│   ├── App.tsx                    ← ~213 行单体
│   ├── App.css                    ← 全局样式单文件
│   └── main.tsx
├── src-tauri/
│   └── src/
│       └── lib.rs                 ← ~800 行单体
└── ...
```

### 目标结构（Level 2：Feature-based，渐进迁移后）

```
skill-manager/
├── src/
│   ├── features/                     # 按功能模块组织
│   │   ├── skill/
│   │   │   ├── components/
│   │   │   │   ├── SkillGrid/
│   │   │   │   │   ├── index.tsx
│   │   │   │   │   ├── SkillCard.tsx
│   │   │   │   │   └── SkillGrid.module.css
│   │   │   │   └── SkillDetail/
│   │   │   │       ├── index.tsx
│   │   │   │       └── SkillDetail.module.css
│   │   │   ├── modals/
│   │   │   │   ├── NewSkillModal.tsx
│   │   │   │   ├── EditSkillModal.tsx
│   │   │   │   └── DeleteConfirmModal.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useSkillManager.ts
│   │   │   └── index.ts
│   │   └── agent/
│   │       ├── components/
│   │       │   ├── AgentSidebar/
│   │       │   │   ├── index.tsx
│   │       │   │   └── AgentSidebar.module.css
│   │       │   └── AgentEditorModal.tsx
│   │       ├── hooks/
│   │       │   └── useAgentManager.ts
│   │       └── index.ts
│   ├── shared/                       # 跨功能共享
│   │   ├── components/               # 通用 UI 组件
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   └── Button.module.css
│   │   │   ├── Badge/
│   │   │   ├── Modal/
│   │   │   ├── Input/
│   │   │   ├── ParamEditor/
│   │   │   ├── EmptyState/
│   │   │   ├── ErrorDisplay/
│   │   │   └── LoadingSpinner/
│   │   ├── hooks/
│   │   │   └── useTauriReady.ts
│   │   ├── lib/
│   │   │   ├── tauri.ts
│   │   │   ├── skillParser.ts
│   │   │   ├── skillParam.ts
│   │   │   └── constants.ts
│   │   └── types/
│   │       └── index.ts
│   ├── app/                          # 应用壳
│   │   ├── AppShell.tsx              # 全局布局（从 App.tsx 提取）
│   │   ├── HeroPanel.tsx             # 应用级面板
│   │   ├── ToolbarPanel.tsx
│   │   └── StoragePanel.tsx
│   ├── store/
│   │   └── index.ts                  # Zustand store（保持集中）
│   ├── styles/
│   │   ├── tokens.css                # Design Token（保留）
│   │   └── globals.css               # 全局基础样式（保留）
│   ├── App.tsx                       # 精简为 < 30 行：Provider + AppShell
│   └── main.tsx                      # 入口 + Error Boundary
├── src-tauri/
│   └── src/
│       ├── commands/                 # Rust 后端模块化
│       │   ├── mod.rs
│       │   ├── skill.rs
│       │   ├── agent.rs
│       │   └── storage.rs
│       ├── lib.rs                    # 精简为 mod 声明 + 注册
│       └── main.rs
├── docs/
│   └── frontend-architecture.md
├── tests/                            # Vitest 测试
│   ├── store/
│   └── lib/
├── public/
├── package.json
├── vite.config.ts
├── vitest.config.ts
└── tsconfig.json
```

### 目录职责说明

| 目录 | 职责 | 放什么 | 不放什么 |
|---|---|---|---|
| `src/features/skill/` | Skill 功能模块 | Skill 相关组件、hooks、弹窗 | Agent 相关逻辑 |
| `src/features/agent/` | Agent 功能模块 | Agent 相关组件、hooks、弹窗 | Skill 相关逻辑 |
| `src/shared/components/` | 通用 UI 组件 | Button、Modal 等纯展示组件 | 业务逻辑、API 调用 |
| `src/shared/hooks/` | 通用 hooks | useTauriReady 等跨功能 hooks | 业务专用 hooks |
| `src/shared/lib/` | 工具函数 | skillParser、tauri 封装、常量 | 依赖业务上下文的逻辑 |
| `src/app/` | 应用壳 | 全局布局、应用级面板（Hero/Toolbar/Storage） | 功能模块的组件 |
| `src/store/` | 全局状态 | Zustand store 定义 | 组件代码 |
| `src/styles/` | 全局样式 | Design Token、全局 CSS | 组件级 scoped 样式 |
| `src-tauri/src/commands/` | Rust 命令模块 | 按领域拆分的 Tauri commands | 前端代码 |

---

## 6. 模块边界与领域拆分

### 领域划分

| 域类型 | 模块 | 职责 | 依赖 |
|---|---|---|---|
| 核心域 | **skill** | Skill 的 CRUD、解析、参数编辑、导入 | shared（types, lib, components） |
| 核心域 | **agent** | Agent 列表管理、排序、Skill 关联 | shared（types, lib, components） |
| 支撑域 | **storage** | 文件系统读写、存储路径管理 | shared（tauri.ts） |
| 通用域 | **shared** | 通用 UI 组件、hooks、工具函数、类型 | 无业务依赖 |

### 模块间依赖规则

```
skill / agent → shared
（核心域只依赖通用域，核心域之间通过 store 解耦，不直接 import）
```

| 规则 | 说明 |
|---|---|
| `features/skill/` 不得 import `features/agent/` 的内部文件 | 通过 Zustand store 共享数据 |
| `features/agent/` 不得 import `features/skill/` 的内部文件 | 同上 |
| `shared/` 不得 import `features/` 的任何文件 | 通用层无业务感知 |
| `app/` 可组合 `features/` 和 `shared/` | 应用壳是组装层 |

---

## 7. 核心组件清单

### 应用壳组件

| 组件 | 职责 | 优先级 | 当前状态 |
|---|---|---|---|
| AppShell | 全局布局框架：侧边栏 + 主内容区 + Hero + Toolbar | P0 | 待从 App.tsx 提取 |
| HeroPanel | 顶部概览面板（存储统计等） | P1 | 已有，需归入 app/ |
| ToolbarPanel | 工具栏（新建、导入等操作入口） | P1 | 已有，需归入 app/ |
| StoragePanel | 存储路径配置面板 | P2 | 已有，需归入 app/ |

### 业务组件

| 组件 | 职责 | 所属模块 | 优先级 | 当前状态 |
|---|---|---|---|---|
| AgentSidebar | Agent 列表侧边栏，支持排序 | features/agent | P0 | 已有 |
| AgentEditorModal | Agent 编辑弹窗 | features/agent | P0 | 已有 |
| SkillGrid | Skill 卡片网格展示 | features/skill | P0 | 已有 |
| SkillCard | 单个 Skill 卡片 | features/skill | P0 | 已有 |
| SkillDetail | Skill 详情展示 | features/skill | P0 | 已有 |
| NewSkillModal | 新建 Skill 弹窗 | features/skill | P0 | 已有 |
| EditSkillModal | 编辑 Skill 弹窗 | features/skill | P0 | 已有 |
| DeleteConfirmModal | 删除确认弹窗 | features/skill | P0 | 已有 |

### 通用组件

| 组件 | 职责 | 优先级 | 当前状态 |
|---|---|---|---|
| Button | 通用按钮 | P0 | 已有 |
| Badge | 标签/徽章 | P0 | 已有 |
| Modal | 通用弹窗容器 | P0 | 已有 |
| Input | 通用输入框 | P0 | 已有 |
| ParamEditor | 参数编辑器 | P0 | 已有 |
| EmptyState | 空状态展示 | P1 | 已有 |
| ErrorDisplay | 错误展示 | P1 | 已有 |
| LoadingSpinner | 加载状态 | P1 | 已有 |

### 组件封装原则

1. **先封装稳定重复的结构**：现有 common 组件已封装良好，保持不变
2. **不为了"看起来高级"而提前抽象**：当前无新的抽象需求
3. **业务组件和纯 UI 组件分开**：迁移后 shared/components/ 纯 UI，features/ 含业务逻辑
4. **组件命名遵循 AI 友好规则**：可搜索（AgentSidebar 而非 Sidebar）、语义明确、遵循约定、模式稳定

---

## 8. Design Token 初稿

### 现有 Token 体系（保留并增强）

项目已建立完善的 Design Token 体系（`src/styles/tokens.css`，54 个语义变量），涵盖 color、spacing、radius、shadow、typography、z-index 等维度。以下为现有 Token 的核心结构：

### Spacing

```css
:root {
  /* 现有 — 保持不变 */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-12: 3rem;     /* 48px */
}
```

### Color（现有结构，保持不变）

```css
:root {
  /* 背景 */
  --bg-primary: ;
  --bg-secondary: ;
  --bg-elevated: ;

  /* 文字 */
  --text-primary: ;
  --text-secondary: ;
  --text-muted: ;

  /* 功能色 */
  --color-primary: ;
  --color-danger: ;
  --color-success: ;
  --color-warning: ;

  /* 边框 */
  --border-default: ;
  --border-strong: ;
}
```

### Typography（现有结构，保持不变）

```css
:root {
  --font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', ...;
  --font-mono: 'SF Mono', 'Fira Code', ...;

  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
}
```

### Layout（建议新增）

```css
:root {
  /* 建议新增 — 当前硬编码在 App.css 中 */
  --sidebar-width: 280px;
  --sidebar-collapsed-width: 0px;   /* 当前无折叠功能 */
  --hero-height: auto;
  --content-max-width: 100%;        /* 桌面应用全宽 */
  --modal-max-width: 600px;
  --modal-max-height: 80vh;
}
```

### Z-Index（现有，保持不变）

```css
:root {
  --z-base: 0;
  --z-sidebar: 10;
  --z-overlay: 100;
  --z-modal: 200;
  --z-tooltip: 300;
}
```

> **Token 演进策略**：现有 54 个变量保持不变，新增 Token 仅在发现硬编码值时提取。不预先发明未使用的 Token。

---

## 9. 数据与状态架构

### 状态分类

| 数据类型 | 方案 | 说明 |
|---|---|---|
| 服务端数据（server-state） | **Zustand + Tauri invoke** | 通过 Tauri IPC 读取本地文件系统的 Skill/Agent 数据，store 中缓存 + persist |
| 全局 UI 状态（client-state） | **Zustand** | 当前选中 Agent、弹窗开关、加载状态、错误状态 |
| 组件局部状态 | **React useState** | 表单输入值、搜索关键词等瞬时状态 |

### 数据获取层

```
数据流向：
Rust 后端（文件系统读写 + Skill 解析）
  ↕ Tauri IPC (invoke)
Zustand Store（缓存 + persist + 竞态保护）
  ↕ custom hooks (useSkillManager / useAgentManager)
React 组件（订阅 store 切片）
```

- **请求封装入口**：`shared/lib/tauri.ts`（已有，封装 invoke 调用）
- **错误处理策略**：store 层 catch 错误并设置 error state（M3 步骤增强）
- **缓存策略**：Zustand persist middleware 将数据持久化到 localStorage，页面刷新不丢失
- **竞态保护**：syncSeq 计数器机制（已有，保留）

### 全局状态边界

| 放全局（Zustand Store） | 不放全局 |
|---|---|
| Agent 列表及排序 | 弹窗开关（改为组件局部 state） |
| Skill 列表及当前选中 | 表单输入值 |
| 当前选中 Agent ID | 搜索关键词（组件局部） |
| 全局加载状态 | 单次操作的 loading |
| 全局错误状态 | 列表滚动位置 |
| 存储路径配置 | |

### Store 结构优化建议

当前 store（`src/store/index.ts`）采用单 store 集中管理，对当前规模是合理的。建议的改进方向：

1. **提取 feature hooks**（M7）：组件通过 `useSkillManager` / `useAgentManager` 访问 store，而非直接在 App.tsx 中抽取 45 个 selector
2. **错误状态增强**（M3）：catch 块中设置用户可见的错误信息
3. **合并重复方法**（M2）：syncSkill 合并到 upsertSkill

---

## 10. 主题与国际化预留

> 本项目复杂度为 Level 2，此项暂不展开。

**当前状态**：项目使用 Light Mode，通过 CSS Variables 管理所有颜色和间距。tokens.css 中已建立语义化颜色命名（如 `--bg-primary`、`--text-secondary`），为后续暗色模式切换预留了基础结构——只需添加 `[data-theme="dark"]` 选择器覆盖变量值即可。

**国际化**：项目用户为中文开发者，当前无国际化需求。

---

## 11. MVP 第一阶段范围

### 必须完成的改造（按迁移步骤 M1-M11 顺序）

| 改造项 | 核心目标 | 优先级 |
|---|---|---|
| M1: 清理脚手架残留 | 删除死代码 | P0 |
| M2: 合并重复函数 | 消除 syncSkill/upsertSkill 重复 | P0 |
| M3: store 错误处理增强 | 错误可见性 | P0 |
| M4-M6: CSS Modules 渐进迁移 | 样式作用域隔离 | P0 |
| M7-M8: App.tsx 解耦 | 消除单体组件反模式 | P1 |
| M9: Rust 后端模块化 | 提升后端可维护性 | P1 |
| M10: Vitest 测试基础设施 | 建立测试安全网 | P2 |
| M11: 目录结构演进 | feature-based 结构 | P2 |

### 必须保持的功能（不可退化）

| 功能 | 说明 |
|---|---|
| Skill CRUD | 新建、编辑、删除、查看 |
| Agent 管理 | 列表、排序、编辑 |
| Skill 导入 | 从外部文件导入 |
| 参数编辑器 | ParamEditor 组件 |
| 删除确认 | DeleteConfirmModal |
| Seed Data 回退 | Web 环境降级 |
| Tauri 文件系统操作 | 读写 Skill 文件 |

### 明确延后的能力

| 能力 | 延后理由 | 预计引入阶段 |
|---|---|---|
| 暗色模式 | 功能优先，视觉其次 | 视觉设计阶段 |
| 多路由 | 当前单页足够 | 需要设置页/帮助页时 |
| 组件库引入 | 自研组件足够 | 组件数 > 20 时 |
| E2E 测试 | 单元测试优先 | 功能稳定后 |
| 命令面板（Cmd+K） | 桌面应用增强 | P3 |
| 文件监听实时同步 | 当前手动刷新足够 | 需要多窗口协作时 |

### 可接受的技术债

| 技术债 | 接受理由 | 清理时机 |
|---|---|---|
| 弹窗开关放在 Zustand store | 功能稳定，重构收益低于风险 | M7 解耦时一并处理 |
| 部分 CSS 仍使用全局类名 | 渐进迁移中，不影响功能 | M4-M6 完成后消除 |
| Store 单文件结构 | 当前规模可管理 | 业务复杂度增长时拆分 |

### 不可接受的技术债

- **App.tsx 单体不经任何拆分直接继续堆积功能**：213 行已是临界点，继续增长将显著降低可维护性
- **全局 CSS 无作用域地继续新增样式**：新样式必须使用 CSS Modules

---

## 12. 后续编码建议

### 编码前

1. 按 M1-M3 完成低风险清理，建立干净的工作基础
2. 配置 CSS Modules（M4），确定新样式编写规范
3. 按本方案的目标目录结构创建 `features/`、`shared/`、`app/` 骨架目录
4. 建立 Vitest 测试基础设施（M10），确保后续重构有测试保护

### 编码中

1. **新增组件遵循分层原则**：UI 组件 → `shared/components/`，业务组件 → `features/[module]/components/`
2. **新样式使用 CSS Modules**：文件命名为 `ComponentName.module.css`
3. **状态访问通过 feature hooks**：使用 `useSkillManager` / `useAgentManager` 而非直接访问 store
4. **样式使用 Token**：不硬编码颜色和间距，引用 `tokens.css` 中的 CSS 变量
5. **命名遵循 AI 友好规则**：可搜索、语义明确、遵循约定、模式稳定
6. **错误处理不可静默吞掉**：catch 块中必须设置用户可见的错误状态
7. **Rust 新命令按领域拆分**：新增命令放入对应的 `commands/[domain].rs`，不追加到 lib.rs

### 编码后

1. 运行 `npm run test` 确认测试通过
2. 运行 `cargo test` 确认 Rust 测试通过
3. 运行 `cargo build` 确认 Tauri 构建通过
4. 全功能手动回归（Agent CRUD + Skill CRUD + 导入 + 删除确认）
5. 架构变更记录在下方"架构变更记录"区

### 衔接建议

| 后续步骤 | 推荐方式 | Fallback |
|---|---|---|
| 视觉设计 | Impeccable craft 或 shape | 本方案中的 Token 规范 + 现有视觉风格 |
| 编码流程 | 逐步骤执行 M1-M11 | 按优先级选择关键步骤执行 |
| 质量审计 | Impeccable audit | decision-checklist 自检 |
| 功能扩展 | 遵循 feature-based 结构 | 在现有结构中新增 |

### 可访问性与质量基线

- **可访问性（a11y）**：组件使用语义化 HTML + 基础 aria 属性；弹窗使用 focus trap；键盘可操作
- **性能 / bundle**：React 19 + Vite 7 默认优化，无需额外配置；关注 Tauri 打包体积
- **测试策略**：核心 store 方法 + skillParser + skillParam 单元测试；后续扩展 E2E
- **错误监控**：Error Boundary（已有）+ store error state（M3 增强）
- **CI/CD**：当前手动构建可部署，后续可加 GitHub Actions（cargo build + npm run test）

---

## 架构决策记录

| 日期 | 决策 | 理由 | 替代方案 | 触发方 |
|---|---|---|---|---|
| 2026-06-13 | 保持 Zustand 而非迁移到 TanStack Query | 数据来源于 Tauri IPC 非 HTTP API，Zustand + invoke 模式足够 | TanStack Query | 架构诊断 |
| 2026-06-13 | 使用 CSS Modules 而非引入 Tailwind | 渐进迁移成本最低，与现有 CSS Variables 体系兼容 | Tailwind CSS | 架构诊断 |
| 2026-06-13 | 暂不引入 shadcn/ui | 当前 15 个自研组件足够，引入增加学习成本 | shadcn/ui | 架构诊断 |
| 2026-06-13 | feature hooks 替代 props drilling | 消除 App.tsx 单体问题，同时避免 store 过度暴露 | Context / Jotai | 架构诊断 |
| 2026-06-13 | Rust lib.rs 按领域拆分为 commands 模块 | 800 行临界点，拆分后可维护性和可测试性提升 | 保持单文件 | 架构诊断 |

---

## 架构约束（下游 skill 应遵守）

- **技术栈**：不得更改 React 19 / Zustand 5 / Vite 7 / Tauri 2 技术栈
- **目录结构**：新增目录应遵循 feature-based 分层规则（features/ → shared/ → app/）
- **组件库**：不得引入与现有自研组件冲突的第三方组件库（需经架构决策记录审批）
- **状态管理**：不得绕过 feature hooks 直接在 App.tsx 中提取大量 selector
- **样式方案**：新增样式必须使用 CSS Modules（.module.css），不得在 App.css 中追加全局样式
- **命名规范**：组件 PascalCase、hooks camelCase use 前缀、文件名与组件名一致
- **MVP 边界**：超出本方案列出的功能范围，应先更新本文件的"架构变更记录"
- **Rust 后端**：新增命令必须放入 `commands/[domain].rs`，不得追加到 lib.rs
- **Token 使用**：不得硬编码颜色值和间距，必须引用 tokens.css 中的 CSS 变量
- **错误处理**：不得静默吞掉错误（空 catch 块），必须设置用户可见的错误状态

## 架构变更记录

| 日期 | 变更内容 | 原因 | 触发方 |
|---|---|---|---|
| 2026-06-13 | 初始架构方案创建 | 存量项目架构诊断与增量改造规划 | 架构设计 |
