# Frontend Architecture Designer

前端架构设计 Skill — 在编码前完成项目级前端架构规划。

## 这是什么

一个结构化的工作流系统，帮助 AI Agent 在前端项目编码前完成架构规划。覆盖技术选型、组件库选择、目录结构、模块边界、组件封装策略、Design Token、状态架构、主题/i18n 预留、MVP 范围规划。

产出物为一份结构化的 `frontend-architecture.md`，供后续 AI coding 和下游 skill 消费。

## 目录结构

```
frontend-architecture-designer/
├── core/                    ← 核心工作流（4 个文件）
│   ├── workflow.md          ← 工作流入口：完整流程、路由表、复杂度模型
│   ├── output-template.md   ← 输出模板（12 章节 + YAML frontmatter）
│   ├── decision-checklist.md← 自检清单（含 Anti-Slop 反模式检测）
│   └── maintenance.md       ← 架构方案维护与变更指南
├── references/              ← 按需加载的参考知识
│   ├── ui-library-selection.md
│   ├── directory-structure.md
│   ├── component-architecture.md
│   ├── state-and-data.md
│   ├── design-tokens.md
│   ├── theme-and-i18n.md
│   ├── domain-boundaries.md
│   └── mvp-planning.md
├── adapters/                ← 各平台适配器
│   ├── claude-code/         ← Claude Code（CLAUDE.md）
│   ├── cursor/              ← Cursor（.mdc rules）
│   └── codex/               ← OpenAI Codex（SKILL.md）
└── outputs/                 ← 已生成的架构方案示例
```

## 核心概念

### 复杂度 Level

| 维度 | L1 | L2 | L3 |
|---|---|---|---|
| 团队规模 | 1 人 | 3-5 人 | 10+ 人 |
| 生命周期 | < 1 个月 | 3-6 个月 | > 6 个月 |
| 业务复杂度 | 原型验证 / 内部工具 | 明确业务模型 | 多业务域 / 复杂权限 |

三维度各自定级，取最高级。Level 决定输出深度和 reference 加载范围。

### Register（架构策略分轨）

| Register | 适用场景 | 典型框架 |
|---|---|---|
| Content | 知识库、博客、官网、文档站 | Astro / Next.js SSG |
| Tool | SaaS、后台管理、数据看板 | React + Vite / Next.js |
| Desktop | Tauri / Electron 应用 | Tauri + React |
| Hybrid | 内容展示 + 工具操作混合 | 按需组合 |

### 工作流

**新项目（工作流 A）**：5 必问题 → 复杂度判定 → Register 判定 → Architecture Read → Reference 路由 → 输出架构方案 → 自检

**存量项目（工作流 B）**：5 必问题 → 复杂度判定 → Register 判定 → 结构化上下文采集 → 现状诊断 → 增量改造方案 → 自检

**局部问题**：走路由表子工作流，只输出相关章节，不强制完整 12 项。

## 安装使用

### Claude Code

将 `adapters/claude-code/CLAUDE.md` 复制到目标项目的 `.claude/` 目录，或作为 skill 包安装到知识库。

### Cursor

将 `adapters/cursor/frontend-architecture.mdc` 放入目标项目的 `.cursor/rules/`。

### 其他平台

参考 `adapters/codex/` 中的适配格式自行调整。

## 产出路径

| 场景 | 路径 |
|---|---|
| 目标项目已有目录 | `{项目根}/docs/frontend-architecture.md` |
| 无真实项目目录（试跑/诊断） | `outputs/{项目名}-frontend-architecture.md` |
| 用户指定路径 | 优先使用用户指定路径 |
