<template>
    <div>
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/goods">商品管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <!-- 搜索区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryGoodInfo.query" clearable @clear="getGoodsList">
                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addGoodInfo">添加商品</el-button>
                </el-col>
            </el-row>
            <!-- 表格区域 -->
            <el-table border stripe :data="goodInfo">
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="商品名称" prop="goods_name">
                </el-table-column>
                <el-table-column label="商品价格（元）" prop="goods_price" width="95px">
                </el-table-column>
                <el-table-column label="商品重量" prop="goods_weight" width="70px">
                </el-table-column>
                <el-table-column label="创建时间" prop="add_time" width="140px">
                    <template slot-scope="scope">
                        {{scope.row.add_time | dateFormat}}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" @click="showEditGoodInfoDialog(scope.row)" size="mini"></el-button>
                        <el-button type="danger" icon="el-icon-delete" @click="delGoodInfo(scope.row)" size="mini"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryGoodInfo.pagenum"
                :page-sizes="[5, 10, 20, 100]"
                :page-size="queryGoodInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
                background>
            </el-pagination>
            <!-- 弹出dialog区域 -->
            <!-- 编辑dialog -->
            <!-- <el-dialog
            title="修改商品"
            :visible.sync="editGoodDialogVisible"
            width="50%"
            @close="editGoodDialogClose">
                <el-form
                :model="editGoodInfo"
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
            </el-dialog> -->
        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        return {
            queryGoodInfo: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            goodInfo: [],
            total: 0,
            editGoodDialogVisible: false
        }
    },
    methods: {
        async getGoodsList() {
            const { data: res } = await this.$http.get('goods', { params: this.queryGoodInfo })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类失败')
            }
            this.goodInfo = res.data.goods
            this.total = res.data.total
        },
        editGoodInfo(data) {
        },
        addGoodInfo() {
            this.$router.push('/goods/add')
        },
        async delGoodInfo(data) {
            const confirmRes = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (!confirmRes) {
                return this.$message.info('已取消删除')
            }
            const { data: res } = await this.$http.delete('goods/' + data.goods_id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除商品失败')
            }
            this.$message.success('删除商品成功')
            this.getGoodsList()
        },
        handleSizeChange(newSize) {
            this.queryGoodInfo.pagesize = newSize
            this.getGoodsList()
        },
        handleCurrentChange(newPageNum) {
            this.queryGoodInfo.pagenum = newPageNum
            this.getGoodsList()
        },
        showEditGoodInfoDialog(data) {
            this.editGoodDialogVisible = true
        },
        editGoodDialogClose() {
            this.editGoodDialogVisible = false
        }
    },
    created() {
        this.getGoodsList()
    }
}
</script>

<style lang="less" scoped>

</style>
