# 主题与国际化参考

> 本文件供 `output-template.md` §10（主题与国际化预留）查表使用。
> 仅在 **Level 3** 输出时参考。Level 1/2 项目只需要预留接口。

---

## 一、主题系统

### 主题模式

| 模式 | 说明 | 实现方式 |
|---|---|---|
| Light | 默认亮色主题 | `:root` 下的 CSS Variables |
| Dark | 深色主题 | `[data-theme="dark"]` 覆盖变量 |
| System | 跟随系统偏好 | `prefers-color-scheme` 媒体查询 |

### 实现结构

```css
/* 基础：亮色主题 */
:root {
  --color-bg: #ffffff;
  --color-text-primary: #111827;
  /* ... 其他 token ... */
}

/* 深色主题覆盖 */
[data-theme="dark"] {
  --color-bg: #1a1a2e;
  --color-text-primary: #f3f4f6;
  /* ... 只需覆盖变化的变量 ... */
}

/* 系统跟随 */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme]) {
    --color-bg: #1a1a2e;
    --color-text-primary: #f3f4f6;
  }
}
```

### 主题切换逻辑

```typescript
// utils/theme.ts
type Theme = 'light' | 'dark' | 'system'

function getTheme(): Theme {
  return (localStorage.getItem('theme') as Theme) || 'system'
}

function setTheme(theme: Theme) {
  localStorage.setItem('theme', theme)
  const resolved = theme === 'system'
    ? window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
    : theme
  document.documentElement.setAttribute('data-theme', resolved)
}
```

### 各阶段策略

| 阶段 | 主题能力 | 说明 |
|---|---|---|
| MVP / Level 1 | 只用 CSS Variables，不切换 | Token 值写在 `:root`，方便后续切换 |
| Level 2 | Light + Dark，手动切换 | `[data-theme]` 覆盖 + localStorage 持久化 |
| Level 3 | Light + Dark + System + 动态切换 | 完整主题系统 + SSR 兼容（防闪烁） |

### SSR / SSG 防闪烁

```
在 <head> 中内联一段脚本，在渲染前读取 localStorage 设置 data-theme，
避免页面加载时先显示亮色再闪烁为深色。
```

---

## 二、国际化（i18n）

### MVP 判断

| 问题 | 回答 | 建议 |
|---|---|---|
| 项目是否只面向单一语言用户？ | 是 | MVP 不做 i18n，只预留接口 |
| 是否有明确的国际化需求？ | 否 | 延后到需求出现时 |
| 是否是中文个人项目？ | 是 | 不需要 i18n |

### i18n 框架选型

| 框架 | 适合场景 | 核心特点 |
|---|---|---|
| **react-i18next** | React 项目 | 最成熟，社区最大 |
| **vue-i18n** | Vue 项目 | Vue 官方推荐 |
| **astro-i18n** | Astro 项目 | 构建时生成多语言页面 |

### 目录结构

```
src/
├── i18n/
│   ├── index.ts              # i18n 初始化配置
│   ├── locales/
│   │   ├── zh-CN.json        # 中文
│   │   └── en-US.json        # 英文
│   └── types.ts              # key 类型定义
```

### 文案 key 命名规则

**原则**：按功能模块组织，用点分路径，不用扁平字符串。

```json
{
  "common": {
    "confirm": "确认",
    "cancel": "取消",
    "save": "保存",
    "delete": "删除",
    "search": "搜索"
  },
  "auth": {
    "login": "登录",
    "logout": "退出",
    "username": "用户名",
    "password": "密码"
  },
  "product": {
    "title": "商品管理",
    "create": "新建商品",
    "edit": "编辑商品",
    "deleteConfirm": "确定删除商品「{name}」吗？"
  }
}
```

| 规则 | 好 | 差 |
|---|---|---|
| 按模块分层 | `product.deleteConfirm` | `productDeleteConfirmText` |
| 语义明确 | `auth.login` | `btn1` |
| 支持插值 | `删除「{name}」？` | `删除` + 拼接 |

### 日期 / 数字 / 货币格式化

| 能力 | 推荐方案 | 说明 |
|---|---|---|
| 日期格式化 | `Intl.DateTimeFormat`（浏览器原生） | 不引入 moment.js |
| 数字格式化 | `Intl.NumberFormat`（浏览器原生） | 千分位、小数位 |
| 货币格式化 | `Intl.NumberFormat` + `style: 'currency'` | 不引入额外库 |
| 相对时间 | `Intl.RelativeTimeFormat` | "3 天前"、"2 小时后" |

**统一入口**：

```typescript
// utils/format.ts
export function formatDate(date: Date | string, locale = 'zh-CN'): string {
  return new Intl.DateTimeFormat(locale, {
    year: 'numeric', month: '2-digit', day: '2-digit'
  }).format(new Date(date))
}

export function formatNumber(num: number, locale = 'zh-CN'): string {
  return new Intl.NumberFormat(locale).format(num)
}
```

---

## 三、Register 差异化建议

### Content Register

| 主题 | 建议 |
|---|---|
| 主题 | MVP 不做切换。CSS Variables 写在 `:root` 即可，P2 加深色模式 |
| i18n | 不需要。中文内容站无国际化需求 |

### Tool Register

| 主题 | 建议 |
|---|---|
| 主题 | Light + Dark 切换是标配。组件库通常内置主题能力 |
| i18n | 按产品规划。国内后台通常不需要，国际化产品必须做 |

### Desktop Register

| 主题 | 建议 |
|---|---|
| 主题 | 跟随系统偏好（System）+ 手动切换。shadcn/ui 的 CSS Variables 天然支持 |
| i18n | 桌面应用通常只做 1–2 种语言。预留 key 即可 |

### Hybrid Register

| 主题 | 建议 |
|---|---|
| 主题 | 全站统一主题系统，不同页面类型共用 token |
| i18n | 与 Tool Register 策略一致 |

---

## 四、与既有主题/i18n 对比

当存量项目已有主题或国际化实现时，参考本文件的价值是**校验和补全**，而非全量替换。

### 校验清单（主题）

| 检查项 | 健康信号 | 问题信号 |
|---|---|---|
| 主题实现 | CSS Variables + `data-theme` 切换 | JS 切换 className、CSS-in-JS 运行时 |
| 主题模式 | 至少支持 Light / Dark | 只有单一主题，无法扩展 |
| 持久化 | localStorage 保存用户偏好 | 刷新页面主题丢失 |
| SSR/SSG | `<head>` 内联脚本防闪烁 | 加载时先亮后暗闪烁 |
| Token 复用 | 主题切换复用 Design Token | 主题相关值硬编码在 CSS |

### 校验清单（i18n）

| 检查项 | 健康信号 | 问题信号 |
|---|---|---|
| 文案管理 | 集中管理（i18n 文件） | 硬编码中文字符串散落 |
| Key 命名 | 按模块分层（`product.deleteConfirm`） | 扁平字符串（`productDeleteConfirmText`） |
| 日期/数字 | 用 `Intl.*` 原生 API | 引入 moment.js（200KB+） |
| 范围 | 只对用户可见文案做 i18n | 把 API 文档/错误码都翻译 |
| 插值支持 | 支持 `{name}` 等占位符 | 字符串拼接 |

### 补充建议

- 如果主题用 JS 切换 className，建议迁移到 CSS Variables + `data-theme`（零运行时）
- 如果没有 SSR 防闪烁，建议在 `<head>` 内联 localStorage 读取脚本
- 如果文案硬编码散落，建议先提取到 `constants/strings.ts`，再考虑 i18n
- 如果用 moment.js 做日期格式化，建议替换为 `Intl.DateTimeFormat`
- 如果 i18n key 命名扁平，建议改为 `module.submodule.key` 路径式

---

## 五、常见反模式

| 反模式 | 为什么是错的 | 正确做法 |
|---|---|---|
| MVP 阶段做完整 i18n | 无需求时过度设计 | 只预留接口，key 写在组件内，后续提取 |
| 硬编码中文字符串散落各处 | 后续国际化成本高 | 集中管理文案，或至少用常量 |
| 引入 moment.js 做日期格式化 | 体积大（200KB+），已停止维护 | 用 `Intl.DateTimeFormat` |
| 主题用 JS 切换 class | 性能差、闪烁 | 用 `data-theme` 属性 + CSS Variables |
| 不处理 SSR 主题闪烁 | 页面加载先亮后暗 | `<head>` 内联脚本读取 localStorage |
| 为主题切换引入 CSS-in-JS 运行时 | 增加运行时开销 | CSS Variables 零运行时 |
| 把所有文案都做 i18n key | API 文档、错误码等不需要翻译 | 只对用户可见文案做 i18n |

---

## 六、AI 友好命名约定

主题与 i18n 相关命名影响 AI 切换主题、提取文案、添加新语言的效率。
命名四原则统一遵循 `core/workflow.md` §7.5（可搜索、语义明确、遵循约定、模式稳定）。本节只补充主题与 i18n 领域的命名模式。

### 命名规则

| 类型 | 命名模式 | 示例 |
|---|---|---|
| 主题属性 | `data-theme="{mode}"` | `data-theme="light"`、`data-theme="dark"` |
| 主题存储 key | `theme` / `theme-preference` | `localStorage.theme` |
| 主题类型 | union type | `type Theme = 'light' \| 'dark' \| 'system'` |
| i18n 命名空间 | `module.submodule.key` | `auth.login`、`product.deleteConfirm` |
| 通用文案 | `common.{action}` | `common.confirm`、`common.cancel` |
| 错误信息 | `errors.{module}.{key}` | `errors.auth.invalidCredentials` |
| 日期/数字 locale | `zh-CN` / `en-US` | `formatDate(date, 'zh-CN')` |
| 文案插值 | `{variableName}` | `删除「{name}」？` |

---

## 七、Level 与主题/i18n 策略

| Level | 主题策略 | i18n 策略 | 输出内容 |
|---|---|---|---|
| **Level 1** | 不需要 | 不需要 | 标注"暂不展开" |
| **Level 2** | Light + Dark 预留 | 不需要 | 主题表格（MVP 预留 vs 后续） |
| **Level 3** | 完整主题系统 | 按需 | 主题 + i18n 完整方案 |
