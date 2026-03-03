# 本地 Mock 文件

为 Vue 组件中的接口调用生成本地 Mock 数据文件，在组件旁边创建独立的 `.ts` 文件并集成到组件中。适用于接口未就绪时的前端本地调试。

## 参数

| 参数       | 必填 | 说明                    | 示例                                        |
| ---------- | ---- | ----------------------- | ------------------------------------------- |
| `file`     | ✅   | Vue 组件文件            | `@src/modules/agent/linkTrans/index.vue`    |
| `api`      | ✅   | 需要 mock 的接口方法名  | `getShopMallServiceGshopSalesGetSaleSource` |
| `mockName` | ❌   | mock 文件名（不含后缀） | `saleSource`（默认根据接口名推断）          |
| `count`    | ❌   | 生成数据条数            | `5`                                         |

## 执行步骤

### 1. 定位接口调用

在指定的 Vue 文件中找到接口调用位置。

### 2. 查找接口类型定义

根据接口方法名，在 `@/api/xxx/controller` 中找到对应的请求参数类型和响应数据类型。

### 3. 创建 Mock 文件

在 Vue 文件同级目录下创建 mock 文件，命名规则：`mock${PascalCase(mockName)}.ts`

```
src/modules/agent/linkTrans/
├── index.vue
├── mockSaleSource.ts  ← 新建的 mock 文件
└── components/
```

Mock 文件内容：

```ts
import type { GetShopMallServiceGshopSalesGetSaleSourceResult } from '@/api/shop/controller';

/** Mock: 商品素材来源数据 */
export const mockSaleSourceData: GetShopMallServiceGshopSalesGetSaleSourceResult['data'] = {
  imgs: [
    { id: 1, url: 'https://picsum.photos/800/800?random=1' },
    { id: 2, url: 'https://picsum.photos/800/800?random=2' },
  ],
  content: '<div><img src="https://picsum.photos/800/1200?random=3" /></div>',
};
```

### 4. 修改 Vue 文件

**方式一：注释原调用，直接使用 mock 数据（推荐）**

```ts
import { mockSaleSourceData } from './mockSaleSource';

// TODO: 移除 mock，恢复接口调用
// const { data } = await getShopMallServiceGshopSalesGetSaleSource({ sale_id: formData.value.saleId });
const data = { data: mockSaleSourceData };
```

**方式二：保留接口调用，覆盖返回数据**

```ts
import { mockSaleSourceData } from './mockSaleSource';

const { data } = await getShopMallServiceGshopSalesGetSaleSource({ sale_id: formData.value.saleId });

// TODO: 移除 mock 数据覆盖
data.data = mockSaleSourceData;
```

## 文件命名规则

| 接口方法名                                  | 推荐 mockName | 文件名              |
| ------------------------------------------- | ------------- | ------------------- |
| `getShopMallServiceGshopSalesGetSaleSource` | `saleSource`  | `mockSaleSource.ts` |
| `postGmpV1CrowdList`                        | `crowdList`   | `mockCrowdList.ts`  |
| `getBasicV1UserInfo`                        | `userInfo`    | `mockUserInfo.ts`   |

## 注意事项

1. **类型安全**：mock 文件必须正确导入接口返回类型，确保类型安全
2. **TODO 标记**：修改处添加 `TODO` 注释，方便后续移除 mock
3. **不提交 mock 文件**：开发完成后记得删除 mock 文件和相关修改
4. **保持原有逻辑**：mock 数据结构要符合后续代码的使用方式

## 恢复接口调用

开发完成后：

1. 删除 mock 文件（如 `mockSaleSource.ts`）
2. 移除 Vue 文件中的 mock import
3. 取消注释原接口调用
4. 删除 mock 相关的临时代码
