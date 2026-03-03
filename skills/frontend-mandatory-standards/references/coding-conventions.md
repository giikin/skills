# 编码约定详解

本文档是 SKILL.md 中「通用编码规则」的详细版本，包含完整示例。

## 注释规范

### JSDoc 注释

每个方法和变量定义时，请添加适量 JSDoc 注释，使用中文：

```ts
/** 用户信息 */
const userInfo = ref<UserInfo>();

/**
 * 提交表单
 * @description 校验表单后提交到后端
 */
const handleSubmit = async () => {
  // 校验表单
  await formRef.value?.validate();

  // 调用提交接口
  await submitExec({ ...formData.value });
};
```

## 函数声明规范

优先使用 `const` 箭头函数而非 `function` 声明，除非需要重载：

```ts
// ✅ 箭头函数
const handleSubmit = async () => {
  await submitExec();
};

// ✅ 需要重载时才用 function
function formatValue(value: string): string;
function formatValue(value: number): string;
function formatValue(value: string | number): string {
  return String(value);
}
```

## 守卫语句

使用条件提前退出，减少嵌套深度：

```ts
// ❌ 嵌套过深
const handleSaveNested = async () => {
  if (formData.value.name) {
    if (formData.value.age > 0) {
      if (!isSubmitting.value) {
        await saveExec();
      }
    }
  }
};

// ✅ 守卫语句提前退出
const handleSaveFlat = async () => {
  if (!formData.value.name) return;
  if (formData.value.age <= 0) return;
  if (isSubmitting.value) return;

  await saveExec();
};
```

## 函数参数设计

1-2 个主参数 + options 对象，避免参数超长和顺序混乱：

```ts
// ❌ 参数过多
const uploadFileBad = (file: File, bucket: string, path: string, compress: boolean, maxSize: number) => {};

// ✅ options 对象
interface UploadOptions {
  /** 存储桶 */
  bucket?: string;
  /** 上传路径 */
  path?: string;
  /** 是否压缩 */
  compress?: boolean;
  /** 最大文件大小（MB） */
  maxSize?: number;
}

const uploadFile = (file: File, options?: UploadOptions) => {};
```

## 工具库优先

减少造轮子，优先使用成熟的第三方库：

### 日期处理 — dayjs

```ts
import dayjs from 'dayjs';

// 格式化
dayjs(date).format('YYYY-MM-DD HH:mm:ss');

// 相对时间
dayjs(date).fromNow();

// 计算
dayjs().add(7, 'day').toDate();
```

### Vue 工具 — vueuse

```ts
// 安全的 onMounted
tryOnMounted(() => {
  fetchData();
});

// 事件监听（自动移除）
useEventListener(window, 'resize', handleResize);

// 本地存储（响应式）
const token = useLocalStorage('auth-token', '');

// 防抖
const debouncedFn = useDebounceFn(() => {
  search();
}, 300);
```

### 数据处理 — lodash-es

仅使用 ES 未提供的方法：

```ts
import { cloneDeep, compact, groupBy, uniqBy } from 'lodash-es';

// 深拷贝
const copied = cloneDeep(formData.value);

// 数组去重（按字段）
const uniqueList = uniqBy(list, 'id');

// 去除假值
const cleaned = compact([0, 1, '', null, 'hello']); // [1, 'hello']
```

## 现代 ES 特性

优先使用现代 JavaScript 特性：

```ts
// 可选链
const userName = response?.data?.user?.name;

// 空值合并
const pageSize = queryParams.size ?? 10;

// Promise.allSettled（不会因单个失败而中断）
const results = await Promise.allSettled(promises);

// Object.groupBy（ES2024）
const grouped = Object.groupBy(items, item => item.category);

// replaceAll
const cleaned = rawText.replaceAll('\n', ' ');

// Array.at（负索引）
const lastItem = arr.at(-1);
```

## 异步操作

优先 `async/await`，不用 Promise 链：

```ts
// ❌ Promise 链
const handleSubmitChain = () => {
  validate()
    .then(() => submitExec())
    .then(() => GmMessage.success('提交成功'))
    .catch(err => console.error(err));
};

// ✅ async/await
const handleSubmitAsync = async () => {
  await validate();
  await submitExec();

  GmMessage.success('提交成功');
};
```

## Git 操作

不要操作 git 命令，除非用户明确要求。包括：不要自动 `git add`、`git commit`、`git push` 等。
