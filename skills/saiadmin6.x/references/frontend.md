# Frontend Development Rules

## Directory Structure (`saiadmin-artd/src/views/plugin/<plugin_name>`)

| Directory/File | Description |
| :--- | :--- |
| `api/` | API definition files (TypeScript). |
| `<module>/` | Feature-specific views (e.g., `member`, `cms`). |
| `<module>/<page>/index.vue` | List page main entry. |
| `<module>/<page>/modules/` | Sub-components (`edit-dialog.vue`, `table-search.vue`). |

## 1. API Definition (`api/...`)

```typescript
import request from '@/utils/http'

/**
 * 模块名称 API接口
 */
export default {
  /**
   * 获取数据列表
   * @param params 搜索参数
   * @returns 数据列表
   */
  list(params: Record<string, any>) {
    return request.get<Api.Common.ApiPage>({
      // 重要: 插件接口统一以前缀 `/app/<plugin_name>/admin/...` 开头
      url: '/app/<plugin_name>/admin/module/Controller/index',
      params
    })
  },

  /**
   * 读取数据
   * @param id 数据ID
   * @returns 数据详情
   */
  read(id: number | string) {
    return request.get<Api.Common.ApiData>({
      url: '/app/<plugin_name>/admin/module/Controller/read?id=' + id
    })
  },

  /**
   * 创建数据
   * @param params 数据参数
   * @returns 执行结果
   */
  save(params: Record<string, any>) {
    return request.post<any>({
      url: '/app/<plugin_name>/admin/module/Controller/save',
      data: params
    })
  },

  /**
   * 更新数据
   * @param params 数据参数
   * @returns 执行结果
   */
  update(params: Record<string, any>) {
    return request.put<any>({
      url: '/app/<plugin_name>/admin/module/Controller/update',
      data: params
    })
  },

  /**
   * 删除数据
   * @param params 包含ids的参数
   * @returns 执行结果
   */
  delete(params: Record<string, any>) {
    return request.del<any>({
      url: '/app/<plugin_name>/admin/module/Controller/destroy',
      data: params
    })
  }

  // Tree models 额外添加:
  // listTree(params: Record<string, any>) {
  //   return request.get<any>({
  //     url: '/app/<plugin_name>/admin/module/Controller/index?tree=true',
  //     params
  //   })
  // }
}
```

## 1.1 Common Components (`src/components/sai/`)

在 `edit-dialog.vue` 中可使用以下预置组件：

| 组件 | 用途 | 示例 |
|:---|:---|:---|
| `sa-select` | 字典/远程下拉选择 | `<sa-select v-model="formData.type" dict="dict_type" />` |
| `sa-radio` | 字典单选 | `<sa-radio v-model="formData.status" dict="status" />` |
| `sa-checkbox` | 字典多选 | `<sa-checkbox v-model="formData.tags" dict="tags" />` |
| `sa-switch` | 状态开关 | `<sa-switch v-model="formData.status" />` |
| `sa-dict` | 字典标签展示 | `<sa-dict :value="row.type" dict="dict_type" />` |
| `sa-editor` | 富文本编辑器 | `<sa-editor v-model="formData.content" height="400px" />` |
| `sa-md-editor` | Markdown 编辑器 | `<sa-md-editor v-model="formData.content" />` |
| `sa-image-upload` | 单图上传 | `<sa-image-upload v-model="formData.avatar" />` |
| `sa-file-upload` | 文件上传 | `<sa-file-upload v-model="formData.file" />` |
| `sa-chunk-upload` | 大文件分片上传 | `<sa-chunk-upload v-model="formData.video" />` |
| `sa-icon-picker` | 图标选择器 | `<sa-icon-picker v-model="formData.icon" />` |
| `sa-image-picker` | 图库选择 | `<sa-image-picker v-model="formData.images" />` |
| `sa-code` | 代码展示 | `<sa-code :code="formData.code" lang="json" />` |
| `sa-user` | 用户选择器 | `<sa-user v-model="formData.user_id" />` |
| `sa-button` | 操作按钮 | `<sa-button type="secondary" @click="handleEdit" />` |
| `sa-label` | 标签展示 | `<sa-label :value="row.status" />` |
| `sa-search-bar` | 搜索栏容器 | 用于 `table-search.vue` |
| `sa-export` | 数据导出 | `<sa-export :api="api.export" />` |
| `sa-import` | 数据导入 | `<sa-import :api="api.import" />` |

> **Note**: 所有组件均已全局注册，可直接使用无需 import。

## 2. Table Search (`modules/table-search.vue`)
*Strictly follow `table-search.stub` pattern.*

> **规则**: 搜索字段超过 3 个时，需启用展开/收起功能，将第 4 个及之后的字段放入展开区域。

**关键配置:**
- `:showExpand="true"` - 显示展开/收起按钮（字段 > 3 时使用）
- `v-show="isExpanded"` - 控制额外字段的显示/隐藏

```vue
<template>
  <sa-search-bar
    ref="searchBarRef"
    v-model="formData"
    label-width="100px"
    :showExpand="true"
    @reset="handleReset"
    @search="handleSearch"
    @expand="handleExpand"
  >
    <!-- 前 3 个字段始终显示 -->
    <el-col v-bind="setSpan(6)">
      <el-form-item label="关键词" prop="keywords">
        <el-input v-model="formData.keywords" placeholder="请输入关键词" clearable />
      </el-form-item>
    </el-col>

    <el-col v-bind="setSpan(6)">
      <el-form-item label="类型" prop="type">
        <sa-select v-model="formData.type" dict="dict_type" placeholder="请选择类型" clearable />
      </el-form-item>
    </el-col>

    <el-col v-bind="setSpan(6)">
      <el-form-item label="状态" prop="status">
        <el-select v-model="formData.status" placeholder="请选择状态" clearable>
          <el-option label="启用" :value="1" />
          <el-option label="禁用" :value="0" />
        </el-select>
      </el-form-item>
    </el-col>

    <!-- 第 4 个及之后的字段：展开后显示 -->
    <el-col v-bind="setSpan(12)" v-show="isExpanded">
      <el-form-item label="创建时间" prop="create_time">
        <el-date-picker
          v-model="formData.create_time"
          type="datetimerange"
          range-separator="至"
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          clearable
        />
      </el-form-item>
    </el-col>
  </sa-search-bar>
</template>

<script setup lang="ts">
  interface Props {
    modelValue: Record<string, any>
  }
  interface Emits {
    (e: 'update:modelValue', value: Record<string, any>): void
    (e: 'search', params: Record<string, any>): void
    (e: 'reset'): void
  }
  const props = defineProps<Props>()
  const emit = defineEmits<Emits>()

  // 展开/收起
  const isExpanded = ref<boolean>(false)

  // 表单数据双向绑定
  const searchBarRef = ref()
  const formData = computed({
    get: () => props.modelValue,
    set: (val) => emit('update:modelValue', val)
  })

  // 重置
  function handleReset() {
    searchBarRef.value?.ref.resetFields()
    emit('reset')
  }

  // 搜索
  async function handleSearch() {
    emit('search', formData.value)
  }

  // 展开/收起
  function handleExpand(expanded: boolean) {
    isExpanded.value = expanded
  }

  // 栅格占据的列数 (响应式布局)
  const setSpan = (span: number) => {
    return {
      span: span,
      xs: 24,
      sm: span >= 12 ? span : 12,
      md: span >= 8 ? span : 8,
      lg: span,
      xl: span
    }
  }
</script>
```

---

## 3. Regular List Patterns

### Index Page (`index.vue`)
*Strictly follow `index.stub` (Regular Logic).*

```vue
<template>
  <div class="art-full-height">
    <!-- Search -->
    <TableSearch v-model="searchForm" @search="handleSearch" @reset="resetSearchParams" />

    <ElCard class="art-table-card" shadow="never">
      <!-- Table Header -->
      <ArtTableHeader v-model:columns="columnChecks" :loading="loading" @refresh="refreshData">
        <template #left>
          <ElSpace wrap>
            <ElButton v-permission="'plugin:module:save'" @click="showDialog('add')" v-ripple>
              <template #icon><ArtSvgIcon icon="ri:add-fill" /></template>
              新增
            </ElButton>
            <ElButton
              v-permission="'plugin:module:destroy'"
              :disabled="selectedRows.length === 0"
              @click="deleteSelectedRows(api.delete, refreshData)"
              v-ripple
            >
              <template #icon><ArtSvgIcon icon="ri:delete-bin-5-line" /></template>
              删除
            </ElButton>
          </ElSpace>
        </template>
      </ArtTableHeader>

      <!-- Table -->
      <ArtTable
        ref="tableRef"
        rowKey="id"
        :loading="loading"
        :data="data"
        :columns="columns"
        :pagination="pagination"
        @sort-change="handleSortChange"
        @selection-change="handleSelectionChange"
        @pagination:size-change="handleSizeChange"
        @pagination:current-change="handleCurrentChange"
      >
        <template #operation="{ row }">
          <div class="flex gap-2">
            <SaButton v-permission="'plugin:module:update'" type="secondary" @click="showDialog('edit', row)" />
            <SaButton v-permission="'plugin:module:destroy'" type="error" @click="deleteRow(row, api.delete, refreshData)" />
          </div>
        </template>
      </ArtTable>
    </ElCard>

    <EditDialog v-model="dialogVisible" :dialog-type="dialogType" :data="dialogData" @success="refreshData" />
  </div>
</template>

<script setup lang="ts">
  import { useTable } from '@/hooks/core/useTable'
  import { useSaiAdmin } from '@/composables/useSaiAdmin'
  import api from '../../api/module/controller'
  import TableSearch from './modules/table-search.vue'
  import EditDialog from './modules/edit-dialog.vue'

  const searchForm = ref({
    keywords: undefined,
  })

  const handleSearch = (params: Record<string, any>) => {
    Object.assign(searchParams, params)
    getData()
  }

  const {
    columns, columnChecks, data, loading, getData, searchParams, pagination,
    resetSearchParams, handleSortChange, handleSizeChange, handleCurrentChange, refreshData
  } = useTable({
    core: {
      apiFn: api.list,
      apiParams: { ...searchForm.value },
      columnsFactory: () => [
        { type: 'selection' },
        { prop: 'id', label: 'ID' },
        { prop: 'name', label: 'Name' },
        { prop: 'operation', label: '操作', width: 100, fixed: 'right', useSlot: true }
      ]
    }
  })

  const {
    dialogType, dialogVisible, dialogData, showDialog, deleteRow, deleteSelectedRows, handleSelectionChange, selectedRows
  } = useSaiAdmin()
</script>
```

### Edit Dialog (`modules/edit-dialog.vue`)
*Strictly follow `edit-dialog.stub`.*

```vue
<template>
  <el-drawer
    v-model="visible"
    :title="dialogType === 'add' ? '新增' : '编辑'"
    size="600px"
    @close="handleClose"
  >
    <el-form ref="formRef" :model="formData" :rules="rules" label-width="100px">
      <el-form-item label="Name" prop="name">
        <el-input v-model="formData.name" placeholder="Enter name" />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="handleClose">取消</el-button>
      <el-button type="primary" @click="handleSubmit">提交</el-button>
    </template>
  </el-drawer>
</template>

<script setup lang="ts">
  import api from '../../../api/module/controller'
  import { ElMessage } from 'element-plus'
  import type { FormInstance, FormRules } from 'element-plus'

  interface Props {
    modelValue: boolean
    dialogType: string
    data?: Record<string, any>
  }
  const props = withDefaults(defineProps<Props>(), {
    modelValue: false,
    dialogType: 'add',
    data: undefined
  })
  const emit = defineEmits(['update:modelValue', 'success'])

  const formRef = ref<FormInstance>()
  const visible = computed({
    get: () => props.modelValue,
    set: (val) => emit('update:modelValue', val)
  })

  // Initial Data
  const initialFormData = { name: '' }
  const formData = reactive({ ...initialFormData })
  
  const rules = reactive<FormRules>({
    name: [{ required: true, message: 'Required', trigger: 'blur' }]
  })

  watch(() => props.modelValue, (newVal) => {
    if (newVal) initPage()
  })

  const initPage = async () => {
    Object.assign(formData, initialFormData)
    if (props.data) {
      await nextTick()
      Object.assign(formData, props.data)
    }
  }

  const handleClose = () => {
    visible.value = false
    formRef.value?.resetFields()
  }

  const handleSubmit = async () => {
    if (!formRef.value) return
    try {
      await formRef.value.validate()
      if (props.dialogType === 'add') {
        await api.save(formData)
        ElMessage.success('Added')
      } else {
        await api.update(formData)
        ElMessage.success('Updated')
      }
      emit('success')
      handleClose()
    } catch (e) {
      console.error(e)
    }
  }
</script>
```

---

## 4. Tree List Patterns

### Index Page (`index.vue`)
*Add Expand/Collapse logic.*

```vue
<!-- In Header -->
<ElButton @click="toggleExpand" v-ripple>
  <template #icon>
    <ArtSvgIcon v-if="isExpanded" icon="ri:collapse-diagonal-line" />
    <ArtSvgIcon v-else icon="ri:expand-diagonal-line" />
  </template>
  {{ isExpanded ? '收起' : '展开' }}
</ElButton>

<script setup>
  // ... imports ...
  // State
  const isExpanded = ref(false)
  const tableRef = ref()

  // Logic
  const toggleExpand = () => {
    isExpanded.value = !isExpanded.value
    nextTick(() => {
      if (tableRef.value?.elTableRef && data.value) {
        const processRows = (rows: any[]) => {
          rows.forEach((row) => {
             if (row.children?.length) {
                tableRef.value.elTableRef.toggleRowExpansion(row, isExpanded.value)
                processRows(row.children)
             }
          })
        }
        processRows(data.value)
      }
    })
  }
</script>
```

### Edit Dialog (`modules/edit-dialog.vue`)
*Add Tree Data loading.*

```vue
<template>
   <!-- ... -->
   <el-form-item label="Parent" prop="parent_id">
      <el-tree-select 
         v-model="formData.parent_id" 
         :data="optionData.treeData" 
         check-strictly 
         clearable 
      />
   </el-form-item>
</template>

<script setup>
  // ...
  const optionData = reactive({ treeData: [] })

  const initPage = async () => {
    Object.assign(formData, initialFormData)
    
    // Load Tree Data
    const res = await api.list({ tree: true })
    optionData.treeData = [{ id: 0, value: 0, label: 'Root', children: res }]

    if (props.data) {
       await nextTick()
       Object.assign(formData, props.data)
    }
  }
</script>
```
