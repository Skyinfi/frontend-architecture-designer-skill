---
description: 前端架构设计模式 — 当用户要开始前端项目、做技术选型、规划目录结构、或诊断存量前端架构时激活。主动触发：新前端项目、新模块、技术选型讨论、前端重构、或用户首次提及前端编码但尚未规划架构。
alwaysApply: true
---

# 前端架构设计模式 / Frontend Architecture Design Pattern

本文件定义前端架构设计模式：在编码前先规划架构。所有决策规则、工作流和输出模板都在 `<SKILL_ROOT>/core/` 中，本文件只负责触发和读取入口。

## 安装假设 / Installation Assumption

本文件通常位于 `<SKILL_ROOT>/adapters/claude-code/CLAUDE.md`。读取核心资料时必须从 skill 包根目录解析路径：

```text
frontend-architecture-designer/
  adapters/
    claude-code/
      CLAUDE.md      ← 本文件（Claude Code 入口）
  core              ← 核心工作流目录
  references        ← 按需加载参考资料目录
```

`<SKILL_ROOT>` 是本 skill 包的安装根目录。安装到知识库时，对应 `AI工具与工作流/SKILL/frontend-architecture-designer/`。

## 必读文件 / Required Reading

1. 在做架构决策前，完整读取 `<SKILL_ROOT>/core/workflow.md`。
2. 在输出方案前，完整读取 `<SKILL_ROOT>/core/output-template.md`。
3. 根据 `<SKILL_ROOT>/core/workflow.md` 的路由表按需加载 `<SKILL_ROOT>/references/`。
4. 最终输出前使用 `<SKILL_ROOT>/core/decision-checklist.md` 自检。
5. 说明后续 AI coding 如何消费或更新架构方案时，读取 `<SKILL_ROOT>/core/maintenance.md`。

## 执行规则 / Execution Rules

- 如果用户尚未提供足够信息，先按 `<SKILL_ROOT>/core/workflow.md` 的 5 个必问题收集上下文。（→ `<SKILL_ROOT>/core/workflow.md` §4 关键问题清单）
- 加载 references 前，先判定 complexity Level 和 Register。（→ `<SKILL_ROOT>/core/workflow.md` §5 复杂度模型 + §6 Register 分轨）
- 输出 Architecture Read（一句话理解声明），让用户确认或纠正后再进入最终方案。（→ `<SKILL_ROOT>/core/workflow.md` §7 Architecture Read）
- 对组件库选择、state management、directory structure 等局部问题，走对应子工作流，不输出完整 12 项方案。（→ `<SKILL_ROOT>/core/workflow.md` §8 路由表）
- 对全量 frontend architecture design，使用 `<SKILL_ROOT>/core/output-template.md` 并按 Level 裁剪矩阵输出。
- 默认将架构方案写入 `docs/frontend-architecture.md`，除非用户指定其他路径。

## 边界 / Boundaries

- 不直接实现 UI 页面。
- 不跳过 Architecture Read。
- 不在本文件中复制 `<SKILL_ROOT>/core/workflow.md` 的完整规则。
- 不默认全量加载 references，只加载 workflow 路由到的文件。
- 信息不足时，先问聚焦问题；若仍不足，则输出带明确假设的 Level 1 草案。

## Skill 协作 / Skill Collaboration

架构方案输出完成后，根据项目需要建议后续步骤。**委托不是强制的，每个委托点都有 fallback。**

| 委托点 | 触发条件 | 推荐 | Fallback |
|---|---|---|---|
| 视觉设计 | 需要具体页面视觉 | Impeccable / Taste Skill | 架构方案内补充基础视觉规格 |
| 项目脚手架 | 方案确认，创建项目骨架 | create-next-app / create vite / create vue / create astro | 手动创建目录结构 |
| 编码工作流 | 进入功能开发 | Superpowers brainstorming → plan → TDD | 按架构方案逐功能实现 |
| 存量项目诊断 | 需要深度质量审计 | Impeccable audit / Superpowers systematic-debugging | 用 decision-checklist 自检 |

协作原则：委托非强制；每点有 fallback；架构方案是共享契约；架构约束优先（冲突时回到本 skill 重新评估）；架构变更必记录。

> 委托点的完整场景表和协作要求见 `<SKILL_ROOT>/core/workflow.md` §12（后续步骤与 Skill 协作）。
