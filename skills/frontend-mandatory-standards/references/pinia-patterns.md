# Pinia 使用模式

本文档是 SKILL.md 中「Pinia 状态管理」的详细版本。

## 何时使用 Pinia

在开发初期就应该有使用 Pinia 的意识，但不要过度封装。

### 适合使用 Pinia 的场景

- 跨多层组件共享的状态（避免 props 层层传递）
- 需要在多个页面/组件间共享的数据
- 异步任务状态（如轮询状态、任务进度）
- 需要持久化或缓存的数据

### 不需要 Pinia 的场景

- 只在父子组件间传递的数据（用 props/emits）
- 只在当前组件使用的局部状态（用 ref）
- 简单的表单数据（用 v-model）

## Store 文件组织

```
modules/xxx/
├── stores/
│   └── useXxxStore.ts    # 模块专用 store
└── index.vue
```

## Store 标准写法

```ts
import { defineStore } from 'pinia';

/**
 * xxx模块 Store
 * @description 管理xxx模块的共享状态
 */
export const useXxxStore = defineStore('xxx', () => {
  /** 任务列表 */
  const taskList = ref<TaskItem[]>([]);

  /** 当前任务状态 */
  const taskStatus = ref<'idle' | 'running' | 'completed'>('idle');

  /** 是否正在轮询 */
  const isPolling = ref(false);

  /**
   * 重置 Store
   * @description 每次进入页面时调用，确保干净状态
   */
  const resetStore = () => {
    taskList.value = [];
    taskStatus.value = 'idle';
    isPolling.value = false;
  };

  return {
    taskList,
    taskStatus,
    isPolling,
    resetStore,
  };
});
```

## 使用规范

### 命名：体现业务含义

```ts
// ❌ 不要简写
const store = useXxxStore();

// ✅ 体现业务含义
const xxxStore = useXxxStore();
const chatStore = useChatStore();
const taskStore = useTaskStore();
```

### 进入页面时重置

在主组件初始化时调用 `resetStore()`，确保每次进入页面是干净状态：

```ts
const xxxStore = useXxxStore();

xxxStore.resetStore();
```

## 跨组件共享状态

当组件嵌套超过 2 层时，使用 Pinia 替代 props 层层传递：

```
index.vue → Result.vue → ResultSuccess.vue → ImageSection.vue
                    ↓
              使用 Pinia 共享状态，各组件直接访问 store
```

### index.vue（初始化 Store）

```ts
const taskStore = useTaskStore();

taskStore.resetStore();

// 提交任务后更新 store
const handleSubmit = async () => {
  const { data } = await submitExec({ data: formData.value });

  taskStore.taskStatus = 'running';
  taskStore.taskId = data.value?.data?.taskId;
};
```

### 子组件（直接访问 Store）

```ts
// ResultSuccess.vue — 不需要从父组件接收 props
const taskStore = useTaskStore();

// 直接使用 store 中的数据
const isCompleted = computed(() => taskStore.taskStatus === 'completed');
```

## 轮询任务模式

异步任务轮询是 Pinia 的典型使用场景：

```ts
export const useTaskStore = defineStore('task', () => {
  const taskId = ref('');
  const taskStatus = ref<'idle' | 'running' | 'completed' | 'failed'>('idle');
  const taskResult = ref<TaskResult | null>(null);
  const isPolling = ref(false);

  /** 开始轮询 */
  const startPolling = async () => {
    if (!taskId.value || isPolling.value) return;

    isPolling.value = true;
    taskStatus.value = 'running';

    const { pause } = useTimeoutPoll(async () => {
      const { data } = await getTaskStatusExec({ params: { id: taskId.value } });

      const status = data.value?.data?.status;

      if (status === 'completed' || status === 'failed') {
        taskStatus.value = status;
        taskResult.value = data.value?.data?.result ?? null;
        isPolling.value = false;
        pause();
      }
    }, 2000, { immediate: true });
  };

  const resetStore = () => {
    taskId.value = '';
    taskStatus.value = 'idle';
    taskResult.value = null;
    isPolling.value = false;
  };

  return {
    taskId,
    taskStatus,
    taskResult,
    isPolling,
    startPolling,
    resetStore,
  };
});
```
