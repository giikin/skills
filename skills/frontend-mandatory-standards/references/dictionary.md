# 字典模块使用规范

## 概述

字典模块用于管理业务中的枚举数据（如下拉选项、状态标签等）。每个业务模块应该有一个统一的字典 Store 来管理该模块所需的所有字典数据。

## 核心原则

1. **一个模块一个字典 Store**：每个 `modules/xxx` 下基本只需要创建一个字典 Store
2. **集中管理**：模块内所有字典编码和选项都在同一个 Store 中定义
3. **按需加载**：只加载当前模块需要的字典数据

## 文件位置

字典 Store 文件应放置在模块的 `stores` 目录下：

```
modules/
  └── xxx/                      # 业务模块
      └── stores/
          └── useDictionaryStore.ts  # 字典 Store
```

当子模块较大或明确指定在某个子模块添加字典时，也可以在子模块下创建字典 Store：

```
modules/
  └── xxx/                      # 业务模块
      ├── stores/
      │   └── useDictionaryStore.ts  # 模块级字典 Store
      └── subModule/            # 子模块
          └── stores/
              └── useDictionaryStore.ts  # 子模块级字典 Store
```

## 标准写法

```ts
import { useDictionary } from 'giime';
import { defineStore } from 'pinia';

/**
 * xxx模块字典 Store
 * @description 管理xxx模块共用的字典数据
 */
export const useXxxDictionaryStore = defineStore('xxxDictionary', () => {
  /** 字典编码常量 */
  const dictCodes = {
    /** 字典项说明1 */
    dictName1: 'dict_code_from_backend_1',
    /** 字典项说明2 */
    dictName2: 'dict_code_from_backend_2',
  } as const;

  /** 使用 giime 字典 hook */
  const { dictionary, isLoading } = useDictionary([dictCodes.dictName1, dictCodes.dictName2]);

  /** 字典选项1 */
  const dictName1Options = computed(() => dictionary.value[dictCodes.dictName1] || []);

  /** 字典选项2 */
  const dictName2Options = computed(() => dictionary.value[dictCodes.dictName2] || []);

  return {
    /** 字典数据 */
    dictionary,
    /** 加载状态 */
    isLoading,
    /** 字典选项1 */
    dictName1Options,
    /** 字典选项2 */
    dictName2Options,
  };
});
```

## 使用方式

在组件中使用字典 Store：

```ts
import { useXxxDictionaryStore } from '@/modules/xxx/stores/useDictionaryStore';

const dictionaryStore = useXxxDictionaryStore();

// 在模板中使用
// <gm-select :options="dictionaryStore.dictName1Options" />
```

## 新增字典项

当需要新增字典时，按以下步骤操作：

1. 在 `dictCodes` 对象中添加新的字典编码常量
2. 在 `useDictionary` 的参数数组中添加新的编码
3. 添加对应的 computed 选项
4. 在 return 中导出新的选项

## 命名规范

| 项目 | 规范 |
|------|------|
| dictCodes 键名 | camelCase，描述字典用途 |
| Store 名称 | `use[模块名]DictionaryStore` |
| defineStore id | `[模块名]Dictionary` |
| 选项变量 | `[字典名]Options` |

## 注意事项

1. 字典编码值需要从后端字典管理系统获取
2. 每个字典项都应添加 JSDoc 注释说明用途
3. 使用 `as const` 确保类型安全
4. 选项的 computed 需要提供默认空数组，避免 undefined
