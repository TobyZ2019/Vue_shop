<template>
    <div>
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/">商品管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- tree-table 区域 -->
            <tree-table
            :data="cateList"
            :columns="columns"
            :selection-type="false"
            :expand-type="false"
            show-index
            index-text="#"
            border
            :show-row-hover="false"
            style="margin-top: 15px">
                <template slot="isok" slot-scope="scope">
                    <i class="el-icon-success"
                    v-if="scope.row.cat_deleted === false"
                    style="color: green;"></i>
                    <i class="el-icon-error" v-else style="color: red;"></i>
                </template>
                <template slot="order" slot-scope="scope">
                    <el-tag type="primary" size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>
                <template slot="opt" slot-scope="scope">
                    <el-button icon="el-icon-search" type="primary" @click="showEditCateDialog(scope.row)">编辑</el-button>
                    <el-button icon="el-icon-search" type="danger" @click="removeCateById(scope.row.cat_id)">删除</el-button>
                </template>
            </tree-table>
            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryCateParams.pagenum"
                :page-sizes="[5, 10, 20, 100]"
                :page-size="queryCateParams.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
    <!-- 弹出式dialog区域 -->
        <!-- 添加dialog -->
        <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClose">
            <el-form
            :model="addCateForm"
            :rules="addCateFormRules"
            ref="addCateFormRef"
            label-width="100px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类">
                    <!-- option指定数据源 -->
                    <el-cascader
                    v-model="selectedKeys"
                    :options="parentCateList"
                    expand-trigger="hover"
                    :props="Selectprops"
                    @change="parentCateChange"
                    clearable
                    change-on-select></el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 编辑dialog -->
        <el-dialog
        title="修改分类"
        :visible.sync="editCateDialogVisible"
        width="50%"
        @close="editCateDialogClose">
            <el-form
            :model="editCateInfo"
            :rules="addCateFormRules"
            ref="editCateFormRef"
            label-width="100px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="editCateInfo.cat_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            cateList: [],
            queryCateParams: {
                type: [],
                pagenum: 1,
                pagesize: 5
            },
            total: 0,
            // tree-table的列
            columns: [
                {
                    label: '分类名称',
                    prop: 'cat_name'
                },
                {
                    label: '是否有效',
                    // 自定义模板列
                    type: 'template',
                    template: 'isok'
                },
                {
                    label: '排序',
                    type: 'template',
                    template: 'order'
                },
                {
                    label: '操作',
                    type: 'template',
                    template: 'opt'
                }
            ],
            addCateDialogVisible: false,
            editCateDialogVisible: false,
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            addCateFormRules: {
                cat_name: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' }
                ],
                cat_pid: [
                    { required: true, message: '请选择分类名称', trigger: 'blur' }
                ],
                cat_level: [
                    { required: true, message: '请选择分类层级', trigger: 'blur' }
                ]
            },
            parentCateList: [],
            // 级联选择器配置
            selectedKeys: [],
            Selectprops: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            editCateInfo: {}
        }
    },
    methods: {
        async getCateList() {
            const { data: res } = await this.$http.get('categories',
                { params: this.queryCateParams })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类失败')
            }
            this.cateList = res.data.result
            this.total = res.data.total
        },
        // pagesize变化事件
        handleSizeChange(newSize) {
            this.queryCateParams.pagesize = newSize
            this.getCateList()
        },
        // pagenum变化事件
        handleCurrentChange(newPageNum) {
            this.queryCateParams.pagenum = newPageNum
            this.getCateList()
        },
        showAddCateDialog() {
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        async getParentCateList() {
            const { data: res } = await this.$http.get('categories',
                { params: { type: 2 } })
            if (res.meta.status !== 200) {
                return this.$message.error('获取父级分类失败')
            }
            this.parentCateList = res.data
        },
        parentCateChange() {
            if (this.selectedKeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                this.addCateForm.cat_level = this.selectedKeys.length
            } else {
                this.addCateForm.cat_level = 0
            }
        },
        addCate() {
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.post('categories',
                    this.addCateForm)
                if (res.meta.status !== 201) {
                    return this.$message.error('添加商品分类失败')
                }
                this.$message.success('添加商品分类成功')
                this.getCateList()
                this.addCateDialogVisible = false
            })
        },
        addCateDialogClose() {
            this.$refs.addCateFormRef.resetFields()
            this.selectedKeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0
        },
        showEditCateDialog(cateInfo) {
            this.editCateInfo = cateInfo
            this.editCateDialogVisible = true
        },
        editCateDialogClose() {
            this.$refs.editCateFormRef.resetFields()
            this.editCateInfo = {}
        },
        editCate() {
            this.$refs.editCateFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.put(`categories/${this.editCateInfo.cat_id}`,
                    { cat_name: this.editCateInfo.cat_name })
                if (res.meta.status !== 200) {
                    return this.$message.error('修改商品分类失败')
                }
                this.$message.success('修改商品分类成功')
                this.getCateList()
                this.editCateDialogVisible = false
            })
        },
        async removeCateById(id) {
            const confirmResult = await this.$confirm('此操作会永久删除该分类，是否继续？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('取消了删除操作')
            }
            const { data: res } = await this.$http.delete(`categories/${id}`)
            if (res.meta.status !== 200) {
                return this.$message.error('删除失败')
            }
            this.$message.success('删除成功')
            this.getCateList()
        }
    },
    created() {
        this.getCateList()
    }
}
</script>

<style lang="less" scoped>

</style>
