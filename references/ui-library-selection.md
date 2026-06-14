# UI 组件库选型参考

> 本文件供 `core/workflow.md` §4（技术选型）和 §7（UI 组件库选择）查表使用。
> 聚焦**判断规则**和**反模式**，不搬运库文档。

---

## 一、AI 友好度评判标准

Agent 在选型时，应按以下维度评估组件库的 AI 友好度：

| 维度 | AI 友好 | AI 不友好 |
|---|---|---|
| 源码可见性 | 组件源码在本地（如 shadcn/ui），AI 可直接读取和修改 | 组件藏在 node_modules，行为黑盒 |
| 类型完整性 | TypeScript 类型完善，AI 可推断 props | 类型缺失或大量 `any`，AI 只能猜 |
| 命名规律性 | 组件名语义明确、模式稳定（Dialog/DialogContent/DialogTrigger） | 命名不一致或过度缩写 |
| 样式可控制 | 样式通过 CSS Variables / Tailwind / token 控制 | 样式内联或通过复杂主题系统包裹 |
| 文档与语料 | 文档规范、社区使用量大、AI 训练语料丰富 | 文档稀少或非英文为主且无类型 |

---

## 二、Register 差异化选型表

不同 Register 类型对组件库的需求差异巨大，**不应一套方案套所有类型**。

### Content（内容展示型）

**特征**：静态内容为主，零 JS 优先，无复杂交互。

| 应该 | 不应该 |
|---|---|
| 无头 UI 或极轻量方案 | 引入 Ant Design / Element Plus 等重型组件库 |
| Astro 组件 + 原生 HTML | 使用完整的 React 组件体系 |
| 仅在需要交互时引入 island 组件 | 默认加载整套组件库 runtime |
| 内联 SVG 图标 | 引入完整图标库 npm 包 |

**推荐方案**：
- Astro 项目 → Astro 组件 + 原生 HTML + CSS Variables
- React 内容站 → shadcn/ui（仅用少量组件）或 Radix UI（按需引入）
- Vue 内容站 → 无头方案或极少量组件

### Tool（工具/管理型）

**特征**：表单密集、表格密集、权限管理、CRUD 操作。

| 应该 | 不应该 |
|---|---|
| 企业级组件库（Ant Design / Element Plus / MUI） | shadcn/ui 做复杂表格（生态不成熟） |
| 选择表格/表单生态完整的库 | 同时引入多套组件库 |
| 利用库内置的权限、布局、菜单方案 | 自己从零造表格/表单轮子 |

**推荐方案**：
- React 后台 → Ant Design（国内）/ MUI（国际化）
- Vue 后台 → Element Plus（国内）/ Naive UI（现代化）
- SaaS Dashboard → Mantine 或 shadcn/ui + 自定义表格

### Desktop（桌面应用型）

**特征**：本地集成、现代轻量、AI 友好、易定制。

| 应该 | 不应该 |
|---|---|
| shadcn/ui + Radix + Tailwind | Ant Design（桌面体验差） |
| 关注 Bundle 体积 | 引入完整企业级组件体系 |
| 本地组件源码可维护 | 依赖 CDN 或远程资源 |

**推荐方案**：
- Tauri / Electron → React + shadcn/ui + Tailwind
- 通用桌面 → shadcn/ui + Radix UI

### Hybrid（混合型）

**特征**：同时存在内容页和工具页。

| 应该 | 不应该 |
|---|---|
| 按主要页面类型选择主组件库 | 混用多套组件库 |
| 内容页用轻量方案，工具页用重型方案 | 全站统一用重型组件库 |
| 明确划分哪些页面用哪套组件 | 同一页面混用不同组件库的组件 |

---

## 三、组件库速查对比

### React 生态

| 组件库 | AI 友好度 | 适合 Register | Bundle 影响 | 定制能力 | 核心优势 | 核心风险 |
|---|---|---|---|---|---|---|
| **shadcn/ui** | ★★★★★ | Tool / Desktop / Hybrid(主) | 极低（按需） | 极高（源码在本地） | AI 可直接修改源码，结构清晰 | 需理解 Tailwind，需自己维护源码 |
| **Radix UI** | ★★★★★ | 所有（作为底层） | 极低 | 极高 | Headless，语义化结构，无样式绑定 | 无默认样式，需自己处理视觉 |
| **Ant Design** | ★★★★☆ | Tool | 较大 | 中等 | 表格/表单/权限生态完整，中文语料丰富 | 风格偏传统，深度定制成本高 |
| **MUI** | ★★★★☆ | Tool / Hybrid | 中等 | 中高 | Material Design 体系完整，国际化 | MD 风格较重，深度定制成本较高 |
| **Mantine** | ★★★★☆ | Tool / Hybrid | 中等 | 中高 | Hooks 丰富，功能全面 | 与 Tailwind 融合不如 shadcn 自然 |
| **Chakra UI** | ★★★☆☆ | Tool(轻量) | 中等 | 高 | 主题系统友好，上手快 | 大型项目中可控性需评估 |

### Vue 生态

| 组件库 | AI 友好度 | 适合 Register | Bundle 影响 | 定制能力 | 核心优势 | 核心风险 |
|---|---|---|---|---|---|---|
| **Element Plus** | ★★★★☆ | Tool | 中等 | 中等 | 国内后台生态最成熟，语料丰富 | 风格偏传统后台 |
| **Naive UI** | ★★★★☆ | Tool | 中等 | 中高 | TS 友好，API 统一，现代风格 | 企业存量生态不如 Element Plus |

### 跨框架

| 组件库 | AI 友好度 | 适合 Register | 核心优势 | 核心风险 |
|---|---|---|---|---|
| **Ark UI** | ★★★★☆ | Desktop / 自定义设计系统 | 跨框架 Headless，状态机模型 | 学习成本高 |

---

## 四、按 Register 的推荐组合

### Content Register

```
Astro 项目：Astro 组件 + 原生 HTML + CSS Variables
  （零组件库，或仅 Pagefind/搜索用 island）

React 内容站：shadcn/ui（少量组件）+ Tailwind
  （大部分页面零 JS，交互组件按需加载）
```

### Tool Register

```
React 国内后台：Ant Design + TypeScript
React 国际化后台：MUI + TypeScript
Vue 国内后台：Element Plus + TypeScript
Vue 现代后台：Naive UI + TypeScript
```

### Desktop Register

```
Tauri/Electron：React + shadcn/ui + Radix UI + Tailwind
```

### Hybrid Register

```
按主要页面类型选择主组件库，次要页面用轻量补充方案。
不混用多套组件库。如果工具页面占比大，以 Tool Register 推荐为主。
```

---

## 五、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| 对内容展示站推 Ant Design / Element Plus | 静态内容不需要重型组件库，白白增加 JS 体积 | 用 Astro 组件 / 原生 HTML / 无头 UI |
| 对后台管理系统推 shadcn/ui 做复杂表格 | shadcn/ui 的 DataGrid 生态远不如 Ant Design / MUI | 后台用 Ant Design / MUI / Element Plus |
| 同时引入 Ant Design + Element Plus | 两套组件库冲突，Bundle 翻倍，维护噩梦 | 选一套，统一使用 |
| 对所有项目都推 React + shadcn/ui | Vue 后台用 shadcn/ui 不如 Element Plus 原生 | 按 Register + 技术栈匹配选型 |
| 为了"现代化"把 Ant Design 替换为 shadcn/ui | 已有后台运行良好，替换成本远大于收益 | 存量项目保持稳定，新项目再考虑 |
| 不选组件库，全部手写 | 重复造轮子，AI 维护成本高 | 至少选一套轻量方案覆盖基础组件 |
| 在 Desktop 项目中用 Ant Design | 桌面应用需要现代轻量体验，Ant Design 偏重 | 用 shadcn/ui + Tailwind |
| 引入组件库但只用了 Button 和 Input | 引入整个库只为几个基础组件不值得 | 考虑无头 UI 或极轻量方案 |

---

## 六、选型决策流程

```
1. 确定 Register 类型（Content / Tool / Desktop / Hybrid）
2. 确定技术栈（React / Vue / Astro）
3. 查上面的「按 Register 的推荐组合」
4. 评估项目特殊需求：
   - 是否需要复杂表格？→ Tool 类推荐
   - 是否需要零 JS？→ Content 类推荐
   - 是否需要本地源码可控？→ shadcn/ui
   - 是否面向国内企业？→ Ant Design / Element Plus
   - 是否面向国际？→ MUI / shadcn/ui
5. 检查反模式表，确认未命中
6. 输出选择 + 理由
```

---

## 七、资料来源

本文件基于 `ai-friendly-frontend-ui-libraries.md` 精简整理，补充了 Register 差异化视角和反模式。
