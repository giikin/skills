## 增删改查 代码模板，供你学习代码规范和风格

1. 代码模板中包含增删改查功能的基本实现，请根据实际需求进行调整。
2. 代码模板中的请求地址为示例地址，请根据实际情况修改为真实请求地址。
3. 代码模板中的参数和返回数据结构为示例，请根据实际情况进行调整。
4. 模块拆分完整，不能少文件
5. 如果项目中包含`modules` 和 `views`模块，那么下面的代码都应该出现在`modules`目录下
6. useAppOptions只是一个demo，请根据实际模块调整为`useXXXOptions`
7. 以下文件按包涵顺序创建，比如子组件完成后 再创建index.vue。下面的模板是关于 `应用管理` 相关功能

index.vue

```vue
<template>
  <gm-table-ctx class="p-5" :tableId="tableId">
    <h1 class="mb-6 font-bold">你的标题</h1>
    <!-- 搜索 -->
    <Search v-model:queryParams="queryParams" :showSearch="showSearch" @getList="getList" />
    <!-- 工具栏 -->
    <TableToolbar
      v-model:showSearch="showSearch"
      v-model:queryParams="queryParams"
      class="mb-3"
      :notSelected="notSelected"
      :total="total"
      :selectedIds="selectedIds"
      @openAddForm="editDialogRef?.openAddForm"
      @batchDelete="handleDelete()"
      @getList="getList"
    />
    <!-- 表格 -->
    <Table
      v-model:sortValue="sortValue"
      v-loading="isLoading"
      :listData="listData"
      @selectionChange="handleSelectionChange"
      @handleDelete="handleDelete"
      @openUpdateForm="editDialogRef?.openUpdateForm"
      @getList="getList"
    />
    <!-- 分页 -->
    <gm-table-pagination v-model:page="queryParams.current" v-model:limit="queryParams.size" :total="total" @pagination="getList" />
    <!-- 编辑弹窗 -->
    <EditDialog ref="editDialogRef" @getList="getList" />
  </gm-table-ctx>
</template>

<script setup lang="ts">
import { GmConfirmBox } from 'giime';
// import { useRoute } from 'vue-router';
import EditDialog from './components/EditDialog.vue';
import Search from './components/Search.vue';
import Table from './components/Table.vue';
import TableToolbar from './components/TableToolbar.vue';
import { useAppOptions } from './composables/useAppOptions';
import type { TableProSortValue } from 'giime';

// 接口
import type { SelectIsvAppReq, SelectIsvAppVo } from '@/api/open/interface';
import { postOpenV1IsvAppRemove, usePostOpenV1IsvAppPage } from '@/api/open/controller';

// const route = useRoute();
const editDialogRef = ref<InstanceType<typeof EditDialog>>();

const { tableId } = useAppOptions();
// 打开搜索模块
const showSearch = ref(true);
const selectedIds = ref<number[]>([]);
// 多选
const isMultiple = ref(false);
// 未选中
const notSelected = ref(true);

const total = ref(0);
// 列表请求参数
const queryParams = ref<SelectIsvAppReq>({
  corpId: 0,
  appName: undefined,
  authMode: undefined,
  appIdStr: undefined,
  serviceId: undefined,
  appIp: undefined,
  createTime: [],
  status: undefined,
  updateTime: [],
  /** 当前页 */
  current: 1,
  /** 每页显示条数 */
  size: 10,
});

const sortValue = ref<TableProSortValue[]>([]);
const listData = ref<SelectIsvAppVo[]>([]);
const { isLoading, exec: getListExec } = usePostOpenV1IsvAppPage();
/** 获取列表 */
const getList = async () => {
  const { data } = await getListExec({
    ...queryParams.value,
    sorts: sortValue.value,
  });

  listData.value = data.value?.data.records || [];
  total.value = data.value?.data.total || 0;
};

/** 多选框选中数据 */
const handleSelectionChange: (value: any[]) => any = (selection: SelectIsvAppVo[]) => {
  selectedIds.value = selection.map(item => item.id);
  isMultiple.value = selection.length !== 1;
  notSelected.value = !selection.length;
};

/** 删除按钮操作 */
const handleDelete = async (row?: SelectIsvAppVo) => {
  const selectIds = row ? [row.id] : selectedIds.value;

  GmConfirmBox({ message: `请确定是否删除，删除后不可恢复！` }, async () => {
    await postOpenV1IsvAppRemove({ ids: selectIds });
    GmMessage.success('删除成功');
    getList();
  });
};

onMounted(() => {
  getList();
});
</script>
```

components/Search.vue 如果无需级联选择器的话，无需selectedSysName相关代码

```vue
<template>
  <section>
    <gm-search-form v-show="showSearch" v-model:query-params="queryParams" @handle-query="handleQuery" @reset-query="resetQuery">
      <gm-search-form-input prop="appName" label="应用名称" />
      <gm-search-form-select prop="status" label="应用状态" :options="statusOptions" />
      <gm-search-form-select prop="authMode" label="鉴权模式" :options="authModeOptions" />
      <gm-search-form-cascader
        prop="subId"
        label="所属系统"
        :options="systemTreeStore.systemTree"
        :cascader-props="{ value: 'id', label: 'name', children: 'children', emitPath: false }"
      />
      <gm-search-form-date-picker prop="createTime" label="创建" type="daterange" />
    </gm-search-form>
  </section>
</template>
<script lang="ts" setup>
import { useAppOptions } from '../composables/useAppOptions';
import type { SelectIsvAppReq } from '@/api/open/interface';

defineProps<{
  showSearch: boolean;
}>();
const emit = defineEmits<{
  (e: 'getList'): Promise<any>;
}>();
const systemTreeStore = useSystemTreeStore();
const { authModeOptions, statusOptions } = useAppOptions();
// 列表请求参数
const queryParams = defineModel<SelectIsvAppReq>('queryParams', { required: true });

/** 搜索按钮操作 */
const handleQuery = () => {
  queryParams.value.current = 1;
  emit('getList');
};
/** 重置按钮操作 */
const resetQuery = () => {
  // queryRef.value?.resetFields();
  handleQuery();
};

defineExpose({
  queryParams,
});
</script>
```

components/TableToolbar

```vue
<template>
  <div class="mb-3">
    <gm-table-toolbar
      v-model:showSearch="showSearch"
      v-model:queryParams="queryParams"
      :tableId="tableId"
      :total="total"
      :selected-count="selectedIds.length"
      @getList="emit('getList')"
    >
      <gm-col :span="1.5">
        <gm-button plain @click="emit('openAddForm')">新增</gm-button>
      </gm-col>
      <gm-col :span="1.5">
        <gm-button plain :disabled="notSelected" @click="handleBatchModifyStatus(0)">禁用</gm-button>
      </gm-col>
      <gm-col :span="1.5">
        <gm-button plain :disabled="notSelected" @click="emit('batchDelete')">删除</gm-button>
      </gm-col>
    </gm-table-toolbar>
  </div>
</template>
<script lang="ts" setup>
import { useAppOptions } from '../composables/useAppOptions';
import type { SelectIsvAppReq } from '@/api/open/interface';
import { postOpenV1IsvAppModifyStatus } from '@/api/open/controller';

const props = defineProps<{
  notSelected: boolean;
  total: number;
  selectedIds: number[];
}>();
const emit = defineEmits<{
  (e: 'openAddForm'): void;
  (e: 'batchDelete'): void;
  (e: 'getList'): Promise<any>;
}>();

const showSearch = defineModel<boolean>('showSearch', { required: true });
const queryParams = defineModel<SelectIsvAppReq>('queryParams', { required: true });

const { tableId } = useAppOptions();
/** 批量启用禁用操作 */
const handleBatchModifyStatus = async (status: number) => {
  // const selectIds = row ? [row.id] : selectedIds.value;
  GmConfirmBox({ message: `确定要禁用该应用吗？` }, async () => {
    await postOpenV1IsvAppModifyStatus({ ids: props.selectedIds, status });

    GmMessage.success('禁用成功');
    emit('getList');
  });
};
</script>
```

components/Table.vue

```vue
<template>
  <div>
    <gm-table-pro
      v-model:sortValue="sortValue"
      :data="listData"
      :page="tableId"
      :selection="true"
      :height-offset="20"
      @selectionChange="emit('selectionChange', $event)"
      @sortChange="emit('getList')"
    >
      <gm-table-column-pro show-overflow-tooltip label="应用信息" align="left" prop="appName" min-width="330">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <div class="flex items-center gap-3">
            <gm-avatar
              shape="square"
              size="large"
              src="https://cube.elemecdn.com/9/c2/f0ee8a3c7c9638a54940382568c9dpng.png"
              class="h-[60px] w-[60px] shrink-0"
            />
            <div>
              <div>
                {{ row.appName }}
                <gm-copy-icon
                  :str="`应用名: ${row.appName}
应用ID: ${row.appIdStr}
client_id: ${row.clientId}
client_secret: ${row.clientSecret}`"
                />
              </div>
              <div class="mt-1 text-[#999]">应用ID：{{ row.appIdStr }}</div>
            </div>
          </div>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro label="应用描述" align="left" prop="appDesc" width="250">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <div>
            <el-tooltip effect="dark" placement="top" :content="row.appDesc">
              <template #content>
                <div class="max-w-[300px] whitespace-break-spaces">{{ row.appDesc }}</div>
              </template>
              <div class="line-clamp-1">{{ row.appDesc }}</div>
            </el-tooltip>
          </div>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro show-overflow-tooltip label="应用IP" align="left" prop="appIp" width="140">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <div v-for="(item, index) in row.appIp?.split(',') ?? []" :key="index">
            {{ item }}
          </div>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro label="鉴权模式" align="left" prop="authMode" width="140">
        <template #default="scope">
          <template v-for="item in authModeOptions">
            <template v-if="scope.row.authMode === item.value">
              <gm-tag :key="item.label" :type="item.tageType">{{ item.label }}</gm-tag>
            </template>
          </template>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro label="应用状态" align="left" prop="status" width="120">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <el-switch
            v-model="row.status"
            inline-prompt
            class="ml-2"
            size="large"
            active-text="正常"
            inactive-text="禁用"
            :active-value="1"
            :inactive-value="0"
            :loading="modifyStatusLoading"
            :before-change="() => beforeChangeStatus(row)"
          />
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro label="client_id" align="left" prop="clientId" width="180" />
      <gm-table-column-pro label="client_secret" align="left" prop="clientSecret" width="180" />
      <gm-table-column-pro isSort label="创建时间" align="left" prop="createTime" width="180" min-width="180">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <span>{{ row.createTime }}</span>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro isSort label="更新时间" align="left" prop="updateTime" width="180" min-width="180">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <span>{{ row.updateTime }}</span>
        </template>
      </gm-table-column-pro>
      <gm-table-column-pro prop="" type="edit">
        <template #default="{ row }: { row: SelectIsvAppVo }">
          <gm-operate-button label="编辑" prop="edit" type="primary" class="mb-1 mr-1" @click="emit('openUpdateForm', row)" />
          <gm-operate-button label="删除" prop="delete" type="primary" class="mb-1" @click="emit('handleDelete', row)" />
          <gm-operate-button label="开通服务" prop="subscribe" type="primary" :disabled="row.status === 0" @click="handleSubscribe(row)" />
        </template>
      </gm-table-column-pro>
    </gm-table-pro>
    <SubscribeServiceDialog ref="subscribeServiceDialogRef" :listData="listData" @getList="emit('getList')" />
  </div>
</template>
<script setup lang="ts">
import { useLoading } from 'giime';
import { useAppOptions } from '../composables/useAppOptions';
import SubscribeServiceDialog from './SubscribeServiceDialog.vue';
import type { TableProSortValue } from 'giime';
import type { SelectIsvAppVo } from '@/api/open/interface';
import { postOpenV1IsvAppModifyStatus } from '@/api/open/controller';

defineProps<{
  listData: SelectIsvAppVo[];
}>();
const emit = defineEmits<{
  (e: 'selectionChange', value: any[]): any;
  (e: 'openUpdateForm', row: SelectIsvAppVo): any;
  (e: 'handleDelete', row: SelectIsvAppVo): any;
  (e: 'getList'): Promise<any>;
}>();

const sortValue = defineModel<TableProSortValue[]>('sortValue', { required: true });

const subscribeServiceDialogRef = ref<InstanceType<typeof SubscribeServiceDialog>>();

const { tableId, authModeOptions } = useAppOptions();

const { isLoading: modifyStatusLoading, exec: modifyStatusExec } = useLoading(postOpenV1IsvAppModifyStatus);
const beforeChangeStatus = async (row: SelectIsvAppVo) => {
  await modifyStatusExec({ ids: [row.id], status: row.status === 1 ? 0 : 1 });

  return true;
};

const handleSubscribe = (row: SelectIsvAppVo) => {
  subscribeServiceDialogRef.value?.openDialogForm(row);
};
</script>
```

components/EditDialog.vue

```vue
<template>
  <!-- 添加/修改对话框 -->
  <gm-dialog v-model="editDialogVisible" append-to-body width="500px" :title="dialogTitle" @closed="cancel">
    <EditForm ref="editFormRef" v-model:editForm="editForm" @getList="emit('getList')" />
    <template #footer>
      <div class="dialog-footer">
        <gm-button type="primary" @click="submitForm">确 定</gm-button>
        <gm-button @click="cancel">取 消</gm-button>
      </div>
    </template>
  </gm-dialog>
</template>
<script lang="ts" setup>
import { resetObject } from 'giime';
import { cloneDeep } from 'lodash-es';
import { AppEditDialogType } from '../composables/types';
import { useAppOptions } from '../composables/useAppOptions';
import EditForm from './EditForm.vue';
import type { AddIsvAppReq, SelectIsvAppVo } from '@/api/open/interface';
import { useCompanyStore } from '@/modules/company/stores/companyStore';
import { postOpenV1IsvAppModify, postOpenV1IsvAppSave } from '@/api/open/controller';

const emit = defineEmits<{
  (e: 'getList'): Promise<any>;
}>();

const companyStore = useCompanyStore();

const { dialogTitleOptions } = useAppOptions();
/**弹窗标题 */
const dialogTitle = computed(() => dialogTitleOptions.find(item => item.value === dialogType.value)?.label);
/**
 * 编辑表单
 */
const editDialogVisible = ref(false);
const dialogType = ref(AppEditDialogType.ADD);
const editFormRef = ref<InstanceType<typeof EditForm>>();
const formId = ref(0);
const defaultEditForm: AddIsvAppReq = {
  corpId: '',
  appName: '',
  authMode: 1,
  appHome: undefined,
  appIp: '',
  appDesc: '',
};
const editForm = ref(cloneDeep(defaultEditForm));

/** 新增按钮操作 */
const openAddForm = () => {
  editDialogVisible.value = true;
  dialogType.value = AppEditDialogType.ADD;
};

/** 取消按钮 */
function cancel() {
  editDialogVisible.value = false;
  reset();
}
/** 表单重置 */
function reset() {
  formId.value = 0;
  // 表单加载可能有延迟   会导致重置失败
  editFormRef.value?.resetFields?.();
  // 手动重置
  editForm.value = cloneDeep(defaultEditForm);
}

/** 修改按钮操作 */
const openUpdateForm = (row: SelectIsvAppVo) => {
  reset();
  dialogType.value = AppEditDialogType.EDIT;
  // 如果需要从接口获取详情
  // const { data } = await getItemById({ id: selectId });
  formId.value = row.id;
  resetObject(editForm.value, row);
  editDialogVisible.value = true;
};

/** 提交按钮 */
const submitForm = async () => {
  await editFormRef.value?.validate?.();

  if (!companyStore.activeCompany?.corpId) {
    return;
  }

  if (!formId.value) {
    // 新增
    await postOpenV1IsvAppSave({
      ...editForm.value,
      corpId: companyStore.activeCompany.corpId,
    });
  } else {
    // 修改
    await postOpenV1IsvAppModify({ ...editForm.value, id: formId.value });
  }

  if (dialogType.value === AppEditDialogType.ADD) {
    companyStore.getList();
  }
  GmMessage.success('操作成功');
  editDialogVisible.value = false;
  emit('getList');
};

defineExpose({
  openAddForm,
  openUpdateForm,
});
</script>
```

components/EditForm.vue

```vue
<template>
  <div>
    <gm-form ref="editFormRef" label-width="auto" :model="editForm" :rules="rules">
      <gm-form-item label="应用名称" prop="appName">
        <gm-input v-model="editForm.appName" show-word-limit placeholder="请输入应用名称" maxlength="50" />
      </gm-form-item>
      <gm-form-item label="应用描述" prop="appDesc">
        <gm-input v-model="editForm.appDesc" show-word-limit type="textarea" placeholder="请输入应用描述" maxlength="200" :rows="4" />
      </gm-form-item>
      <gm-form-item label="应用IP" prop="appIp">
        <gm-input v-model="editForm.appIp" show-word-limit placeholder="请输入应用IP" maxlength="100" />
      </gm-form-item>
      <gm-form-item label="应用首页地址" prop="appHome">
        <gm-input v-model="editForm.appHome" show-word-limit placeholder="请输入应用首页地址" maxlength="100" />
      </gm-form-item>
      <gm-form-item label="鉴权模式" prop="authMode">
        <template #label>
          <div class="flex items-center gap-1">
            <span>鉴权模式</span>
            <el-tooltip class="box-item mt-[2px]" effect="dark" content="" placement="top">
              <template #content>
                <div class="w-[250px]">
                  为了保障系统安全，目前所有应用和服务默认采用“严格模式”。即使您选择了“宽松模式”，系统也会按照严格标准执行。建议您暂时保持“严格模式”，后续我们将提供更灵活的权限设置。
                </div>
              </template>
              <el-icon size="18"><i-ep-QuestionFilled /></el-icon>
            </el-tooltip>
          </div>
        </template>
        <gm-radio-group v-model="editForm.authMode">
          <gm-radio v-for="it in authModeOptions" :key="it.label" :value="it.value">{{ it.label }}</gm-radio>
        </gm-radio-group>
      </gm-form-item>
    </gm-form>
  </div>
</template>
<script setup lang="ts">
import { useAppOptions } from '../composables/useAppOptions';
import type { FormInstance } from 'giime';
import type { AddIsvAppReq } from '@/api/open/interface';

const editForm = defineModel<AddIsvAppReq>('editForm', { required: true });
const editFormRef = ref<FormInstance>();

const { authModeOptions, rules } = useAppOptions();

const resetFields = () => {
  return editFormRef.value?.resetFields();
};
const validate = () => {
  return editFormRef.value?.validate();
};

defineExpose({
  resetFields,
  validate,
});
</script>
```

composables/types.ts

```ts
/**表单弹窗类型 */
export enum AppEditDialogType {
  /**添加 */
  ADD = 'add',
  /**编辑 */
  EDIT = 'edit',
}
```

composables/useAppOptions.ts

注意：tableId你应该自己生成一个，写死一个就行，不要使用uuid这个库动态生成，也不要使用我提供的这个。保证能uuid级别的唯一即可。

```ts
import { AppEditDialogType } from './types';

export const useAppOptions = () => {
  /**
   * 表格id
   */
  const tableId = '8145c521-054d-4c5c-bbdb-296528ad860c';

  /**表单弹窗标题选项 */
  const dialogTitleOptions = [
    { label: '新增会话标签', value: AppEditDialogType.ADD },
    { label: '编辑会话标签', value: AppEditDialogType.EDIT },
  ];
  /**
   * tageType 用于在列表中显示不同的颜色的tag
   */
  const authModeOptions = [
    { label: '严格模式', value: 1, tageType: 'danger' as const },
    { label: '宽松模式', value: 2, tageType: 'success' as const },
  ];
  // 状态：0-禁用，1-正常
  const statusOptions = [
    { label: '禁用', value: 0 },
    { label: '正常', value: 1 },
  ];

  const rules = {
    appName: [{ required: true, message: '应用名称不能为空', trigger: 'blur' }],
    authMode: [{ required: true, message: '鉴权模式不能为空', trigger: 'blur' }],
    clientId: [{ required: true, message: '应用的client_id不能为空', trigger: 'blur' }],
    appId: [{ required: true, message: '应用ID不能为空', trigger: 'blur' }],
    clientSecret: [{ required: true, message: '应用的client_secret不能为空', trigger: 'blur' }],
    createTime: [{ required: true, message: '录入时间不能为空', trigger: 'blur' }],
    status: [{ required: true, message: '应用状态不能为空', trigger: 'blur' }],
    serviceCount: [{ required: true, message: '开通服务数量不能为空', trigger: 'blur' }],
    updateTime: [{ required: true, message: '更新时间不能为空', trigger: 'blur' }],
  };

  return {
    tableId,
    rules,
    authModeOptions,
    statusOptions,
    dialogTitleOptions,
  };
};
```
