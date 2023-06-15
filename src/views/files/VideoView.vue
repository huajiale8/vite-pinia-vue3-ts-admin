<template>
    <el-card class="box-card">
        <el-row class="el-row1">
            <el-col :span="2">
                <AddButton @click="add"/>
            </el-col>
        </el-row>
        <el-table
            border
            :data="navList"
        >
            <el-table-column prop="id" label="id" align="center"/>
            <el-table-column prop="title" label="名称" align="center"/>
            <el-table-column prop="status" label="状态" align="center">
                <template #default="scope">
                    <el-tag v-if="scope.row.status === 1" class="ml-2" type="success">启用💕</el-tag>
                    <el-tag v-else class="ml-2" type="danger">禁用🖤</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="url" label="url" align="center"/>
            <el-table-column prop="tag" label="标签" align="center">
                <template #default="scope">
                    <el-tag v-if="scope.row.tag === '导航菜单'" type="warning">导航菜单🐇🐇</el-tag>
                    <el-tag v-if="scope.row.tag === 'button'">页面按钮</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="created_at" label="开始日期" align="center">
                <template #default="scope">
                    <span>{{ infoData(scope.row.created_at) }}</span>
                </template>
            </el-table-column>
            <el-table-column prop="updated_at" label="截止日期" align="center">
                <template #default="scope">
                    <span>{{ infoData(scope.row.updated_at) }}</span>
                </template>
            </el-table-column>
            <el-table-column prop="updated_at" label="操作" align="center" width="90px">
                <template #default="scope">
                    <el-button @click="edit(scope.row.id)" type="primary" size="small" icon="Edit">编辑</el-button>
                </template>
            </el-table-column>
        </el-table>
        <ModpagingModule
            v-model:page-size="page.pageSize"
            v-model:page="page.page"
            :total="total"
            @render="getList"/>

        <el-dialog
            v-model="centerDialogVisible"
            :title="title"
            width="30%"
            destroy-on-close
            :before-close="close"
            draggable
            center>
            <el-form
                ref="ruleFormRef"
                :model="ruleForm"
                status-icon
                :rules="rules"
                label-width="120px">
                <el-form-item label="导航名称" prop="title">
                    <el-input v-model="ruleForm.title" />
                </el-form-item>
                <el-form-item label="导航地址" prop="url">
                    <el-input v-model="ruleForm.url"/>
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="centerDialogVisible = false">取消</el-button>
                    <el-button v-if="buttonFlag === true" type="primary" @click="submitForm(ruleFormRef)">确定
                    </el-button>
                    <el-button v-else type="primary" @click="updata">更新
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </el-card>
</template>
<script lang="ts" setup>
import {onMounted, reactive, ref} from "vue";
import {infoData} from '@/utils/infoData'
import ModpagingModule from "@/components/common/ModpagingModule.vue";
import AddButton from "@/components/common/AddButton.vue";
import {addNav_item, getNav_itemDetails, getNav_itemList, putNav_itemDetails} from "@/api/dataNavMenu";
import type {FormInstance, FormRules} from 'element-plus'
import {ElMessage} from 'element-plus'

const total = ref(0)
const navList = ref([])
const centerDialogVisible = ref(false)
const buttonFlag = ref(false)
const title = ref('')
const add = () => {
    centerDialogVisible.value = true
    buttonFlag.value = true
    title.value = '新增导航菜单'
}
const page = reactive({
    page: 1,
    pageSize: 10
})
onMounted(() => {
    getList()
})
const getList = () => {
    getNav_itemList(page).then(res => {
        navList.value = res.data.data.list
        total.value = res.data.data.total
    })
}
const ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    title: '',
    url: '',
    id: 0,
    tag: '导航菜单',
    group_id: 0,
    status: 0
})

const rules = reactive<FormRules>({
    title: [
        {required: true, message: '请输入菜单名称', trigger: 'blur'},
    ],
    url: [
        {required: true, message: '请输入菜单地址', trigger: 'blur'},
    ],
})

const submitForm = (formEl: FormInstance | undefined) => {
    if (!formEl) return
    formEl.validate((valid) => {
        if (valid) {
            addNav_item(ruleForm).then(res => {
                ElMessage({
                    type: 'success',
                    message: res.msg,
                })
                ruleForm.title = ''
                ruleForm.url = ''
                ruleForm.group_id = 0
                centerDialogVisible.value = false
                buttonFlag.value = false
                getList()
            })
        } else {
            return false
        }
    })
}
const edit = (id: number) => {
    title.value = '编辑导航菜单'
    getNav_itemDetails(id).then(res => {
        let s = res.data.data
        ruleForm.title = s.title
        ruleForm.url = s.url
        ruleForm.id = s.id
        ruleForm.group_id = s.group_id
        ruleForm.status = s.status
        centerDialogVisible.value = true
        buttonFlag.value = false
    })
}
const updata = () => {
    putNav_itemDetails(ruleForm.id,ruleForm).then(res=>{
        centerDialogVisible.value = false
        buttonFlag.value= false
        getList()
    })
}
const close = () => {
    centerDialogVisible.value = false
    buttonFlag.value = false
    ruleForm.title = ''
    ruleForm.url = ''
    ruleForm.tag = '导航菜单'
    ruleForm.id = 0
    ruleForm.group_id = 0
    ruleForm.status = 0
}
</script>

<style lang="scss" scoped>
.el-row1 {
    margin-bottom: 10px;
}
</style>
