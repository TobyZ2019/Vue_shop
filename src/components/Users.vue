/* eslint-disable no-unused-expressions */
<template>
    <div>
        <!-- 面包屑区域 -->
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card class="box-card">
            <div style="margin-top: 15px;">
                <!-- 搜索与添加区 -->
                <el-row :gutter="10">
                    <el-col :span="8">
                        <div class="grid-content bg-purple">
                            <el-input clearable @clear="getUserList" @keyup.enter.native="getUserList()" placeholder="请输入内容" v-model="queryInfo.query">
                                <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                            </el-input>
                        </div>
                    </el-col>
                    <el-col :span="2">
                        <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
                    </el-col>
                </el-row>
                <!-- 用户列表区 -->
                <el-table :data="userList" style="width: 100%" border stripe>
                    <el-table-column type="index" label="#"></el-table-column>
                    <el-table-column prop="role_name" label="角色"></el-table-column>
                    <el-table-column prop="username" label="姓名"></el-table-column>
                    <el-table-column prop="create_time" label="创建日期"></el-table-column>
                    <el-table-column prop="mobile" label="手机号码"></el-table-column>
                    <el-table-column prop="email" label="邮箱地址"></el-table-column>
                    <el-table-column label="状态">
                        <template slot-scope="scope">
                            <el-switch @change="userStateChange(scope.row)"
                                v-model="scope.row.mg_state"
                                active-color="#13ce66"
                                inactive-color="#ff4949">
                            </el-switch>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editUserDialog(scope.row.id)"></el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                                <el-button type="warning" icon="el-icon-setting" size="mini"
                                @click="setRole(scope.row)"></el-button>
                            </el-tooltip>
                            {{scope.row.id}}
                        </template>
                    </el-table-column>
                </el-table>
                <!-- 分页区域 -->
                <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
                </el-pagination>
                <!-- 添加用户对话框 -->
                <el-dialog
                title="添加用户"
                :visible.sync="addDialogVisible"
                width="50%" @close="addDialogClose">
                    <el-form :model="addForm" :rules="addRules" ref="addRuleFormRef" label-width="70px">
                        <el-form-item label="用户名" prop="username">
                            <el-input v-model="addForm.username"></el-input>
                        </el-form-item>
                        <el-form-item label="密码" prop="password">
                            <el-input v-model="addForm.password"></el-input>
                        </el-form-item>
                        <el-form-item label="邮箱" prop="email">
                            <el-input v-model="addForm.email"></el-input>
                        </el-form-item>
                        <el-form-item label="手机" prop="mobile">
                            <el-input v-model="addForm.mobile"></el-input>
                        </el-form-item>
                    </el-form>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="addDialogVisible = false">取 消</el-button>
                        <el-button type="primary" @click="addUser">确 定</el-button>
                    </span>
                </el-dialog>
                <!-- 修改用户对话框 -->
                <el-dialog
                title="修改用户"
                :visible.sync="editDialogVisible"
                width="50%" @close="editDialogClose">
                    <el-form :model="editForm" :rules="addRules" ref="editRuleFormRef" label-width="70px">
                        <el-form-item label="用户名">
                            <el-input v-model="editForm.username" disabled></el-input>
                        </el-form-item>
                        <el-form-item label="邮箱" prop="email">
                            <el-input v-model="editForm.email"></el-input>
                        </el-form-item>
                        <el-form-item label="手机" prop="mobile">
                            <el-input v-model="editForm.mobile"></el-input>
                        </el-form-item>
                    </el-form>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="editDialogVisible = false">取 消</el-button>
                        <el-button type="primary" @click="editUser">确 定</el-button>
                    </span>
                </el-dialog>
                <!-- 分配角色对话框 -->
                <el-dialog
                title="分配角色"
                :visible.sync="setRoleDialogVisible"
                width="50%"
                @close="setRoleDialogClose">
                <div>
                    <p>当前用户：{{userInfo.username}}</p>
                    <p>当前角色：{{userInfo.role_name}}</p>
                    <p>分配新角色：
                        <el-select v-model="selectedRoleId" placeholder="请选择">
                            <el-option
                            v-for="item in rolesList"
                            :key="item.id"
                            :label="item.roleName"
                            :value="item.id">
                            </el-option>
                        </el-select>
                    </p>

                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
                </span>
                </el-dialog>
            </div>
        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 验证邮箱规则
            checkEmail: (rule, value, cb) => {
                const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
                if (regEmail.test(value)) {
                    return cb()
                }
                cb(new Error('请输入合法的邮箱'))
            },
            // 验证手机规则
            checkMobile: (rule, value, cb) => {
                const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
                if (regMobile.test(value)) {
                    return cb()
                }
                cb(new Error('请输入合法的手机号'))
            },
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 5
            },
            userList: [],
            total: 0,
            addDialogVisible: false,
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            addRules: {
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' },
                    { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: this.checkEmail, trigger: 'blur' }],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: this.checkMobile, trigger: 'blur' }]
            },
            editDialogVisible: false,
            editForm: {},
            setRoleDialogVisible: false,
            // 分配角色的用户信息
            userInfo: {},
            // 所有角色的数据列表
            rolesList: [],
            // 已选中的角色id值
            selectedRoleId: ''
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList() {
            const { data: res } = await this.$http.get('users', { params: this.queryInfo })
            if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
            this.userList = res.data.users
            this.total = res.data.total
        },
        search: function(ev) {
            if (ev.keycode === '13') {
                this.getUserList()
            }
        },
        // 监听pagesize改变的事件
        handleSizeChange(newpage) {
            this.queryInfo.pagesize = newpage
            this.getUserList()
        },
        // 页码值切换事件
        handleCurrentChange(newpage) {
            this.queryInfo.pagenum = newpage
            this.getUserList()
        },
        // 监听用户状态改变
        async userStateChange(userinfo) {
            const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
            if (res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state
                return this.$message.error('用户更新失败')
            }
            this.$message.success('用户更新成功')
        },
        // 监听对话框关闭事件
        addDialogClose() {
            this.$refs.addRuleFormRef.resetFields()
        },
        // 点击确定按钮，添加新用户
        addUser() {
            this.$refs.addRuleFormRef.validate(async vaild => {
                if (!vaild) return
                const { data: res } = await this.$http.post('/users', this.addForm)
                if (res.meta.status !== 201) {
                    this.$message.error('添加用户失败')
                }
                this.$message.success('添加用户成功')
                this.addDialogVisible = false
                this.getUserList()
            })
        },
        // 展示编辑用户对话框
        async editUserDialog(id) {
            const { data: res } = await this.$http.get('/users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('查询用户信息失败')
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClose() {
            this.$refs.editRuleFormRef.resetFields()
        },
        // 修改用户并提交
        editUser() {
            this.$refs.editRuleFormRef.validate(async vaild => {
                if (!vaild) return
                const { data: res } = await this.$http.put('/users/' + this.editForm.id,
                    {
                        email: this.editForm.email,
                        mobile: this.editForm.mobile
                    })
                if (res.meta.status !== 200) {
                    return this.$message.error('修改用户失败')
                }
                this.editDialogVisible = false
                this.$message.success('修改用户成功')
                console.log(res)
                this.getUserList()
            })
        },
        async removeUserById(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => { return err })
            if (confirmResult !== 'confirm') {
                return this.$message.info('已经取消删除')
            }
            const { data: res } = await this.$http.delete('/users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除用户失败')
            }
            this.$message.success('删除用户失败')
            this.getUserList()
        },
        // 展示分配角色对话框，并获取修改用户的信息
        async setRole(userinfo) {
            this.userInfo = userinfo
            // 获取所有的角色列表
            const { data: res } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('分配角色失败')
            }
            this.rolesList = res.data
            this.$message.success('分配角色成功')
            this.setRoleDialogVisible = true
        },
        // 将更改用户角色的信息提交到服务器
        async saveRoleInfo() {
            if (!this.selectedRoleId) {
                return this.$message.error('请选择角色')
            }
            const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`,
                { rid: this.selectedRoleId })
            if (res.meta.status !== 200) {
                return this.$message.error('保存角色失败')
            }
            this.setRoleDialogVisible = false
            this.getUserList()
            this.$message.success('分配角色成功')
        },
        setRoleDialogClose() {
            this.selectedRoleId = ''
            this.rolesList = []
            this.userInfo = {}
        }
    }
}
</script>

<style lang="less" scoped>

</style>
