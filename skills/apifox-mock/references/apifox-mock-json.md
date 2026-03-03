# Apifox Mock JSON

根据接口文件或类型定义，生成符合 Apifox Mock 语法（Mock.js）的 JSON 数据，可直接复制到 Apifox 的高级 Mock 功能中使用。适用于字段之间没有关联依赖的简单接口。

## 参数

| 参数    | 必填 | 说明                                  | 示例                                                                              |
| ------- | ---- | ------------------------------------- | --------------------------------------------------------------------------------- |
| `file`  | ✅   | 接口文件路径或类型定义文件路径        | `@src/api/mcp/controller/.../postAcoreToolMcpV1ServicePage.ts`                    |
| `count` | ❌   | 生成数据条数（针对列表类型），默认 12 | `10`                                                                              |

## 字段映射规则（Mock.js 语法）

| 字段类型/注释                | Mock 语法                                                                 |
| ---------------------------- | ------------------------------------------------------------------------- |
| 枚举值（如 1-A，2-B）        | `"fieldName|1-2": 1` 或 `"@pick([1, 2])"`                                 |
| `id` / `*Id`                 | `"id|+1": 1`                                                              |
| `name` / `*Name`（中文）     | `"@ctitle(3,8)"` 或 `"@cname"`                                            |
| `name`（英文标识符）         | `"prefix-@word(3,6)"`                                                      |
| `description` / `*Desc`      | `"@cparagraph(1,2)"`                                                       |
| `url` / `*Url` / `icon`      | `"@image('100x100')"` 或 `"https://picsum.photos/100"`                    |
| `*Time` / `*At`              | `"@datetime('yyyy-MM-dd HH:mm:ss')"`                                      |
| `status` / `is*`（布尔语义） | `"status|0-1": 1`                                                         |
| `*Color`                     | `"@pick(['#3B82F6', '#10B981', '#F59E0B', '#EF4444', '#8B5CF6'])"`         |
| `*Count` / `*Num`            | `"@natural(0, 100)"`                                                      |
| `*Order` / `*Sort`           | `"sortOrder|+1": 1`                                                       |
| `string` 类型               | `"@ctitle(2,6)"` 或 `"@word(3,8)"`                                        |
| `number` 类型               | `"@natural(1, 100)"`                                                      |
| `boolean` 类型              | `"field|1": true`                                                         |
| 数组类型                     | `"field|{count}": [{ ... }]`                                              |
| 分页结构                     | 包含 `records`、`total`、`size`、`current` 等字段                          |

### 特殊处理

- **枚举字段**：从注释中提取枚举值，使用 `@pick([])` 或 `|min-max` 语法
- **嵌套对象**：递归生成子对象的 mock 结构
- **关联数组**：如 `tags: TagVo[]`，生成 `"tags|1-4": [{ TagVo 结构 }]`

## 输出示例

```json
{
  "code": 200,
  "bizCode": 0,
  "data": {
    "records|12": [
      {
        "id|+1": 1,
        "name": "mcp-@word(3,6)",
        "createType|1-2": 1,
        "description": "@cparagraph(1)",
        "icon": "@image('100x100')",
        "status": 1,
        "tags|1-3": [
          {
            "id|+1": 100,
            "tagName": "@pick(['AI', '工具', '数据', 'API', '服务'])",
            "tagColor": "@pick(['#3B82F6', '#10B981', '#F59E0B', '#EF4444'])",
            "sortOrder|+1": 1
          }
        ],
        "createTime": "@datetime('yyyy-MM-dd HH:mm:ss')",
        "updateTime": "@datetime('yyyy-MM-dd HH:mm:ss')"
      }
    ],
    "total": 36,
    "size": 12,
    "current": 1,
    "searchCount": true
  },
  "msg": "success"
}
```

## Mock.js 语法参考

| 语法                          | 说明                       | 示例输出             |
| ----------------------------- | -------------------------- | -------------------- |
| `"name|min-max": value`       | 数字范围                   | 1-10 之间的整数      |
| `"name|+1": 1`                | 自增                       | 1, 2, 3...           |
| `"name|count": [item]`        | 重复数组元素               | 生成 count 个 item   |
| `"name|min-max": [item]`      | 随机数量数组               | min-max 个 item      |
| `"name|1": true`              | 随机布尔                   | true 或 false        |
| `@ctitle(min,max)`            | 中文标题                   | "测试标题"           |
| `@cparagraph(min,max)`        | 中文段落                   | "这是一段描述..."    |
| `@cname`                      | 中文姓名                   | "张三"               |
| `@natural(min,max)`           | 自然数                     | 0-100                |
| `@datetime('format')`         | 日期时间                   | "2024-01-15 12:30:00"|
| `@pick(array)`                | 从数组随机选一个           | 数组中的随机项       |
| `@image('size')`              | 图片占位符                 | 图片 URL             |
| `@word(min,max)`              | 英文单词                   | "hello"              |

## 注意事项

1. **枚举值识别**：优先从 JSDoc 注释中提取，格式如 `数字-描述` 或 `数字:描述`
2. **类型追溯**：需要递归查找所有嵌套类型的定义
3. **语义推断**：根据字段名推断合适的 mock 值类型
4. **分页结构**：识别 `Page*Vo` 或 `R*PageVo` 等分页响应，自动生成分页相关字段
5. **可空字段**：对于 `?` 可选字段，可以使用 `null` 或省略
