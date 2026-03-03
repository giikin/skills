# 组件拆分规范

本规范指导如何将大型页面组件拆分为更小、更可维护的单元。

## 项目整体目录结构

```
src/
├── api/                         # API 接口（自动生成，不新建不删除）
│   ├── xxx/                     #   按业务域划分
│   │   ├── controller/          #     接口函数 + useAxios 封装
│   │   └── interface/           #     类型定义
│   └── ...
├── assets/                      # 静态资源（图标、字体、logo）
├── components/                  # 全局公共组件
├── composables/                 # 全局 composables（跨模块复用）
├── modules/                     # 业务模块（核心开发目录）
├── router/                      # 路由配置
│   ├── index.ts                 #   import.meta.glob 自动收集
│   └── modules/                 #   按模块划分路由文件
├── stores/                      # 全局 Pinia store
├── typings/                     # 全局类型声明（.d.ts）
├── utils/                       # 全局工具函数
├── views/                       # 页面级组件（路由 wrapper）
├── App.vue
└── main.ts
```

### 全局 vs 模块

| 位置              | 说明                       | 示例                                                        |
| ----------------- | -------------------------- | ----------------------------------------------------------- |
| `src/components/` | 跨模块共享的公共组件       | `SvgIcon.vue`、`DetailLoadingStatus/`、`ListLoadingStatus/` |
| `src/stores/`     | 跨模块共享的全局状态       | `useCategoryStore.ts`、`useTagStore.ts`                     |
| `src/utils/`      | 跨模块共享的工具函数       | 文件操作、URL 处理、通用 helper 等                          |
| `src/typings/`    | 全局类型声明               | `global.d.ts`                                               |
| `modules/xxx/`    | 模块内部的组件、逻辑、状态 | 详见下方                                                    |

## 标准模块目录结构

### 典型 CRUD 模块

具有列表页 + 详情页的标准业务模块推荐遵循以下结构：

```
modules/xxx/
├── index.vue                    # 列表主页面（编排子组件、管理数据）
├── DetailIndex.vue              # 详情页（如需要）
├── components/                  # UI 子组件
│   ├── Search.vue               #   搜索区域
│   ├── TableToolbar.vue         #   表格工具栏
│   ├── XxxCard.vue              #   实体卡片
│   ├── NewXxxCard.vue           #   新增入口卡片
│   └── XxxFormDialog.vue        #   表单弹窗
└── composables/                 # 配置与逻辑复用
    └── useXxxOptions.ts         #   模块配置
```

### 复杂模块（含状态管理、类型、布局）

当模块涉及多种功能（状态管理、自定义类型、专属布局等）时，可按需扩展目录：

```
modules/xxx/
├── index.vue                    # 主页面
├── ShareIndex.vue               # 其他入口页面（如需要）
├── components/                  # UI 子组件
│   ├── MainContent.vue
│   ├── InputArea.vue
│   ├── inputArea/               #   功能聚合子目录（camelCase）
│   │   ├── ConfigDrawer.vue
│   │   ├── OptionSelector.vue
│   │   └── useInputFileUpload.ts  # 仅被此子目录使用的 hook
│   └── ...
├── composables/                 # 配置与逻辑复用
│   ├── useXxxOptions.ts
│   └── useXxxShareOptions.ts
├── stores/                      # 模块状态管理
│   ├── useXxxStore.ts
│   └── useSessionStore.ts
├── types/                       # 模块类型定义
│   └── index.ts
└── layout/                      # 模块专属布局（如需要）
    ├── XxxLayout.vue
    └── LeftSidebar.vue
```

### 嵌套子模块

当一个模块包含多个独立子页面时，使用子模块组织：

```
modules/xxx/
├── composables/                 # 子模块共享的配置
│   ├── useXxxOptions.ts
│   └── types.ts                 #   共享类型
├── stores/                      # 子模块共享的状态
│   └── useDictionaryStore.ts
├── subModuleA/                  # 子模块 A
│   ├── index.vue
│   ├── components/
│   │   ├── Search.vue
│   │   ├── Table.vue
│   │   ├── EditDialog.vue
│   │   └── EditForm.vue
│   └── composables/
│       └── useSubModuleAOptions.ts
└── subModuleB/                  # 子模块 B
    ├── index.vue
    ├── components/
    └── composables/
```

## 入口页面命名约定

| 文件名              | 用途            | 说明                             |
| ------------------- | --------------- | -------------------------------- |
| `index.vue`         | 列表/主入口页面 | 每个模块的默认入口               |
| `DetailIndex.vue`   | 详情页面        | 需要独立详情页的模块             |
| `DocumentIndex.vue` | 子域页面        | 模块下某个子域的独立页面         |
| `ShareIndex.vue`    | 分享页面        | 需要对外分享功能的模块           |

## 组件命名约定

推荐遵循以下统一的组件命名模式：

| 命名模式                 | 用途         | 示例                                         |
| ------------------------ | ------------ | -------------------------------------------- |
| `Search.vue`             | 搜索区域     | 通用搜索栏组件                               |
| `Table.vue`              | 表格主体     | 数据表格                                     |
| `TableToolbar.vue`       | 表格工具栏   | 表格上方的操作按钮区                         |
| `{Entity}Card.vue`       | 实体卡片     | `UserCard.vue`、`ProductCard.vue`            |
| `New{Entity}Card.vue`    | 新增入口卡片 | `NewUserCard.vue`、`NewProductCard.vue`      |
| `{Entity}FormDialog.vue` | 表单弹窗     | `UserFormDialog.vue`、`ProductFormDialog.vue` |
| `{Xxx}Dialog.vue`        | 功能弹窗     | `ShareDialog.vue`、`RenameDialog.vue`        |
| `{Xxx}Drawer.vue`        | 抽屉         | `ConfigDrawer.vue`、`DetailDrawer.vue`       |
| `{Xxx}Panel.vue`         | 面板         | `MentionPanel.vue`、`PreviewPanel.vue`       |

## 拆分时机

当出现以下情况时，应考虑拆分组件：

- 非入口级别的 `.vue` 文件超过 **300 行**
- 模板中存在大量重复的卡片/区块结构
- 脚本中混杂了多种不相关的业务逻辑
- 组件承担了太多职责（违反单一职责原则）

## 拆分原则

### 按功能区域拆分 UI 组件

将页面按视觉区域拆分为独立组件：

```
modules/xxx/
├── index.vue                    # 主页面（组合所有组件）
└── components/
    ├── Search.vue               # 搜索区域
    ├── TableToolbar.vue         # 表格工具栏
    ├── Table.vue                # 表格区域
    ├── EditDialog.vue           # 编辑弹窗
    ├── EditForm.vue             # 编辑表单
    └── XxxCard.vue              # 卡片
```

### 按业务逻辑分组子目录

当某个功能区域有多个相关组件时，使用 **camelCase 命名**的子目录组织：

```
components/
├── messageInput/                # camelCase 子目录
│   ├── ConfigDrawer.vue         # PascalCase 组件
│   ├── OptionSelector.vue
│   └── useInputFileUpload.ts    # 仅被此目录使用的 hook 可放在这里
└── messageList/
    └── ScrollToBottomButton.vue
```

### 保持业务逻辑内聚

- **核心业务逻辑**（如提交、轮询、状态管理）保留在 `index.vue`
- **UI 展示逻辑**下沉到子组件
- **操作方法**（如下载、复制、预览）由使用该功能的组件自己实现

## Props 传递原则

### 避免过度传参

```vue
<!-- ❌ 传递过多 props 和 events -->
<Result :prop1="..." :prop2="..." :prop3="..." :prop4="..." :prop5="..." :prop6="..." @event1="..." @event2="..." @event3="..." />
```

```vue
<!-- ✅ 只传递核心数据，让组件自己计算和处理 -->
<Result :coreData="translationResult" :status="taskStatus" @retry="handleRetry" />
```

### 使用 Pinia 替代深层传递

当组件嵌套超过 2 层且需要共享状态时，应使用 Pinia 而不是层层传递 props：

```
index.vue → Result.vue → ResultSuccess.vue → ImageSection.vue
                    ↓
              使用 Pinia 共享状态，各组件直接访问 store
```

### 让组件自己实现操作

组件内部的操作（如复制、下载、预览）应该由组件自己实现，而不是通过 events 传递给父组件：

```vue
<!-- TextResultSection.vue -->
<script setup lang="ts">
import { GmCopy } from 'giime';

// 组件自己处理复制，不需要向父组件发送事件
// eslint-disable-next-line @typescript-eslint/no-unused-vars
const handleCopy = (text: string) => {
  GmCopy(text);
};
</script>
```

### 计算属性下沉

如果某个计算属性只在子组件中使用，应该放在子组件中计算，而不是在父组件计算后传递：

```vue
<!-- ResultSuccess.vue -->
<!-- eslint-disable @typescript-eslint/no-unused-vars -->
<script setup lang="ts">
const { translationResult, submittedFlags } = defineProps<{
  translationResult: TranslationResult;
  submittedFlags: string[];
}>();

// 在子组件内计算，而不是从父组件传入
const translatedTitle = computed(() => translationResult?.text?.find(g => g.flag === 'title'));
</script>
```

## Composables 命名模式

| 模式                        | 说明                         | 示例                                                       |
| --------------------------- | ---------------------------- | ---------------------------------------------------------- |
| `use{模块}Options.ts`       | 模块配置（选项、规则、常量） | `useUserOptions.ts`、`useOrderOptions.ts`                    |
| `use{模块}{功能}Options.ts` | 子功能配置                   | `useUserShareOptions.ts`、`useOrderDocumentOptions.ts`       |
| `use{模块}{功能}.ts`        | 非配置类逻辑封装             | `useEditorSDK.ts`、`useFileUpload.ts`                        |

### 仅被单个组件使用的 hook

如果一个 hook 仅被某个特定组件使用，可以放在该组件所在目录下，而不是模块的 `composables/`：

```
components/
└── messageInput/
    ├── MessageInput.vue
    └── useInputFileUpload.ts       # 仅被 MessageInput 使用
```

## 工具函数与类型抽离

### 全局工具函数

放在 `src/utils/` 目录，跨模块使用：

```ts
// src/utils/shareUrl.ts
export const getShareUrl = (shareCode: string) => {
  return urlJoin(VITE_BASE_URL, VITE_SHARE_PATH, shareCode);
};
```

### 业务常量和配置

放在模块的 `composables/useXxxOptions.ts` 中：

```ts
// composables/useImageTranslateOptions.ts
export const useImageTranslateOptions = () => {
  const carouselFlag = 'main_image';
  const detailFlag = 'detail_image';
  const pollingInterval = 2000;

  return { carouselFlag, detailFlag, pollingInterval };
};
```

### 模块类型定义

当模块内有较多自定义类型时，创建 `types/index.ts`：

```ts
// modules/xxx/types/index.ts
export type TaskStatus = 'pending' | 'running' | 'success' | 'failed';

export interface TaskItem {
  id: string;
  status: TaskStatus;
  name: string;
}
```

若模块较简单、类型较少，也可放在 `composables/types.ts`。

## 最终目标

拆分后的代码应该：

- **入口文件可适当放宽**：`index.vue` 作为编排入口，行数可超过 300 行但应尽量精简
- **子组件保持精简**：非入口 `.vue` 文件控制在 300 行以内
- **职责清晰**：每个组件只做一件事
- **易于维护**：修改某个功能只需要改动对应的组件
- **Props 精简**：子组件接收最少必要的数据
- **命名统一**：遵循项目已有的组件命名模式
