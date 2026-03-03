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

## 注意事项

1. **folderName 命名规范**：推荐使用驼峰命名（如 `materialComposer`）
2. **projectId 来源**：从 Apifox 项目设置中获取
3. **重复检测**：如果 `src/api/{folderName}` 目录已存在，提示用户确认是否覆盖
4. **requestPath 使用场景**：当接口网关路径与文件夹名不一致时（如文件夹叫 `shop` 但走 `/guard` 网关），用 requestPath 覆盖
