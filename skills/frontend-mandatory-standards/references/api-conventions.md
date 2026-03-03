# 接口调用完整指南

本文档是 SKILL.md 中「接口调用规范」的详细版本。

## 文件限制

`/src/api` 文件夹下的文件可以简单修改，**不能新建和删除**。接口文件由代码生成工具自动生成。

## 文件命名与结构

每个接口生成**两个文件**：

- **原始接口**：`postXxxYyyZzz.ts` — 直接调用 axios 的原始方法
- **useAxios 封装接口**：`usePostXxxYyyZzz.ts` — 使用 giime 的 `useAxios` 封装

文件名通过 "请求方法+路由地址" 生成：

| 请求                         | 文件名                                                     |
| ---------------------------- | ---------------------------------------------------------- |
| `POST /open/v1/system/list`  | `postOpenV1SystemList.ts` + `usePostOpenV1SystemList.ts`   |
| `GET /acore/v1/model/detail` | `getAcoreV1ModelDetail.ts` + `useGetAcoreV1ModelDetail.ts` |
| `DELETE /acore/v1/task/{id}` | `deleteAcoreV1TaskId.ts` + `useDeleteAcoreV1TaskId.ts`     |

## useAxios 封装接口（默认选择）

正常情况下**必须**使用 `useXxx` 版本的接口，它提供以下特性：

- 响应式数据：`data`、`isLoading`、`isFinished`、`error` 等
- 自动取消上次请求（防止竞态条件）
- 与 Vue 响应式系统无缝集成

```ts
/** 分页查询接口 */
const { exec: getAgentPageExec, isLoading, error } = usePostAgentV1AgentPage();

/**
 * 获取列表数据
 */
const getList = async () => {
  const { data } = await getAgentPageExec(queryParams.value);

  agentList.value = data.value?.data?.records || [];
  total.value = data.value?.data?.total || 0;
};
```

常见的解构方式：

```ts
// 只需要 exec
const { exec: submitFormExec } = usePostXxxV1Submit();

// 需要 exec + isLoading（用于 v-loading）
const { isLoading: isShareLoading, exec: getShareListExec } = useGetXxxV1List();

// 需要重命名避免冲突（多个接口共存）
const { exec: getShareDetailExec, isLoading: isLoadingShareDetail } = useGetChatAiChatV1ShareByShareCode();
const { exec: getAgentDetailExec } = useGetAgentV1AgentById();

// 需要 error（用于错误状态展示）
const { exec: getPageExec, isLoading: isPageLoading, error: pageError } = usePostXxxV1Page();
```

调用 exec 时，直接传入接口所需参数（参照接口类型定义），通过解构 `{ data }` 获取响应结果：

```ts
// GET 请求 — 参数直接传入
const { data: detailData } = await getDetailExec({ id: 'xxx' });

// POST 请求 — 传入请求体（与分页参数一起）
const { data: listData } = await getListExec({ current: 1, size: 10, keyword: '搜索词' });
```

## 原始接口（仅特殊场景）

**只有**以下场景才使用原始接口（不带 `use` 前缀）：

### 循环请求 / Promise.all

```ts
// ✅ 批量删除
const handleBatchDelete = async (ids: string[]) => {
  await Promise.all(ids.map(id => deleteAcoreToolMcpV1Service({ id })));

  GmMessage.success('批量删除成功');
};
```

### Promise.allSettled（部分失败不中断）

```ts
// ✅ 批量操作，允许部分失败
const handleBatchUpdate = async (items: UpdateItem[]) => {
  const results = await Promise.allSettled(items.map(item => postAcoreV1TaskUpdate({ data: item })));

  const failCount = results.filter(r => r.status === 'rejected').length;

  if (failCount > 0) {
    GmMessage.warning(`${failCount} 项更新失败`);
  }
};
```

**原因**：`useAxios` 封装的接口默认会取消上次未完成的请求，在循环调用时会导致只有最后一个请求生效。

## API 导入规则（强制）

### 正确的导入方式

```ts
// ✅ 接口方法和请求/响应类型：从 controller 导入
import type { PostGmpV1CrowdListInput } from '@/api/gmp/controller';
import { usePostGmpV1CrowdList } from '@/api/gmp/controller';
```

```ts
// ✅ 公共数据模型类型：从 interface 导入
import type { CrowdItemVo } from '@/api/gmp/interface';
```

### 错误的导入方式

```ts
// ❌ 禁止从具体文件导入
import type { CrowdItemVo } from '@/api/gmp/interface/apiTypes/CrowdItemVo';
import { usePostGmpV1CrowdList } from '@/api/gmp/controller/RenQunGuanLi/postGmpV1CrowdList';
```

### 重要说明

- 规则同时适用于方法导入和类型导入
- 所有接口和类型都应该从 `controller/index.ts` 或 `interface/index.ts` 统一导出
- 目的是降低代码耦合度，方便后期维护和重构
- Code Review 时应重点检查 API 的导入方式

## 使用流程

1. **查找方法**：如果提供了接口地址，第一步找到 `@/api/xxx/controller` 中定义的请求方法
2. **阅读类型**：仔细阅读接口文件中的类型定义（入参、出参），不要自己编参数
3. **选择版本**：根据选择规则决定使用 `useXxx` 版本或原始版本
4. **数据源抽离**：下拉框、单选框等数据源可从接口文档注释获取，获取后在 `useXxxOptions` 中抽离复用

## 错误处理

### 无需 try/catch

一般情况下不用处理 catch，控制台直接报错即可。`createAxios` 拦截器会自动弹出错误提示。

### 无需判断 code

后端正常响应码为 200/0，非正常码时 Giime 拦截器会自动 reject。因此业务代码中不需要 `if (code !== 200)` 之类的判断。

```ts
// ❌ 不需要判断 code
const handleSubmitBad = async () => {
  const res = await submitExec({ data: formData.value });

  if (res.data.value?.code === 200) {
    GmMessage.success('成功');
  }
};

// ✅ 直接使用，错误由拦截器处理
const handleSubmitGood = async () => {
  await submitExec({ data: formData.value });

  GmMessage.success('成功');
};
```

### 需要 finally 时

使用 try/finally（不写 catch），用于清理其他副作用：

```ts
const handleExport = async () => {
  try {
    isExporting.value = true;
    await exportExec({ data: params.value });

    GmMessage.success('导出成功');
  } finally {
    isExporting.value = false;
  }
};
```
