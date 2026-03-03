# 字体图标管理 (font_grabber)

通过 `zerone font_grabber` 命令从阿里巴巴矢量图标库 (iconfont.cn) 下载字体文件到本地项目。

## 目录结构

```
src/assets/iconfont/
├── iconfont.config.json   ← 配置文件（cssUrl + output）
├── iconfont.css            ← 生成的样式文件（@font-face + icon class）
├── iconfont.ttf            ← 字体文件
├── iconfont.woff           ← 字体文件
└── iconfont.woff2          ← 字体文件
```

## 前置条件

确保 `package.json` 的 `scripts` 中包含 font 脚本。

**大部分项目只有一个 iconfont**，使用默认脚本即可：

```
"font": "zerone font_grabber -p ./src/assets/iconfont"
```

**如果项目有多个 iconfont**，可以像 `api:` 一样使用系列命名：

```
"font": "zerone font_grabber -p ./src/assets/iconfont"
"font:gfont": "zerone font_grabber -p ./src/assets/gfont"
```

对应执行 `pnpm font` 或 `pnpm font:gfont`。

如果脚本不存在，在 `scripts` 中添加（建议放在 `api:` 系列脚本之前）。

---

## 场景一：更新图标

用户提供新的 iconfont CSS 链接（格式如 `//at.alicdn.com/t/c/font_XXXXXXX_XXXXXXX.css`），执行以下步骤：

### 步骤 1：更新配置文件

修改 `src/assets/iconfont/iconfont.config.json`，将 `cssUrl` 替换为新链接：

```json
{
  "cssUrl": "//at.alicdn.com/t/c/font_5112154_xxxxxxx.css",
  "output": "iconfont.css"
}
```

`output` 字段保持不变，只更新 `cssUrl`。

### 步骤 2：执行下载命令

根据实际的脚本名执行，如 `pnpm font` 或 `pnpm font:gfont`：

```bash
pnpm font
```

该命令会根据 `cssUrl` 下载远程 CSS，解析其中的 `@font-face` 引用，将所有字体文件下载到本地，并重写 CSS 中的路径为本地相对路径。

执行后 `src/assets/iconfont/` 下的以下文件会被更新：

- `iconfont.css` — 重写后的本地样式（含 `@font-face` 和所有 `icon-*` class）
- `iconfont.ttf`
- `iconfont.woff`
- `iconfont.woff2`

---

## 场景二：首次创建（项目中尚无 iconfont）

### 步骤 1：确保 package.json 有 font 脚本

检查并添加 `"font": "zerone font_grabber -p ./src/assets/iconfont"` 到 `package.json` scripts。

### 步骤 2：创建配置文件

创建 `src/assets/iconfont/iconfont.config.json`：

```json
{
  "cssUrl": "//at.alicdn.com/t/c/font_XXXXXXX_XXXXXXX.css",
  "output": "iconfont.css"
}
```

用户需要提供实际的 iconfont CSS 链接。

### 步骤 3：执行命令生成字体文件

```bash
pnpm font
```

首次执行会在 `src/assets/iconfont/` 目录下创建 `iconfont.css`、`iconfont.ttf`、`iconfont.woff`、`iconfont.woff2`。

### 步骤 4：在 main.ts 中引入

检查 `src/main.ts` 是否已包含以下引入：

```ts
import './assets/iconfont/iconfont.css';
```

如果没有，添加到 CSS 引入区域（与其他样式 import 放在一起）。参考已有项目中的位置：

```ts
// css
import './assets/iconfont/iconfont.css';
// import 'element-plus/dist/index.css';
import 'giime/theme-chalk/element/index.scss';
```

---

## 图标使用方式

在 Vue 模板中使用 `<i>` 标签，class 组合格式为 `{字体族class} icon-{图标名}`：

```vue
<i class="acore-font icon-Dify text-[32px] text-blue-600" />
```

### class 说明

| class           | 来源                                      | 说明                                                   |
| --------------- | ----------------------------------------- | ------------------------------------------------------ |
| `acore-font`    | `iconfont.css` 中的 `.acore-font` 规则    | 字体族 class，声明 font-family 和基础样式              |
| `icon-{图标名}` | `iconfont.css` 中的 `.icon-*:before` 规则 | 具体图标，通过 `content` 属性指向字体中的 unicode 码点 |

字体族 class 名称（如 `acore-font`）取决于 iconfont 项目的 `font-family` 设置，不同项目可能不同，以生成的 `iconfont.css` 中 `@font-face` 的 `font-family` 值为准。

### 样式控制

- **大小**：用 Tailwind `text-[Npx]` 或 `text-{size}`，如 `text-[32px]`、`text-xl`
- **颜色**：用 Tailwind `text-{color}`，如 `text-blue-600`、`text-gray-400`
- **其他**：`shrink-0`（防止 flex 容器中缩小）、`leading-none`（行高）

### 完整使用示例

```vue
<template>
  <!-- 基本用法 -->
  <i class="acore-font icon-Dify" />

  <!-- 带大小和颜色 -->
  <i class="acore-font icon-Dify text-[32px] text-blue-600" />

  <!-- 在 flex 容器中防止缩小 -->
  <i class="acore-font icon-Dify shrink-0 text-[32px] leading-none text-blue-600" title="已同步至Dify" />

  <!-- 条件渲染 -->
  <i v-if="showIcon" class="acore-font icon-Setting text-lg text-gray-500" />
</template>
```

### 如何查看可用图标

打开 `src/assets/iconfont/iconfont.css`，所有 `.icon-*:before` 规则即为可用图标列表。例如：

```css
.icon-Dify:before {
  content: '\e76e';
}
```

对应使用 `class="acore-font icon-Dify"`。
