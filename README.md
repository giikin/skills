# Giikin Skills

Giikin 公司级 [Agent Skills](https://agentskills.io/home) 技能库，为团队 AI 编码助手提供统一的开发规范、工具使用指南和最佳实践。

## 技能列表

| 技能 | 描述 |
|------|------|
| [frontend-mandatory-standards](skills/frontend-mandatory-standards) | 前端开发规范 — Composition API、命名约定、接口调用、Pinia 状态管理、组件拆分等 |
| [giime-components](skills/giime-components) | Giime 组件库使用规范 — gm-* 组件增强特性、插件用法、MCP 文档获取 |
| [zerone-cli](skills/zerone-cli) | Zerone CLI 工具集 — API 代码生成、字体图标管理、项目脚手架、工作日志 |
| [apifox-mock](skills/apifox-mock) | Apifox Mock 数据生成规范 — Mock.js JSON、高级 Mock 脚本、本地 mock 文件 |
| [git-auto-commit-push](skills/git-auto-commit-push) | Git 自动暂存、提交并推送 — 智能分组提交、自动拉取推送 |

## 安装

```bash
npx skills add giikin/skills
```

安装指定技能：

```bash
npx skills add giikin/skills --skill='frontend-mandatory-standards'
```

全局安装（跨项目可用）：

```bash
npx skills add giikin/skills --skill='*' -g
```

安装到指定 Agent（如 Cursor）：

```bash
npx skills add giikin/skills --skill='*' -a cursor
```

### 安装选项

| 选项 | 说明 |
|------|------|
| `-g, --global` | 全局安装（安装到用户目录，跨项目可用） |
| `-a, --agent <agents...>` | 安装到指定 Agent（如 `cursor`、`claude-code`、`codex`） |
| `-s, --skill <skills...>` | 安装指定技能（`'*'` 表示全部） |
| `-l, --list` | 仅列出可用技能，不安装 |
| `--copy` | 复制文件而非创建符号链接 |
| `-y, --yes` | 跳过所有确认提示 |
| `--all` | 安装所有技能到所有 Agent，无需确认 |

### 安装范围

| 范围 | 标志 | 安装位置 | 适用场景 |
|------|------|----------|----------|
| **项目级** | （默认） | `./<agent>/skills/` | 随项目提交，团队共享 |
| **全局** | `-g` | `~/<agent>/skills/` | 跨项目全局可用 |

### 安装方式

| 方式 | 说明 |
|------|------|
| **Symlink**（推荐） | 创建符号链接，单一源，便于更新 |
| **Copy** | 创建独立副本，适用于不支持符号链接的环境 |

更多 CLI 用法参见 [skills CLI](https://github.com/vercel-labs/skills)。

## 常用命令

### `skills add` — 安装技能

```bash
# 列出仓库中可用的技能
npx skills add giikin/skills --list

# 安装所有技能
npx skills add giikin/skills --skill='*'

# 安装指定技能
npx skills add giikin/skills --skill frontend-mandatory-standards

# 安装多个技能
npx skills add giikin/skills --skill frontend-mandatory-standards --skill giime-components

# 安装到指定 Agent
npx skills add giikin/skills --skill='*' -a cursor -a claude-code

# 全局安装所有技能到所有 Agent（CI/CD 友好）
npx skills add giikin/skills --skill='*' -g -y

# 一键安装（所有技能 + 所有 Agent，无需确认）
npx skills add giikin/skills --all
```

### `skills list` — 查看已安装技能

```bash
# 查看所有已安装技能（项目 + 全局）
npx skills list

# 仅查看全局已安装的技能
npx skills ls -g

# 按 Agent 过滤
npx skills ls -a cursor
```

### `skills find` — 搜索技能

```bash
# 交互式搜索（fzf 风格）
npx skills find

# 按关键词搜索
npx skills find mock
```

### `skills check` / `skills update` — 检查与更新

```bash
# 检查技能是否有更新
npx skills check

# 更新所有已安装技能到最新版本
npx skills update
```

### `skills remove` — 移除技能

```bash
# 交互式移除（从已安装列表中选择）
npx skills remove

# 移除指定技能
npx skills remove frontend-mandatory-standards

# 移除多个技能
npx skills remove apifox-mock git-auto-commit-push

# 从全局移除
npx skills remove -g frontend-mandatory-standards

# 从指定 Agent 移除
npx skills remove -a cursor frontend-mandatory-standards

# 移除所有已安装技能
npx skills remove --all
```

| 选项 | 说明 |
|------|------|
| `-g, --global` | 从全局范围移除 |
| `-a, --agent` | 从指定 Agent 移除（`'*'` 表示全部） |
| `-s, --skill` | 指定要移除的技能（`'*'` 表示全部） |
| `-y, --yes` | 跳过确认提示 |
| `--all` | 等价于 `--skill '*' --agent '*' -y` |

### `skills init` — 创建新技能

```bash
# 在当前目录创建 SKILL.md 模板
npx skills init

# 在子目录中创建新技能
npx skills init my-skill
```

### 命令速查

| 命令 | 说明 |
|------|------|
| `npx skills add <source>` | 安装技能 |
| `npx skills list` | 查看已安装技能（别名：`ls`） |
| `npx skills find [query]` | 搜索技能 |
| `npx skills remove [skills]` | 移除技能（别名：`rm`） |
| `npx skills check` | 检查更新 |
| `npx skills update` | 更新技能 |
| `npx skills init [name]` | 创建技能模板 |

## 支持的 Agent

CLI 会自动检测你已安装的编码 Agent。如果未检测到，会提示你选择要安装到哪些 Agent。

| Agent | `--agent` 参数 | 项目路径 | 全局路径 |
|-------|----------------|----------|----------|
| Amp, Kimi Code CLI, Replit, Universal | `amp`, `kimi-cli`, `replit`, `universal` | `.agents/skills/` | `~/.config/agents/skills/` |
| Antigravity | `antigravity` | `.agent/skills/` | `~/.gemini/antigravity/skills/` |
| Augment | `augment` | `.augment/skills/` | `~/.augment/skills/` |
| Claude Code | `claude-code` | `.claude/skills/` | `~/.claude/skills/` |
| OpenClaw | `openclaw` | `skills/` | `~/.openclaw/skills/` |
| Cline | `cline` | `.agents/skills/` | `~/.agents/skills/` |
| CodeBuddy | `codebuddy` | `.codebuddy/skills/` | `~/.codebuddy/skills/` |
| Codex | `codex` | `.agents/skills/` | `~/.codex/skills/` |
| Command Code | `command-code` | `.commandcode/skills/` | `~/.commandcode/skills/` |
| Continue | `continue` | `.continue/skills/` | `~/.continue/skills/` |
| Cortex Code | `cortex` | `.cortex/skills/` | `~/.snowflake/cortex/skills/` |
| Crush | `crush` | `.crush/skills/` | `~/.config/crush/skills/` |
| Cursor | `cursor` | `.agents/skills/` | `~/.cursor/skills/` |
| Droid | `droid` | `.factory/skills/` | `~/.factory/skills/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/` | `~/.gemini/skills/` |
| GitHub Copilot | `github-copilot` | `.agents/skills/` | `~/.copilot/skills/` |
| Goose | `goose` | `.goose/skills/` | `~/.config/goose/skills/` |
| Junie | `junie` | `.junie/skills/` | `~/.junie/skills/` |
| iFlow CLI | `iflow-cli` | `.iflow/skills/` | `~/.iflow/skills/` |
| Kilo Code | `kilo` | `.kilocode/skills/` | `~/.kilocode/skills/` |
| Kiro CLI | `kiro-cli` | `.kiro/skills/` | `~/.kiro/skills/` |
| Kode | `kode` | `.kode/skills/` | `~/.kode/skills/` |
| MCPJam | `mcpjam` | `.mcpjam/skills/` | `~/.mcpjam/skills/` |
| Mistral Vibe | `mistral-vibe` | `.vibe/skills/` | `~/.vibe/skills/` |
| Mux | `mux` | `.mux/skills/` | `~/.mux/skills/` |
| OpenCode | `opencode` | `.agents/skills/` | `~/.config/opencode/skills/` |
| OpenHands | `openhands` | `.openhands/skills/` | `~/.openhands/skills/` |
| Pi | `pi` | `.pi/skills/` | `~/.pi/agent/skills/` |
| Qoder | `qoder` | `.qoder/skills/` | `~/.qoder/skills/` |
| Qwen Code | `qwen-code` | `.qwen/skills/` | `~/.qwen/skills/` |
| Roo Code | `roo` | `.roo/skills/` | `~/.roo/skills/` |
| Trae | `trae` | `.trae/skills/` | `~/.trae/skills/` |
| Trae CN | `trae-cn` | `.trae/skills/` | `~/.trae-cn/skills/` |
| Windsurf | `windsurf` | `.windsurf/skills/` | `~/.codeium/windsurf/skills/` |
| Zencoder | `zencoder` | `.zencoder/skills/` | `~/.zencoder/skills/` |
| Neovate | `neovate` | `.neovate/skills/` | `~/.neovate/skills/` |
| Pochi | `pochi` | `.pochi/skills/` | `~/.pochi/skills/` |
| AdaL | `adal` | `.adal/skills/` | `~/.adal/skills/` |

## License

MIT
