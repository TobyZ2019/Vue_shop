<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <el-button type="primary">添加角色</el-button>
            <el-table :data="rolesList" style="width: 100%" border stripe>
                <el-table-column label="" type="expand">
                    <template slot-scope="scope">
                        <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id"
                        :class="['bdbottom', i1 === 0 ? 'bdtop': '', 'vcenter']">
                            <!-- 一级权限 -->
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(item1.id, scope.row)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二三级权限 -->
                            <el-col :span="19">
                                <!-- 二级权限 -->
                                <el-row v-for="(item2, i2) in item1.children" :key="item2.id"
                                :class="[ i2 === 0 ?'': 'bdtop']">
                                    <el-col :span="6">
                                        <el-tag type="success"
                                        closable @close="removeRightById(item2.id, scope.row)">
                                        {{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag v-for="(item3) in item2.children" :key="item3.id"
                                        type="warning" closable @close="removeRightById(item3.id, scope.row)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column label="#" type="index">
                </el-table-column>
                <el-table-column label="角色名称" prop="roleName">
                </el-table-column>
                <el-table-column label="角色描述" prop="roleDesc">
                </el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini"
                        @click="showSetRightDialog(scope.row)">
                        分配权限
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <el-dialog title="分配权限" :visible.sync="setRightDialogVisible"
        width="50%" @close="setRightDialogClose">
            <el-tree :data="rightsList" :props="treeProps"
            show-checkbox node-key="id"
            default-expand-all
            :default-checked-keys="defKeys"
            ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editRight">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            rolesList: [],
            setRightDialogVisible: false,
            rightsList: [],
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            defKeys: [],
            roleId: ''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const { data: res } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色失败')
            }
            this.$message.success('获取角色成功')
            this.rolesList = res.data
        },
        async removeRightById(rightId, role) {
            const confirmResult = await this.$confirm('此操作会永久删除该用户的权限，是否继续？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('取消了删除操作')
            }
            const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rightId)
            if (res.meta.status !== 200) {
                return this.$message.error('删除失败')
            }
            role.children = res.data
            this.$message.success('删除成功')
        },
        async showSetRightDialog(role) {
            this.roleId = role.id
            const { data: res } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败')
            }
            this.rightsList = res.data
            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true
        },
        // 获取所有的三级权限
        getLeafKeys(node, arr) {
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => {
                this.getLeafKeys(item, arr)
            })
        },
        setRightDialogClose() {
            this.defKeys = []
        },
        async editRight() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const { data: res } =
            await this.$http.post('roles/' + this.roleId + '/rights', { rids: idStr })
            if (res.meta.status !== 200) {
                return this.$message.error('编辑权限列表失败')
            }
            this.$message.success('编辑权限列表成功')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
    .bdtop {
        border-top: 1px solid #eeeeee;
    }
    .bdbottom {
        border-bottom: 1px solid #eeeeee;
    }
    .el-tag {
        margin: 7px;
    }
    .vcenter {
        display: flex;
        align-items: center;
    }
</style>
