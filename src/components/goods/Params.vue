<template>
    <div>
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/">商品管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
        <!-- 警告区域 -->
            <el-alert title="注意：只允许为第三级分类设置相关参数！"
            type="warning" :closable="false" show-icon>
            </el-alert>
        <!-- 级联选择器区域 -->
            <el-row style="margin: 10px 0" >
                <el-col>
                    <span>选择商品分类：</span>
                    <el-cascader
                    v-model="selectedKeys"
                    :options="cateList"
                    :props="Selectprops"
                    @change="CateChange"
                    clearable></el-cascader>
                </el-col>
            </el-row>
        <!-- 选项卡区域 -->
            <el-tabs v-model="activeName" @tab-click="tabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isDisable" @click="showAddAttDialog">添加参数</el-button>
                    <el-table border stripe :data="manyTableData">
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable style="margin: 5px" @close="delAttVals(i, scope.row)">{{item}}</el-tag>
                                <!-- 添加new tag按钮组合 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInputRef"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    style="width: 120px">
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                                </el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditAttDialog(scope.row)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="delAtt(scope.row)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isDisable" @click="showAddAttDialog">添加属性</el-button>
                    <el-table border stripe :data="onlyTableData">
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable style="margin: 5px" @close="delAttVals(i, scope.row)">{{item}}</el-tag>
                                <!-- 添加new tag按钮组合 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInputRef"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    style="width: 120px">
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                                </el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditAttDialog(scope.row)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="delAtt(scope.row)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        <!-- 分页区域 -->
            <!-- <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryCateParams.pagenum"
                :page-sizes="[5, 10, 20, 100]"
                :page-size="queryCateParams.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination> -->
        <!-- dialog弹出框区域 -->
            <!-- 添加弹出框 -->
            <el-dialog
            :title="'添加'+titleText"
            :visible.sync="addAttDialogVisible"
            width="50%"
            @close="addAttDialogClose">
                <el-form
                :model="addAttInfo"
                :rules="addAttFormRules"
                ref="addAttFormRef"
                label-width="100px">
                    <el-form-item :label="titleText" prop="attr_name">
                        <el-input v-model="addAttInfo.attr_name"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addAttDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addAtt">确 定</el-button>
                </span>
            </el-dialog>
            <!-- 修改弹出框 -->
                <el-dialog
                :title="'修改'+titleText"
                :visible.sync="editAttDialogVisible"
                width="50%"
                @close="editAttDialogClose">
                    <el-form
                    :model="editAttInfo"
                    :rules="addAttFormRules"
                    ref="editAttFormRef"
                    label-width="100px">
                        <el-form-item :label="titleText" prop="attr_name">
                            <el-input v-model="editAttInfo.attr_name"></el-input>
                        </el-form-item>
                    </el-form>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="editAttDialogVisible = false">取 消</el-button>
                        <el-button type="primary" @click="editAtt">确 定</el-button>
                    </span>
                </el-dialog>
        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        return {
            cateList: [],
            selectedKeys: [],
            Selectprops: {
                expandTrigger: 'hover',
                checkStrictly: true,
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            activeName: 'many',
            manyTableData: [],
            onlyTableData: [],
            addAttDialogVisible: false,
            addAttInfo: {
                attr_name: ''
            },
            addAttFormRules: {
                attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
            },
            editAttDialogVisible: false,
            editAttInfo: {},
            inputVisible: false,
            inputValue: ''
        }
    },
    methods: {
        async getCateList() {
            const { data: res } = await this.$http.get('categories')
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类失败')
            }
            this.cateList = res.data
        },
        async CateChange() {
            if (this.selectedKeys.length !== 3) {
                this.selectedKeys = []
                this.manyTableData = []
                this.onlyTableData = []
                return
            }
            const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数列表失败')
            }
            res.data.forEach(item => {
                if (item.attr_vals) {
                    item.attr_vals = item.attr_vals.split(',')
                } else {
                    item.attr_vals = []
                }
                // 控制添加动态参数值得tag
                item.inputVisible = false
                item.inputValue = ''
            })
            if (this.activeName === 'many') {
                this.manyTableData = res.data
            }
            this.onlyTableData = res.data
        },
        tabClick(tab, event) {
            this.CateChange()
        },
        showAddAttDialog() {
            this.addAttDialogVisible = true
        },
        addAttDialogClose() {
            this.$refs.addAttFormRef.resetFields()
        },
        addAtt() {
            this.$refs.addAttFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name: this.addAttInfo.attr_name,
                    attr_sel: this.activeName
                })
                if (res.meta.status !== 201) {
                    return this.$message.error('获取参数列表失败')
                }
                this.addAttDialogVisible = false
                this.CateChange()
            })
        },
        showEditAttDialog(info) {
            this.editAttDialogVisible = true
            this.editAttInfo = info
        },
        editAttDialogClose() {
            this.$refs.editAttFormRef.resetFields()
        },
        editAtt() {
            this.$refs.editAttFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.put(`categories/${this.editAttInfo.cat_id}/attributes/${this.editAttInfo.attr_id}`, {
                    attr_name: this.editAttInfo.attr_name,
                    attr_sel: this.activeName
                })
                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数列表失败')
                }
                this.editAttDialogVisible = false
                this.CateChange()
            })
        },
        async delAtt(delAttInfo) {
            const confirmResult = await this.$confirm(`此操作会永久删除该${this.titleText}，是否继续？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('取消了删除操作')
            }
            const { data: res } = await this.$http.delete(`categories/${delAttInfo.attr_id}/attributes/${delAttInfo.attr_id}`)
            if (res.meta.status !== 200) {
                return this.$message.error('删除失败')
            }
            this.$message.success('删除成功')
            this.CateChange()
        },
        async handleInputConfirm(row) {
            if (row.inputValue.trim().length === 0) {
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            row.attr_vals.push(row.inputValue.trim())
            row.inputValue = ''
            row.inputVisible = false
            this.editAttVals(row)
        },
        async editAttVals(row) {
            const { data: res } = await this.$http.put(`categories/${row.cat_id}/attributes/${row.attr_id}`, { attr_name: row.attr_name, attr_sel: row.attr_sel, attr_vals: row.attr_vals.join(',') })
            if (res.meta.status !== 200) {
                return this.$message.error('修改参数项失败')
            }
            this.$message.success('修改参数项成功')
        },
        showInput(row) {
            row.inputVisible = true
            // $nextTick方法是当页面上的元素被重新渲染后，才会执行指定的回调函数
            // 此时inputVisible虽然为ture，但是还没有被渲染出来，所以需要用$nextTick
            this.$nextTick(_ => {
                this.$refs.saveTagInputRef.$refs.input.focus()
            })
        },
        delAttVals(index, row) {
            row.attr_vals.splice(index, 1)
            this.editAttVals(row)
        }
    },
    created() {
        this.getCateList()
    },
    computed: {
        isDisable() {
            if (this.selectedKeys.length !== 3) {
                return true
            }
            return false
        },
        cateId() {
            if (this.selectedKeys.length === 3) {
                return this.selectedKeys[2]
            }
            return null
        },
        titleText() {
            if (this.activeName === 'many') {
                return '动态参数'
            }
            return '静态属性'
        }
    }
}
</script>

<style lang="less" scoped>

</style>
