# Axiom UI 组件库

自研 Vue 3 组件库，基于 Tailwind v4、Floating UI、Material Symbols 图标、Geist / JetBrains Mono 字体。

## 组件一览

| 组件 | 说明 |
|---|---|
| `AxButton` | variant: primary / outline / ghost / danger；size: xs~icon-lg |
| `AxInput` | 单行/密码/多行文本；size: xs~lg；#prefix / #suffix |
| `AxSelect` | 单选/多选/可搜索；基于 AxDropdown + useFloating |
| `AxDropdown` | 浮层菜单；trigger: click / hover / contextmenu |
| `AxDialog` | 模态弹窗；焦点锁定；滚动锁定；#footer slot |
| `AxAlert` | type: info / success / warning / error |
| `AxSlider` | 范围滑块 |
| `AxTooltip` | hover 文字提示 |
| `AxPropPanel` | schema 驱动属性面板（switch/slider/select/input/textarea/segmented） |
| `AxSwitch` | 开关组件；role="switch" + aria-checked |
| `AxImage` | 懒加载图片；三态（loading/error/loaded）；点击预览；hover 放大图标；自适应宽高比 |
| `AxJsonViewer` | 可折叠 JSON 树查看器；层级展开控制（-1/0/N）；递归折叠；长文本横向滚动 |
| `AxImageViewer` | 全屏图片查看器；缩放/旋转/翻转；键盘快捷键；下载 |
| `useNotify` | 封装 vue-sonner 通知 |
| `useFloating` | Floating UI 定位 hook |
| `FloatingBall` | 可拖拽浮动球组件 |

## 文档导航

| 文档 | 内容 |
|---|---|
| [install.md](./docs/install.md) | 依赖安装、Vite 配置、全局样式、main.ts 注册、字体图标、Toaster 挂载 |
| [design-tokens.md](./docs/design-tokens.md) | 颜色语义 token、字号阶梯、图标规范、间距/圆角/阴影数值 |
| [components.md](./docs/components.md) | **每个组件的 Props / Slots / Events / 使用场景 / 代码示例** |
| [ui-style.md](./docs/ui-style.md) | 设计原则、控件尺寸体系、布局模式、交互规范、动效、检查清单 |
| [html-cdn.md](./docs/html-cdn.md) | HTML 单页（无构建）快速接入 Token |
| [content-script-shadow-dom.md](./docs/content-script-shadow-dom.md) | WXT 浏览器扩展 Content Script Shadow DOM 接入 |

## 快速开始

```bash
# 1. 复制组件到项目
cp -r assets/* src/components/ui/

# 2. 按 install.md 完成依赖安装和配置
# 3. 按 components.md 使用组件
```

## 安装与接入

详见 **[install.md](./docs/install.md)**。完整流程包括：依赖安装 → Vite 配置 → 全局 CSS `@theme` → main.ts 注册组件 → Toaster 挂载 → 自检清单。

## 使用组件

全局注册后直接使用标签：

```vue
<AxButton variant="primary">保存</AxButton>
<AxInput v-model="name" placeholder="名称" />
<AxSelect v-model="selected" :options="options" placeholder="请选择" />
<AxDialog v-model="open" title="设置" icon="settings">
  <template #footer="{ close }">
    <AxButton variant="outline" @click="close">取消</AxButton>
    <AxButton @click="save">保存</AxButton>
  </template>
</AxDialog>
```

按需导入：

```ts
import { AxButton, useNotify, useFloating } from '@/components/ui'
import { FloatingBall } from '@/components/ui'
```

**完整 API 参考（每个组件的所有 Props、Slots、Events、使用场景）见 [components.md](./docs/components.md)**。

## WXT 浏览器扩展（Shadow DOM）

浮层组件（`AxTooltip`、`AxDropdown`、`AxSelect`）依赖 `<Teleport>`，Content Script 中须配合 Shadow DOM + `provideTeleportTarget()`。详见 **[content-script-shadow-dom.md](./docs/content-script-shadow-dom.md)**。

## 更新同步

组件库更新后，在目标项目中执行同步：

```bash
node <skill-dir>/scripts/sync.js
```

`sync.js` 自动执行两步：
1. 重新拉取最新 skill
2. 将 `assets/` 覆盖写入 `src/components/ui/`

若目标路径不同，可传参指定：

```bash
node <skill-dir>/scripts/sync.js src/renderer/components/ui
```

## 为 AI 编辑器配置规则

组件库接入后，必须配置规则让 AI 优先使用 `Ax*` 组件。

### WorkBuddy

在项目 `.workbuddy/rules.md` 中添加：

```markdown
# UI 组件规则

本项目使用 Axiom UI 组件库（`src/components/ui/`）。

- 所有 UI 界面必须使用 Ax* 组件，禁止手写原生 HTML 按钮/输入框/选择器
- 布局使用 Tailwind v4 class，间距使用 gap-ax-sm / p-ax-md 等 ax- 前缀间距
- 颜色使用语义 token（text-primary / bg-surface-container-low），禁止硬编码色值
- 图标使用 Material Symbols Outlined：<span class="material-symbols-outlined">icon_name</span>
- 通知使用 useNotify()，禁止手写通知逻辑
- 弹窗使用 AxDialog，浮层使用 AxDropdown / AxTooltip
```

### Cursor

在项目根目录创建 `.cursorrules`，追加：

```
- Use Ax* Vue components from src/components/ui/ for all UI elements
- Do NOT write raw HTML buttons, inputs, or selects -- use AxButton, AxInput, AxSelect
- Space utilities: gap-ax-sm, p-ax-md, etc.
- Colors: use semantic tokens (text-primary, bg-surface-container-low), never hardcoded hex
- Icons: Material Symbols Outlined via span.material-symbols-outlined
- Dialogs: AxDialog; overlays: AxDropdown/AxTooltip; notifications: useNotify()
```

### GitHub Copilot

在项目根目录创建 `.github/copilot-instructions.md`：

```markdown
Prefer Ax* Vue components from src/components/ui/:
- AxButton, AxInput, AxSelect, AxDialog, AxDropdown, AxTooltip, AxAlert, AxSlider, AxSwitch, AxImage, AxJsonViewer, AxImageViewer, AxPropPanel
- Space: gap-ax-sm, p-ax-md; Colors: semantic Tailwind tokens
- Icons: span.material-symbols-outlined; Notifications: useNotify()
```

## 示例界面

`assets/layout/` 中包含完整示例，可复制到目标项目 `src/views/` 参考：

| 文件 | 说明 |
|---|---|
| `ComponentsView.vue` | 组件展示页 — 每个 Ax* 组件带实时属性编辑面板 |
| `SettingsView.vue` | 设置界面 — 分组配置卡（通用/性能/安全/通知/高级） |
| `ConsoleLayout.vue` | 控制台布局壳 — 侧栏导航 + 主内容区 + 通知 + 弹窗调度 |
| `DemoView.vue` | 交互工坊 — 组件联动演示 |

## 文件结构

```
ax-ui-kit/
├── SKILL.md                       # 本文件
├── scripts/
│   └── sync.js                    # 一键同步脚本
├── assets/                        # 组件库源码（复制到目标项目 src/components/ui/）
│   ├── index.ts                   # 组件注册入口（registerComponents）
│   ├── types.ts
│   ├── AxButton.vue ...           # 13 个 Vue 组件
│   ├── hooks/                     # useNotify / useFloating / useTeleportTarget
│   ├── functional/                # FloatingBall
│   └── layout/                    # 示例布局页面
└── references/                    # 参考文档
    ├── install.md                 # 安装文档（依赖、配置、main.ts、检查清单）
    ├── design-tokens.md           # 设计 token（颜色、字体、图标、间距、圆角）
    ├── components.md              # 组件 API 参考（Props/Slots/Events/示例）
    ├── ui-style.md                # 设计规范（原则、尺寸、布局、交互、动效）
    ├── html-cdn.md                # HTML 单页快速接入
    └── content-script-shadow-dom.md  # WXT Content Script 接入指南
```
