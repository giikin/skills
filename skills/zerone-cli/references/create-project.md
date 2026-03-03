# 创建前端项目 (create-zerone)

使用 `pnpm create zerone` 快速创建前端项目，通过交互式命令行选择框架和模板。

## 执行命令

```bash
pnpm create zerone
```

也可以直接指定项目名和模板跳过交互：

```bash
pnpm create zerone my-project --template vue-giime-ts
```

该命令需要 `required_permissions: ["all"]`。

## 交互流程

执行后会依次提示：

1. **Project name** — 项目名称（默认 `zerone-project`）
2. **Overwrite?** — 如果目标目录已存在且非空，询问是否清空并继续
3. **Package name** — 如果项目名不符合 npm 包名规范，提示输入有效包名
4. **Select a framework** — 选择框架
5. **Select a variant** — 选择该框架下的模板变体

## 可用模板

### Vue 框架

| 模板名 | 显示名 | 说明 |
| --- | --- | --- |
| `vue-giime-ts` | giime-ts | **推荐** — Giime + Vue3 + TypeScript + Tailwind，公司标准前端项目模板 |
| `vue-tailwind-ts` | tailwind-ts | Vue3 + TypeScript + Tailwind CSS，不含 Giime 组件库 |
| `vue-admin-ts` | admin-ts | 后台管理模板，带路由、布局、权限等基础功能 |
| `vue-crx-template` | vue-crx-template | Chrome 浏览器插件模板 |
| `vue-giime-electron-ts` | giime-electron-ts | Giime + Electron 桌面端应用模板 |

### Html 框架

| 模板名 | 显示名 | 说明 |
| --- | --- | --- |
| `html-vite-ts` | vite-ts | 纯 HTML + Vite + TypeScript，适合简单页面或 SDK |

### NestJS 框架

| 模板名 | 显示名 | 说明 |
| --- | --- | --- |
| `nestjs` | TypeScript | NestJS 后端项目模板 |

## 创建后的操作

项目创建完成后，终端会提示：

```bash
cd my-project
pnpm install
pnpm dev
```

## 使用场景

### 用户要求创建项目

告知用户执行命令，由用户在终端中交互选择模板：

```bash
pnpm create zerone
```

### 用户指定了模板

可以通过 `--template` 或 `-t` 参数跳过交互：

```bash
pnpm create zerone my-app --template vue-giime-ts
pnpm create zerone my-app -t vue-tailwind-ts
```

### 用户不确定选哪个模板

给出推荐建议：
- **公司内部业务项目** → `vue-giime-ts`（含 Giime 组件库、Element Plus、Tailwind）
- **轻量项目/不需要 UI 组件库** → `vue-tailwind-ts`
- **后台管理系统** → `vue-admin-ts`
- **Chrome 插件** → `vue-crx-template`
- **桌面端应用** → `vue-giime-electron-ts`
- **简单 HTML 页面** → `html-vite-ts`
- **后端 Node.js 服务** → `nestjs`

## 注意事项

- 该命令是交互式的，需要用户在终端中手动选择，AI 无法代替用户完成交互
- 如果 pnpm 版本不支持 `create` 命令，尝试 `npx create-zerone`
- 模板列表来自 `@zeronejs/cli` 包，随 CLI 版本更新可能新增模板
