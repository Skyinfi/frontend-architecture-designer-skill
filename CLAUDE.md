# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **frontend architecture design skill** — a structured workflow system that guides AI agents through frontend project architecture planning *before* coding begins. It produces a `frontend-architecture.md` document covering tech stack selection, directory structure, component strategy, state management, Design Tokens, and MVP scope.

This is **not a code project** — it's a knowledge base of markdown documents with adapters for different AI coding platforms (Claude Code, Cursor, Codex).

## Architecture

```
frontend-architecture-designer/
  core/                    ← Core workflow engine (4 files)
    workflow.md            ← THE entry point: defines the full workflow, routing, complexity model
    output-template.md     ← Output structure template (12 sections + YAML frontmatter)
    decision-checklist.md  ← Self-check list with anti-pattern detection (Anti-Slop)
    maintenance.md         ← How to maintain/update architecture docs over time
  references/              ← Domain knowledge loaded on-demand by the routing table
    ui-library-selection.md
    directory-structure.md
    component-architecture.md
    state-and-data.md
    design-tokens.md
    theme-and-i18n.md
    domain-boundaries.md
    mvp-planning.md
  adapters/                ← Platform-specific entry points
    claude-code/CLAUDE.md  ← Claude Code adapter (alwaysApply: true)
    cursor/                ← Cursor rules adapter
    codex/                 ← OpenAI Codex adapter
  outputs/                 ← Generated architecture documents (samples/examples)
```

## Key Concepts

- **Complexity Levels (L1/L2/L3)**: Driven by team size, lifecycle, and business complexity. Determines output depth — L1 is flat/minimal, L3 is full enterprise.
- **Register**: Architecture strategy track — `Content` (static/SEO), `Tool` (SaaS/dashboard), `Desktop` (Tauri/Electron), `Hybrid`. Affects framework, component library, and state decisions.
- **Routing table** (workflow.md §8): Maps user intent to sub-workflows and reference files. Local questions ("help me pick a component library") skip the full 12-section output.
- **Architecture Read**: Mandatory one-sentence understanding declaration before outputting any plan.
- **Anti-Slop checks** (decision-checklist.md §9): 10 hard-fail patterns (e.g., pushing React+shadcn for Content projects, full DDD for L1).

## Working with This Repo

- **No build/test/lint commands** — this is a pure documentation project.
- When editing core files, maintain the routing table consistency between `workflow.md` §8 and the reference file names.
- When editing references, ensure anti-pattern sections stay in sync with `decision-checklist.md` §9.
- The YAML frontmatter in output templates has 11 fields — changing them affects all downstream consumers.
- Adapters in `adapters/claude-code/CLAUDE.md` use `<SKILL_ROOT>` path placeholders that resolve at install time.

## Language

Core content is written in Chinese (中文). Maintain language consistency when editing.
