# Design Token 参考

> 本文件供 `output-template.md` §8（Design Token 初稿）查表使用。
> 仅在 **Level 2+** 输出时参考。Level 1 项目不需要 Design Token 体系。

---

## 一、Token 七类清单

一个完整但不过度的 Design Token 体系包含 7 类：

| # | 类别 | 覆盖场景 | 必须/可选 |
|---|---|---|---|
| 1 | Spacing | 间距、内边距、外边距 | 必须 |
| 2 | Radius | 圆角 | 必须 |
| 3 | Color | 背景、文字、功能色、边框 | 必须 |
| 4 | Typography | 字体族、字号、行高、字重 | 必须 |
| 5 | Shadow | 阴影层次 | 推荐 |
| 6 | Z-Index | 层叠顺序 | 推荐 |
| 7 | Layout | 侧栏宽、顶栏高、内容最大宽 | 推荐 |

---

## 二、各类 Token 参考值

### 2.1 Spacing

```css
:root {
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
}
```

**使用规则**：
- 组件内间距用 `--space-1` 到 `--space-4`
- 区块间距用 `--space-6` 到 `--space-12`
- 页面级间距用 `--space-12` 到 `--space-16`
- 不使用不在 token 体系中的间距值

### 2.2 Radius

```css
:root {
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 12px;
  --radius-full: 9999px;
}
```

**使用规则**：
- 按钮、输入框：`--radius-md`
- 卡片：`--radius-lg`
- 弹窗：`--radius-xl`
- 头像、徽章：`--radius-full`
- 小型 inline 元素：`--radius-sm`

### 2.3 Color

```css
:root {
  /* 背景 */
  --color-bg: #ffffff;
  --color-bg-elevated: #f9fafb;
  --color-bg-muted: #f3f4f6;

  /* 文字 */
  --color-text-primary: #111827;
  --color-text-secondary: #4b5563;
  --color-text-muted: #9ca3af;

  /* 功能色 */
  --color-primary: #3b82f6;
  --color-danger: #ef4444;
  --color-success: #22c55e;
  --color-warning: #f59e0b;

  /* 边框 */
  --color-border: #e5e7eb;
  --color-border-strong: #d1d5db;
}

/* 深色主题（如需要） */
[data-theme="dark"] {
  --color-bg: #1a1a2e;
  --color-bg-elevated: #25253e;
  --color-bg-muted: #2d2d4a;
  --color-text-primary: #f3f4f6;
  --color-text-secondary: #d1d5db;
  --color-text-muted: #9ca3af;
  --color-border: #374151;
  --color-border-strong: #4b5563;
}
```

**命名规则**：
- 语义化命名（`--color-text-primary`），不按色值命名（`--color-gray-900`）
- 功能色可额外维护一个色板 map，但组件只引用语义 token

### 2.4 Typography

```css
:root {
  --font-sans: "PingFang SC", "Microsoft YaHei", system-ui, -apple-system, sans-serif;
  --font-mono: "JetBrains Mono", "Fira Code", ui-monospace, monospace;

  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */

  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  --weight-normal: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;
}
```

**使用规则**：
- 正文用 `--text-base` + `--leading-normal`
- 标题用 `--text-xl` 到 `--text-3xl` + `--leading-tight` + `--weight-semibold`
- 代码用 `--font-mono`
- 辅助信息用 `--text-sm` + `--color-text-secondary`

### 2.5 Shadow

```css
:root {
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.15);
}
```

**使用规则**：
- 卡片悬浮：`--shadow-sm` → `--shadow-md`
- 弹窗：`--shadow-lg`
- 全屏遮罩上的浮层：`--shadow-xl`

### 2.6 Z-Index

```css
:root {
  --z-base: 0;
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-toast: 500;
  --z-tooltip: 600;
}
```

**使用规则**：
- 不使用任意的 z-index 值
- 新增层级时在此表中添加，不自定义数值
- 按功能从低到高排列

### 2.7 Layout

```css
:root {
  --sidebar-width: 240px;
  --sidebar-collapsed-width: 64px;
  --topbar-height: 56px;
  --content-max-width: 1280px;
  --page-padding: var(--space-6);
}
```

---

## 三、Tailwind 映射原则

如果项目使用 Tailwind CSS，Tailwind 只做 class 入口，值源仍来自 CSS Variables。这样主题切换只需改变 CSS Variables，不需要重建 Tailwind。

```js
// tailwind.config.js（示例，只展示模式）
module.exports = {
  theme: {
    extend: {
      colors: {
        bg: 'var(--color-bg)',
        primary: 'var(--color-primary)',
      },
      spacing: {
        1: 'var(--space-1)',
        4: 'var(--space-4)',
      },
      borderRadius: {
        md: 'var(--radius-md)',
      },
    },
  },
}
```

**不要**在 Tailwind 配置里维护第二套颜色、间距、圆角、字体完整表；只映射项目实际使用的 token。

---

## 四、Register 差异化建议

### Content Register

| 应该 | 不应该 |
|---|---|
| 极简 Token：字体 + 颜色 + 间距 | 建立完整 7 类 Token 体系 |
| CSS Variables 直接管理 | 引入 Tailwind（如不需要） |
| 构建时内联样式 | 运行时样式切换框架 |

### Tool Register

| 应该 | 不应该 |
|---|---|
| 完整 Token 体系（7 类） | 硬编码颜色和间距 |
| Tailwind + CSS Variables | 纯 inline style |
| 组件库的 token 系统作为基础 | 完全自定义 token（与组件库冲突） |

### Desktop Register

| 应该 | 不应该 |
|---|---|
| 完整 Token + 平台适配 | 忽略平台原生设计语言 |
| shadcn/ui 的 CSS Variables 体系 | 自己发明一套 token |

### Hybrid Register

| 应该 | 不应该 |
|---|---|
| 统一 Token 体系覆盖所有页面类型 | 不同页面类型用不同 token 标准 |
| 一套变量，不同页面组合不同 | 维护两套 token |

---

## 五、与既有 Token 体系对比

当存量项目已有 Design Token 时，参考本文件的价值是**校验和补全**，而非全量替换。

### 校验清单

| 检查项 | 健康信号 | 问题信号 |
|---|---|---|
| Token 完整性 | 7 类齐全（至少 spacing + radius + color + typography） | 只有颜色硬编码，间距随机 |
| 命名规范 | 语义化命名（`--color-primary`） | 按色值命名（`--blue-500`） |
| 主题切换 | 通过 CSS Variables + `data-theme` 切换 | JS 切换 className 重新渲染 |
| Token 与组件库 | 自定义 token 与组件库 token 协调 | 两套体系冲突，组件样式覆盖不上 |
| Tailwind 配置 | Tailwind `theme.extend` 引用 CSS Variables | Tailwind 与 CSS Variables 各管一套 |
| 维护成本 | Token 总数 35–70 个，文档化 | Token 数量爆炸（> 100 个），命名混乱 |

### 补充建议

- 如果 token 数量 < 35，建议补全至 7 类（至少 spacing + radius + color + typography）
- 如果命名按色值，建议改语义化命名（一次性成本，主题切换成本大幅降低）
- 如果 Tailwind 与 CSS Variables 双轨，建议合并为 CSS Variables 作为单一真相源
- 如果没有深色主题，建议在 Level 2 补 `[data-theme="dark"]` 覆盖
- 如果 token 散落在多个文件，建议集中到 `src/styles/tokens.css`

---

## 六、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| 硬编码 `color: #3b82f6` | 修改需要全文搜索替换 | 用 `var(--color-primary)` |
| 对 Level 1 项目输出完整 Token | 过度设计，Level 1 不需要 | 标注"暂不展开"，用基础 CSS |
| Token 命名按色值（`--blue-500`） | 换主题时语义丢失 | 按语义命名（`--color-primary`） |
| 100+ 个 token 变量 | 维护成本爆炸 | 7 类 × 5–10 个 = 35–70 个足够 |
| Tailwind 和 CSS Variables 各管一套 | 两套体系冲突 | Tailwind 引用 CSS Variables |
| 不用 token，全部 Tailwind 原子类 | 主题切换困难 | 关键值用 token，Tailwind 引用 token |
| 在 token 中放组件特定值（`--login-btn-width`） | token 是全局基础，不含组件逻辑 | 组件特定值用组件内 scoped 变量 |

---

## 七、AI 友好命名约定

Token 命名影响 AI 引用、改主题、改设计系统的效率。
命名四原则统一遵循 `core/workflow.md` §7.5（可搜索、语义明确、遵循约定、模式稳定）。本节只补充 Token 领域的命名模式。

### Token 命名规则

| 类别 | 命名模式 | 示例 |
|---|---|---|
| Spacing | `--space-{N}` | `--space-1`、`--space-4`、`--space-8` |
| Radius | `--radius-{size}` | `--radius-sm`、`--radius-md`、`--radius-full` |
| Color | `--color-{角色}-{属性}` | `--color-bg`、`--color-text-primary`、`--color-primary` |
| Typography - 字号 | `--text-{size}` | `--text-sm`、`--text-base`、`--text-2xl` |
| Typography - 行高 | `--leading-{密度}` | `--leading-tight`、`--leading-normal` |
| Typography - 字重 | `--weight-{level}` | `--weight-normal`、`--weight-semibold` |
| Shadow | `--shadow-{size}` | `--shadow-sm`、`--shadow-md`、`--shadow-lg` |
| Z-Index | `--z-{role}` | `--z-dropdown`、`--z-modal`、`--z-toast` |
| Layout | `--{区域}-{属性}` | `--sidebar-width`、`--topbar-height`、`--content-max-width` |

### 命名禁忌

- ❌ 按色值命名（`--blue-500`、`--gray-100`）— 换主题时语义丢失
- ❌ 按组件命名（`--login-btn-bg`）— token 是全局基础，不含组件逻辑
- ❌ 命名不规律（`--p`、`--primary-color`、`--main`）— AI 无法推断正确用法

---

## 八、Level 与 Token 策略

| Level | Token 策略 | 输出内容 |
|---|---|---|
| **Level 1** | 不需要 | 标注"暂不展开" |
| **Level 2** | 基础 Token（4 类：spacing + radius + color + typography） | 输出 4 类参考值 |
| **Level 3** | 完整 Token（7 类） | 输出全部 7 类 + Tailwind 配置 + 深色主题变量 |
