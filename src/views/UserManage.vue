<template>
  <div class="page">
    <el-card shadow="never" class="toolbar">
      <el-form :inline="true" :model="query">
        <el-form-item label="用户名">
          <el-input v-model="query.username" placeholder="模糊搜索" clearable />
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="query.status" placeholder="全部" clearable style="width:120px">
            <el-option label="启用" :value="1" />
            <el-option label="禁用" :value="0" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSearch">查询</el-button>
          <el-button @click="onReset">重置</el-button>
          <el-button type="success" @click="openAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <el-card shadow="never" class="table-card">
      <el-table :data="tableData" v-loading="loading" border stripe>
        <el-table-column prop="id" label="ID" width="80" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="phone" label="手机号" />
        <el-table-column prop="role" label="角色" />
        <el-table-column label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="row.status ? 'success' : 'info'">
              {{ row.status ? '启用' : '禁用' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160" fixed="right">
          <template #default="{ row }">
            <el-button size="small" @click="openEdit(row)">编辑</el-button>
            <el-button size="small" type="danger" @click="onDelete(row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="pager">
        <el-pagination
          background
          layout="total, sizes, prev, pager, next"
          :total="total"
          :page-size="query.pageSize"
          :current-page="query.page"
          @size-change="onSizeChange"
          @current-change="onPageChange"
        />
      </div>
    </el-card>

    <!-- 新增/编辑弹窗 -->
    <el-dialog v-model="dialogVisible" :title="form.id ? '编辑用户' : '新增用户'" width="480px">
      <el-form ref="formRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="form.phone" />
        </el-form-item>
        <el-form-item label="角色" prop="role">
          <el-select v-model="form.role">
            <el-option label="管理员" value="admin" />
            <el-option label="普通用户" value="user" />
          </el-select>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio :label="1">启用</el-radio>
            <el-radio :label="0">禁用</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible=false">取消</el-button>
        <el-button type="primary" :loading="submitting" @click="onSubmit">确认</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { ElMessage, ElMessageBox, type FormInstance } from 'element-plus'

// Mock 数据源（内存里改，刷新会复原，够 Demo 用）
const db = ref([
  { id: 1, username: '张三', phone: '13800000001', role: 'admin', status: 1 },
  { id: 2, username: '李四', phone: '13800000002', role: 'user', status: 0 },
  { id: 3, username: '王五', phone: '13800000003', role: 'user', status: 1 },
  { id: 4, username: '赵六', phone: '13800000004', role: 'user', status: 1 },
  { id: 5, username: '孙七', phone: '13800000005', role: 'user', status: 0 },
  { id: 6, username: '周八', phone: '13800000006', role: 'admin', status: 1 },
  { id: 7, username: '吴九', phone: '13800000007', role: 'user', status: 1 },
  { id: 8, username: '郑十', phone: '13800000008', role: 'user', status: 0 },
  { id: 9, username: '钱十一', phone: '13800000009', role: 'user', status: 1 },
  { id: 10, username: '冯十二', phone: '13800000010', role: 'user', status: 1 },
  { id: 11, username: '陈十三', phone: '13800000011', role: 'user', status: 1 },
  { id: 12, username: '楚十四', phone: '13800000012', role: 'admin', status: 0 },
])

const query = reactive({ username: '', status: '' as '' | 0 | 1, page: 1, pageSize: 5 })
const tableData = ref<any[]>([])
const total = ref(0)
const loading = ref(false)

const fetchData = () => {
  loading.value = true
  setTimeout(() => {
    const filtered = db.value.filter(r =>
      (query.username ? r.username.includes(query.username) : true) &&
      (query.status !== '' ? r.status === query.status : true)
    )
    total.value = filtered.length
    tableData.value = filtered.slice(
      (query.page - 1) * query.pageSize,
      query.page * query.pageSize
    )
    loading.value = false
  }, 200)
}
onMounted(fetchData)

const onSearch = () => { query.page = 1; fetchData() }
const onReset = () => { query.username = ''; query.status = ''; query.page = 1; fetchData() }
const onSizeChange = (size: number) => { query.pageSize = size; query.page = 1; fetchData() }
const onPageChange = (p: number) => { query.page = p; fetchData() }

const dialogVisible = ref(false)
const submitting = ref(false)
const formRef = ref<FormInstance>()
const form = reactive({ id: undefined as number | undefined, username: '', phone: '', role: 'user', status: 1 })
const rules = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  phone: [{ required: true, message: '请输入手机号', trigger: 'blur' }],
  role: [{ required: true, message: '请选择角色', trigger: 'change' }],
}

const openAdd = () => { form.id = undefined; form.username = ''; form.phone = ''; form.role = 'user'; form.status = 1; dialogVisible.value = true }
const openEdit = (row: any) => { Object.assign(form, row); dialogVisible.value = true }
const onSubmit = async () => {
  await formRef.value?.validate()
  submitting.value = true
  setTimeout(() => {
    if (form.id) {
      Object.assign(db.value.find(r => r.id === form.id)!, form)
      ElMessage.success('编辑成功')
    } else {
      db.value.unshift({ ...form, id: Date.now() } as any)
      ElMessage.success('新增成功')
    }
    submitting.value = false
    dialogVisible.value = false
    fetchData()
  }, 300)
}

const onDelete = (row: any) => {
  ElMessageBox.confirm(`删除「${row.username}」？`, '提示', { type: 'warning' }).then(() => {
    db.value = db.value.filter(r => r.id !== row.id)
    ElMessage.success('删除成功')
    fetchData()
  }).catch(() => {})
}
</script>

<style scoped>
.page { padding: 24px; background: #0d1117; min-height: 100vh; color: #e0e0e0; }
.toolbar, .table-card { background: #161b2e; border: 1px solid #30363d; margin-bottom: 16px; }
.pager { margin-top: 16px; display: flex; justify-content: flex-end; }
:deep(.el-card__body) { color: #e0e0e0; }
:deep(.el-table) { background: transparent; --el-table-bg-color: transparent; --el-table-tr-bg-color: #161b2e; --el-table-header-bg-color: #1a2040; }
:deep(.el-pagination) { color: #e0e0e0; }
</style>