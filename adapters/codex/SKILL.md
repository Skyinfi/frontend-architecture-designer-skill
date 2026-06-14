---
name: frontend-architecture-designer
description: Use when 需要在前端编码前做 frontend architecture 规划、技术选型、UI library 选择、directory structure 设计、组件边界、state management、Design Token、主题/i18n、MVP 范围，或诊断存量前端架构。
---

# 前端架构设计 / Frontend Architecture Designer

这是 Codex 的前端架构设计 skill 入口，用于在编码前生成可执行的 frontend architecture plan。入口文件保持精简：实质规则放在 `core/`，详细参考资料放在 `references/`。

## 安装假设 / Installation Assumption

当本文件被复制到包含同级 `core/`、`references/` 和可选 `agents/` 目录的 skill root 时，即可作为安装态 `SKILL.md` 使用：

```text
.agents/skills/frontend-architecture-designer/
  SKILL.md
  core/
  references/
  agents/openai.yaml
```

不要直接从 `frontend-architecture-designer/adapters/codex/` 运行它，除非已经把资源复制到同级目录，或显式修正路径。

## 必读文件 / Required Reading

1. 在做架构决策前，完整读取 `core/workflow.md`。
2. 在输出方案前，完整读取 `core/output-template.md`。
3. 根据 `core/workflow.md` 的 routing table 按需加载 `references/`。
4. 最终输出前使用 `core/decision-checklist.md` 自检。
5. 说明后续 AI coding 如何消费或更新架构方案时，读取 `core/maintenance.md`。

## 执行规则 / Execution Rules

- 如果用户尚未提供足够信息，先按 `core/workflow.md` 的 5 个必问题收集上下文。
- 加载 references 前，先判定 complexity Level 和 Register。
- 输出 Architecture Read，让用户确认或纠正后再进入最终方案。
- 对组件库选择、state management、directory structure 等局部问题，走对应 sub-workflow，不输出完整 12 项方案。
- 对全量 frontend architecture design，使用 `core/output-template.md` 并按 Level 裁剪矩阵输出。
- 默认将架构方案写入 `docs/frontend-architecture.md`，除非用户指定其他路径。

## 边界 / Boundaries

- 不直接实现 UI 页面。
- 不跳过 Architecture Read。
- 不在本入口文件中复制 `core/workflow.md` 的完整规则。
- 不默认全量加载 references，只加载 workflow 路由到的文件。
- 信息不足时，先问聚焦问题；若仍不足，则输出带明确假设的 Level 1 草案。
