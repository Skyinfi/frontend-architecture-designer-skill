---
# 前端架构方案摘要（供下游 skill / agent 快速读取）
framework: Astro
register: Content
complexity_level: 1
decisive_dimension: 团队规模
component_library: 无（Astro 原生组件 + CSS 变量）
styling: CSS Variables
state_management: 未使用全局状态
architecture_focus: 保持轻量，Astro SSG 构建管线优先，不过度组件化
mvp_scope: MD 笔记浏览 + HTML iframe 渲染 + Docker 部署
created: 2026-06-13
updated: 2026-06-13
---

<!-- 下游 coding agent 使用指引：
     1. 先读 YAML frontmatter 了解全局决策
     2. 再读"架构约束"区了解不可违反的底线
     3. 按章节顺序理解架构方案
     4. 编码前检查 MVP 边界
-->

# 知识库展示平台 前端架构设计方案

## 1. 项目理解摘要

**项目名称**：知识库展示平台（Knowledge Base Display Platform）
**项目类型**：知识库 / 静态内容站
**核心用户**：开发者本人 + 外部访问者（通过公网浏览技术知识笔记）
**核心场景**：将本地 Obsidian 知识库（约 151 篇 Markdown + 26 篇 HTML 笔记）转换为可公开访问的静态知识网站。用户按栏目浏览笔记、通过搜索定位内容、阅读 Markdown 渲染页面或 HTML iframe 嵌入页面。
**前端形态**：展示型（纯静态，SSG 零 JS 运行时）
**技术约束**：Astro 4.x + TypeScript（已锁定），Docker + Nginx 部署（已锁定），无后端服务，无运行时状态

---

## 2. 复杂度判定

| 维度 | 判定 | 说明 |
|---|---|---|
| 团队规模 | Level 1 | 1 人维护 |
| 生命周期 | Level 1 | 个人长期项目，无商业化压力 |
| 业务复杂度 | Level 1 | 纯内容展示，无交互逻辑，无用户系统 |

**最终 Level**：1（决定性维度：团队规模，架构侧重：保持轻量，Astro SSG 构建管线优先，不过度组件化）

---

## 3. 推荐技术栈

| 维度 | 选择 | 理由 |
|---|---|---|
| 框架 | Astro 4.x SSG | 零 JS 默认输出，Content Collections 原生支持，构建时数据注入，完美适配静态内容站 |
| 语言 | TypeScript | 类型安全约束 Content Collection schema 和构建脚本 |
| 构建工具 | Astro 内置（Vite） | 无需额外配置 |
| 样式方案 | CSS Variables + Astro Scoped CSS | 无需 Tailwind 开销，Astro 组件级 scoped style 天然隔离 |
| 组件库 | 无 | Astro 原生组件足够，Content Register 不需要重量级 UI 库 |
| 状态管理 | 无 | 纯静态站点，零运行时状态 |
| 数据获取 | 构建时（Content Collections + cheerio + prebuild 脚本） | 所有数据在 build 阶段完成注入，无运行时请求 |
| 图标库 | 内联 SVG / Lucide（按需） | 极少量图标需求，不引入完整图标库 |
| 路由 | Astro 文件系统路由 | `src/pages/` 目录即路由，零配置 |

---

## 4. UI 组件库选择与理由

### 选择：无（Astro 原生组件 + CSS 变量）

### 选择理由

本项目为 Content Register 的 Level 1 纯静态展示站。Astro 的组件模型（`.astro` 文件）本身就是原生 HTML 模板 + scoped CSS，无需引入任何第三方 UI 组件库。所有 UI 元素（卡片、侧边栏、标签列表、徽章）均为轻量 Astro 组件，样式通过 CSS 变量控制。若未来个别交互场景需要 client-side islands（如搜索框增强），可按需引入最小化的 Radix / shadcn 组件，但 MVP 阶段不预置。

---

## 5. 目录结构方案

### Level 1：扁平 Astro SSG 结构

```
astro-vault/
├── scripts/
│   ├── sync-vault.ts           # 统一同步 MD + HTML（prebuild 入口）
│   └── extract-html-meta.ts    # 从 HTML DOM 提取元数据（cheerio）
├── src/
│   ├── content/
│   │   └── config.ts           # Content Collection schema 定义
│   │   └── notes/              # 构建时复制自知识库的 .md 文件
│   ├── data/
│   │   └── html-notes.json     # 构建生成的 HTML 笔记注册表
│   ├── types/
│   │   └── note.ts             # VaultNote 统一接口
│   ├── pages/
│   │   ├── index.astro         # 首页
│   │   ├── [section]/
│   │   │   └── index.astro     # 栏目列表页
│   │   ├── notes/
│   │   │   └── [...slug].astro # 笔记详情页（MD/HTML 分支渲染）
│   │   └── search.astro        # 搜索页
│   ├── layouts/
│   │   ├── BaseLayout.astro    # 全局基础布局
│   │   ├── NoteLayout.astro    # Markdown 三栏布局
│   │   └── HtmlNoteLayout.astro # HTML iframe 外壳布局
│   ├── components/
│   │   ├── NoteCard.astro      # 笔记卡片
│   │   ├── FormatBadge.astro   # 格式标识（MD / HTML）
│   │   ├── TagList.astro       # 标签列表
│   │   ├── Sidebar.astro       # 侧边导航
│   │   └── Search.astro        # 搜索组件
│   └── utils/
│       └── resolveNoteLink.ts  # Wiki 链接解析
├── public/
│   └── html-notes/             # 构建时复制的原始 .html 文件
├── docs/
│   └── frontend-architecture.md
├── Dockerfile
├── nginx.conf
├── astro.config.mjs
└── package.json
```

### 目录职责说明

| 目录 | 职责 | 放什么 | 不放什么 |
|---|---|---|---|
| `scripts/` | 构建前数据准备 | sync-vault.ts, extract-html-meta.ts | 运行时代码 |
| `src/content/` | Astro Content Collections | config.ts, 同步后的 .md 笔记 | HTML 文件、组件 |
| `src/data/` | 构建时生成的数据文件 | html-notes.json 注册表 | 手动维护的数据 |
| `src/types/` | TypeScript 类型定义 | VaultNote 接口 | 运行时逻辑 |
| `src/pages/` | Astro 文件系统路由 | 各页面 .astro 文件 | 组件、布局 |
| `src/layouts/` | 页面布局模板 | BaseLayout, NoteLayout, HtmlNoteLayout | 业务逻辑组件 |
| `src/components/` | 可复用 UI 组件 | NoteCard, Sidebar, Search 等 | 页面级代码 |
| `src/utils/` | 工具函数 | 链接解析等纯函数 | 有副作用的代码 |
| `public/html-notes/` | 原始 HTML 笔记静态资源 | 同步后的 .html 文件 | Markdown 文件 |
| `docs/` | 项目文档 | 本架构方案 | 构建产物 |

---

## 6. 模块边界与领域拆分

本项目复杂度较低，此项暂不展开。

---

## 7. 核心组件清单

### 应用壳组件

| 组件 | 职责 | 优先级 |
|---|---|---|
| BaseLayout | 全局 HTML 骨架（head、meta、CSS 变量、Pagefind 样式注入） | P0 |
| NoteLayout | Markdown 笔记三栏布局（Sidebar + 主内容 + 目录） | P0 |
| HtmlNoteLayout | HTML 笔记 iframe 外壳（全屏 iframe + 返回导航） | P0 |

### 业务组件

| 组件 | 职责 | 所属模块 | 优先级 |
|---|---|---|---|
| NoteCard | 笔记摘要卡片（标题、标签、格式标识、更新时间） | 栏目列表页 | P0 |
| FormatBadge | 格式标识徽章（MD / HTML），帮助用户区分笔记类型 | NoteCard / 详情页 | P0 |
| Sidebar | 侧边导航（栏目列表 + 当前高亮） | NoteLayout | P0 |
| Search | Pagefind 全文搜索入口与结果渲染 | 全局 | P1 |
| TagList | 标签列表展示与标签页导航 | 详情页 / 标签页 | P1 |

### 通用组件

本项目为 Level 1 展示型站点，EmptyState / LoadingState / ErrorState 等 P1 通用组件可在后续阶段按需添加。MVP 阶段不提前封装。

### 组件封装原则

1. **构建管线优先**：`sync-vault.ts` 和 `extract-html-meta.ts` 是整个项目的基础，组件开发在其之后
2. **不过度抽象**：5 个栏目、2 种笔记格式，组件数量控制在个位数
3. **Astro 原生优先**：所有组件使用 `.astro` 文件，不引入 framework 组件（React/Vue）
4. **命名语义明确**：组件名直接对应 UI 职责（NoteCard = 笔记卡片，FormatBadge = 格式徽章）

---

## 8. Design Token 初稿

本项目复杂度较低，此项暂不展开。

---

## 9. 数据与状态架构

本项目复杂度较低，此项暂不展开。

---

## 10. 主题与国际化预留

本项目复杂度较低，此项暂不展开。

---

## 11. MVP 第一阶段范围

### 必须完成的页面

| 页面 | 核心功能 | 优先级 |
|---|---|---|
| `/notes/[...slug]` | 笔记详情页——MD 笔记通过 Content Collection 渲染，HTML 笔记通过 iframe 嵌入；根据 frontmatter 的 format 字段分支渲染 | P0 |

### 必须完成的构建管线

| 管线 | 核心功能 | 优先级 |
|---|---|---|
| `scripts/sync-vault.ts` | 统一同步 MD 文件至 `src/content/notes/`、HTML 文件至 `public/html-notes/`、提取元数据生成 `src/data/html-notes.json` | P0 |
| `scripts/extract-html-meta.ts` | 使用 cheerio 从 4 种 HTML 模板中提取 title / tags / section 元数据 | P0 |
| `src/content/config.ts` | Content Collection schema 定义（含 format、section、tags 字段） | P0 |
| Dockerfile + nginx.conf | Docker 镜像构建 + Nginx 静态文件服务配置 | P0 |

### 必须封装的组件

| 组件 | 理由 | 优先级 |
|---|---|---|
| BaseLayout | 全局 HTML 骨架，所有页面的基础容器 | P0 |
| NoteLayout | Markdown 笔记的三栏布局，承载核心阅读体验 | P0 |
| HtmlNoteLayout | HTML 笔记的 iframe 渲染外壳，处理两种笔记格式的差异 | P0 |
| FormatBadge | 区分 MD / HTML 格式，在卡片和详情页中复用 | P0 |
| Sidebar | 栏目导航，笔记详情页的核心导航组件 | P0 |

### 明确延后的能力

| 能力 | 延后理由 | 预计引入阶段 |
|---|---|---|
| 首页 + 栏目列表页 | 核心价值在"阅读笔记"，列表页可延后 | P1 |
| Pagefind 全文搜索 | 需要先完成内容同步和渲染，搜索基于构建产物 | P1 |
| Mermaid 图表渲染 | 仅部分笔记需要，非核心路径 | P1 |
| 响应式布局 | 桌面优先，移动端适配延后 | P1 |
| Wiki 链接转换 | Obsidian `[[wikilink]]` → 相对路径，需笔记间映射 | P0-P1 |
| 标签过滤页面 | 搜索可部分替代，标签聚合延后 | P2 |
| 深色 / 浅色主题切换 | 非核心功能 | P2 |
| 相关笔记推荐 | 需要 tag 相似度算法 | P2 |
| iframe 高度自适应 | 需跨域 postMessage 方案 | P3 |

### 可接受的技术债

| 技术债 | 接受理由 | 清理时机 |
|---|---|---|
| iframe 双滚动条（内外容器同时滚动） | P0 只需能看 HTML 内容，体验优化延后 | P3 |
| extract-html-meta 仅覆盖 4 种 HTML 模板 | 现有知识库 HTML 格式有限，通用化无收益 | 视新增模板而定 |
| 中文 slug 的 URI 编码 | 构建时可正确处理，不影响用户访问 | P2 |
| 全量同步无增量检测 | 177 篇笔记全量同步耗时可控（< 10s） | P3 |
| Wiki 链接未转换前的死链 | P0 先渲染内容，链接修复紧跟其后 | P0 尾声 / P1 |

### 不可接受的技术债

- **不得绕过 Content Collection 直接读取文件系统**：所有 MD 笔记必须通过 `src/content/notes/` + Content Collection API 访问，确保类型安全和构建校验
- **不得向 HTML iframe 注入 CSS / JS**：HTML 笔记是自包含文件，iframe 沙箱隔离不可破坏
- **不得修改 Obsidian vault 源文件**：`sync-vault.ts` 只读取和复制，不回写
- **不得为搜索引入运行时服务端**：搜索必须基于 Pagefind 构建时静态索引

---

## 12. 后续编码建议

### 编码前

1. 按本方案的目录结构创建 Astro 项目骨架
2. 优先实现 `scripts/sync-vault.ts` —— 它是所有后续开发的数据基础
3. 实现 `src/content/config.ts` 定义 Content Collection schema
4. 实现 `scripts/extract-html-meta.ts` 验证 4 种 HTML 模板的元数据提取
5. 配置 Dockerfile + nginx.conf，确保构建产物可正确部署

### 编码中

1. **构建管线优先**：先跑通 sync → extract → build → deploy 全流程，再开发页面和组件
2. 每个组件保持轻量，Astro 组件内只做模板渲染，不塞逻辑
3. 样式使用 CSS 变量，不硬编码颜色和间距——即使 Level 1 不做完整 Token 系统，也要在 BaseLayout 中定义基础变量
4. MD / HTML 分支渲染逻辑集中在 `[...slug].astro` 页面，通过 frontmatter 的 `format` 字段判断
5. 命名遵循 AI 友好规则：组件名 = UI 职责（NoteCard 不叫 ItemBox）

### 编码后

1. 运行 `decision-checklist` 自检（如已集成该 skill）
2. 架构变更记录在下方"架构变更记录"区
3. 构建管线改动后重新全量同步验证

### 质量基线（F9）

- **可访问性（a11y）**：语义化 HTML 标签（article/nav/aside）、图片 alt 文本、键盘可导航
- **性能**：Astro SSG 零 JS 输出、Pagefind 索引异步加载、HTML 笔记 lazy-load iframe
- **测试**：构建管线单元测试（元数据提取准确性）、端到端冒烟测试（Docker 部署后核心页面可访问）

### 衔接建议（可选）

| 后续步骤 | 推荐方式 | Fallback |
|---|---|---|
| 视觉设计 | 自行定义 CSS 变量色板 + 间距 | 本方案中 BaseLayout 的基础变量 |
| 编码流程 | 先管线后页面，先 P0 后 P1 逐步递进 | 本方案的编码建议 |
| 质量审计 | Docker 部署后手动验证核心页面 | decision-checklist 自检 |

---

## 架构约束（下游 skill 应遵守）

- **技术栈**：不得将 Astro SSG 替换为 SPA 框架（React/Vue/Next.js 等），不得引入运行时服务端
- **目录结构**：新增目录应遵循 `scripts/ → src/content/ → src/pages/ → src/layouts/ → src/components/ → src/utils/` 的分层规则
- **组件库**：不得引入 Ant Design / Element Plus / MUI 等重量级 UI 库，Content Register + Level 1 不需要
- **状态管理**：本项目为纯静态站点，不得引入 Redux / Zustand / Pinia 等状态管理方案
- **数据流**：所有数据必须通过构建管线（sync-vault → Content Collection / html-notes.json）注入，不得在运行时动态获取
- **HTML 笔记处理**：HTML 笔记必须通过 iframe 隔离渲染，不得将 HTML 内容直接注入 DOM
- **源文件保护**：构建脚本对 Obsidian vault 只读，不得修改源文件
- **MVP 边界**：超出 P0 范围的功能（搜索、主题切换、标签聚合等）应先更新本文件再实施
- **命名规范**：组件命名应遵循 AI 友好规则——可搜索、语义明确、遵循 Astro 约定

## 架构变更记录

| 日期 | 变更内容 | 原因 | 触发方 |
|---|---|---|---|
| 2026-06-13 | 初始架构方案创建 | 项目启动 | 前端架构设计 Skill |
