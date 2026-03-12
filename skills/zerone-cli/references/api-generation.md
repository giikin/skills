# API 后端接口生成

通过 `zerone api` 命令从 Apifox 文档自动生成 TypeScript 接口代码（包含 controller、interface 类型定义、useAxios hooks）。

## 场景一：更新已有模块的接口

当用户提到"更新 xxx 的接口"、"重新生成 xxx 接口"，直接执行对应脚本：

```bash
pnpm api:{模块名}
```

脚本对应关系在 `package.json` 的 `scripts` 中，格式为：

```
"api:{模块名}": "zerone api -d -p ./src/api/{模块名}"
```

执行前先确认 `package.json` 中存在该脚本。如果不存在，提示用户是否需要先创建该 API 模块。

---

## 场景二：创建新的 API 模块

生成一整套标准的 API 模块代码，包括 request 配置、swagger 配置、环境变量和 npm script。

### 参数

| 参数          | 必填 | 说明                                     | 示例                         |
| ------------- | ---- | ---------------------------------------- | ---------------------------- |
| `folderName`  | 是   | 模块文件夹名（驼峰命名）                 | `gstore`、`materialComposer` |
| `projectId`   | 否   | Apifox 项目 ID（不传则置为空）           | `7479596`                    |
| `requestPath` | 否   | 自定义请求路径（覆盖 `.env` 中的默认值） | `/guard`                     |

如果未传入 `folderName`，**停止执行**并提示：

> ⚠️ 请传入文件夹名！用法：`生成 API 模块 <folderName> [projectId] [requestPath]`

### 用法示例

```
生成 API 模块 gstore 7479596
生成 API 模块 payment
生成 API 模块 shop 7479596 /guard
```

### 环境变量名转换规则

`folderName` 转换为大写环境变量名的规则：

- 驼峰 → 大写 + 下划线分隔：`materialComposer` → `MATERIAL_COMPOSER`
- 连字符 → 下划线：`material-composer` → `MATERIAL_COMPOSER`
- 纯小写 → 直接大写：`gstore` → `GSTORE`

### 步骤 1：参数校验

检查 `folderName` 是否传入，未传则停止。

### 步骤 2：创建 `src/api/{folderName}/request.ts`

```ts
import { createAxios } from 'giime';

const { service } = createAxios({
  baseURL: import.meta.env.VITE_BASE_XXXX_API, // XXXX 替换为 folderName 的大写形式
  successCode: 200,
});

export default service;
```

`{FOLDER_NAME_UPPER}` 按环境变量名转换规则替换。

### 步骤 3：创建 `src/api/{folderName}/swagger.config.json`

```json
{
  "docsUrl": "https://genapi-giime.giikin.com/apifox?projectId={projectId}",
  "includeTags": [],
  "excludeTags": [],
  "axiosInstanceUrl": "@/api/{folderName}/request",
  "vueUseAxios": "giime"
}
```

- `{projectId}` 替换为实际值，未传则 `projectId=` 后面留空
- `{folderName}` 替换为实际文件夹名

### 步骤 4：修改 `.env` 添加环境变量

在 `.env` 文件中找到 `VITE_BASE_*_API` 变量分组（接口地址区域），在该分组末尾追加：

```
VITE_BASE_{FOLDER_NAME_UPPER}_API=/{folderName}
```

**如果用户传了 `requestPath`**（如 `/guard`），使用该值替代默认的 `/{folderName}`：

```
VITE_BASE_{FOLDER_NAME_UPPER}_API=/guard
```

这种情况常见于多个模块共用同一个网关路径的场景。

### 步骤 5：修改 `package.json` 添加 npm script

在 `scripts` 中的 `"api:"` 系列脚本之后添加：

```
"api:{folderName}": "zerone api -d -p ./src/api/{folderName}"
```

### 步骤 6：输出结果摘要

```
✅ API 模块 [{folderName}] 创建成功！

📁 生成的文件：
  - src/api/{folderName}/request.ts
  - src/api/{folderName}/swagger.config.json

📝 修改的文件：
  - .env（添加 VITE_BASE_{FOLDER_NAME_UPPER}_API）
  - package.json（添加 api:{folderName} script）

🚀 下一步：
  1. 在 Apifox 项目中配置好接口文档
  2. 运行 pnpm api:{folderName} 生成接口代码
```

---

## swagger.config.json 配置参考

完整配置示例：

```json
{
  "docsUrl": "https://genapi-giime.giikin.com/apifox?projectId=7479596",
  "includeTags": ["基础服务/语种管理"],
  "excludeTags": [],
  "includePaths": ["/basic/v1/currency/listAll"],
  "excludePaths": ["/basic/v1/language/remove"],
  "axiosInstanceUrl": "@/api/gstore/request",
  "vueUseAxios": "giime"
}
```

### 字段说明

| 字段               | 类型                | 必填 | 默认值            | 说明                                                                                                                |
| ------------------ | ------------------- | ---- | ----------------- | ------------------------------------------------------------------------------------------------------------------- |
| `docsUrl`          | `string`            | 是   | —                 | Swagger 文档地址，支持 HTTP 链接或本地 JSON 文件相对路径                                                            |
| `includeTags`      | `string[]`          | 否   | `[]`              | 只生成匹配的 tag 的接口，支持前缀匹配                                                                               |
| `excludeTags`      | `string[]`          | 否   | `[]`              | 排除匹配的 tag 的接口，支持前缀匹配                                                                                 |
| `includePaths`     | `string[]`          | 否   | `[]`              | 只生成指定路径的接口，精确匹配                                                                                      |
| `excludePaths`     | `string[]`          | 否   | `[]`              | 排除指定路径的接口，精确匹配                                                                                        |
| `axiosInstanceUrl` | `string`            | 否   | `@/utils/request` | 生成代码中 axios 实例的导入路径                                                                                     |
| `vueUseAxios`      | `boolean \| string` | 否   | —                 | 是否生成 useAxios hooks 文件。`true` 使用 `@vueuse/integrations/useAxios`，传字符串则自定义导入路径（如 `"giime"`） |

### docsUrl

文档数据来源，支持两种形式：

- **HTTP 链接**：如 `https://genapi-giime.giikin.com/apifox?projectId=7479596`，工具会通过 HTTP 请求获取 Swagger JSON
- **本地文件路径**：如 `./docs/api.json`，相对于 swagger.config.json 所在目录解析

### 接口过滤（includeTags / excludeTags / includePaths / excludePaths）

四个过滤字段组合使用，优先级从低到高：

1. **Tags 过滤**：先根据 `includeTags` / `excludeTags` 决定接口是否保留（支持前缀匹配，如 `"基础服务"` 可匹配 `"基础服务/语种管理"`）
2. **includePaths 强制包含**：匹配到的路径会覆盖 Tags 过滤结果，强制保留该接口
3. **excludePaths 强制排除**：优先级最高，匹配到的路径无论 Tags 结果如何都会被排除

空数组 `[]` 表示不过滤。`includeTags` 和 `excludeTags` 都为空时，所有 tag 的接口都会生成。

#### 过滤示例

只生成某个模块下的接口：

```json
{
  "includeTags": ["基础服务/语种管理"],
  "excludeTags": []
}
```

生成所有接口但排除某几个路径：

```json
{
  "includeTags": [],
  "excludeTags": [],
  "excludePaths": ["/basic/v1/language/remove", "/basic/v1/area/delete"]
}
```

按 tag 过滤，但额外包含某个特定路径：

```json
{
  "includeTags": ["用户管理"],
  "includePaths": ["/basic/v1/currency/listAll"]
}
```

仅生成某一个（或几个）特定接口：

```json
{
  "includeTags": [""],
  "includePaths": ["/basic/v1/currency/listAll"]
}
```

> `includeTags` 设为 `[""]` 会使所有 tag 都不匹配（空字符串不会前缀匹配任何 tag），从而排除全部接口；再通过 `includePaths` 强制包含指定路径，最终只生成这些路径对应的接口。

### vueUseAxios

控制是否为每个接口额外生成 `useXxx` 风格的 hooks 文件：

- **不传或 `false`**：不生成
- **`true`**：生成，useAxios 从 `@vueuse/integrations/useAxios` 导入
- **字符串**（如 `"giime"`）：生成，useAxios 从指定路径导入

## 注意事项

1. **folderName 命名规范**：推荐使用驼峰命名（如 `materialComposer`）
2. **projectId 来源**：从 Apifox 项目设置中获取
3. **重复检测**：如果 `src/api/{folderName}` 目录已存在，提示用户确认是否覆盖
4. **requestPath 使用场景**：当接口网关路径与文件夹名不一致时（如文件夹叫 `shop` 但走 `/guard` 网关），用 requestPath 覆盖
