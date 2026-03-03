# Giime Docs MCP 安装指南

`giime-docs` 是 Giime 组件库的文档 MCP 服务，提供 `get-giime-docs-sidebar` 和 `get-giime-component-doc` 两个工具，用于按需获取组件详细文档。

## 安装方式

在当前使用的 AI 工具的 MCP 配置文件中添加以下配置：

```json
{
  "mcpServers": {
    "giime-docs": {
      "url": "https://genapi-giime.giikin.com/mcp"
    }
  }
}
```

> 注意：不同 AI 工具的配置文件格式可能略有差异（如顶层 key 为 `mcpServers` 或 `servers`），请根据对应工具的文档调整。

## 验证

配置完成后，尝试调用以下工具验证是否生效：

- `get-giime-docs-sidebar` — 应返回完整的侧边栏配置
- `get-giime-component-doc({ link: '/components/base/button/' })` — 应返回 gm-button 的文档内容
