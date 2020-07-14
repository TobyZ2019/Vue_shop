<template>
    <div>
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/goods">商品管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-alert
                title="添加商品信息"
                type="info"
                show-icon center :closable="false">
            </el-alert>
            <!-- 步骤区域 -->
            <el-steps :active="stepNum - 0" finish-status="success" align-center>
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>
            <!-- tab区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
                <el-tabs v-model="stepNum" tab-position="left" style="margin-top: 30px;" :before-leave="beforeTabLeave" @tab-click="getCateAttr">
                <el-tab-pane label="基本信息" name="0">
                    <el-form-item label="商品名称" prop="goods_name">
                        <el-input v-model="addForm.goods_name"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格" prop="goods_price">
                        <el-input v-model="addForm.goods_price"></el-input>
                    </el-form-item>
                    <el-form-item label="商品重量" prop="goods_weight">
                        <el-input v-model="addForm.goods_weight"></el-input>
                    </el-form-item>
                    <el-form-item label="商品数量" prop="goods_number">
                        <el-input v-model="addForm.goods_number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品分类" prop="goods_cat">
                        <el-cascader
                        v-model="addForm.goods_cat"
                        :options="cateList"
                        :props="cateProps"
                        @change="cascaderChange"
                        clearable></el-cascader>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品参数" name="1">
                    <el-form-item v-for="item in manyTableData" :label="item.attr_name" :key="item.attr_id">
                        <el-checkbox-group v-model="item.attr_vals">
                            <el-checkbox v-for="(cb, i) in item.attr_vals" :key="i" :label="cb" border></el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品属性" name="2">
                    <el-form-item v-for="item in onlyTableData" :label="item.attr_name" :key="item.attr_id">
                        <el-input v-model="item.attr_vals"></el-input>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品图片" name="3">
                    <el-upload
                    action="http://127.0.0.1:8888/api/private/v1/upload"
                    :on-preview="handlePreview"
                    :on-remove="handleRemove"
                    list-type="picture"
                    :headers="header"
                    :on-success="handleSuccess">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-tab-pane>
                <el-tab-pane label="商品内容" name="4">
                    <quill-editor v-model="addForm. goods_introduce"/>
                    <el-button type="primary" style="margin-top: 20px" @click="addGood">添加商品</el-button>
                </el-tab-pane>
            </el-tabs>
            </el-form>
        </el-card>
        <!-- 图片dialog区域 -->
        <el-dialog
        title="预览图片"
        :visible.sync="picDialogVisible"
        width="50%"
        :before-close="picDialogClose">
            <img :src="preView" alt="" class="preViewPic">
        </el-dialog>
    </div>
</template>

<script>
import _ from 'lodash'
export default {
    data() {
        return {
            cateList: [],
            cateProps: {
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            stepNum: '0',
            addForm: {
                goods_name: '',
                goods_cat: [],
                goods_price: 0,
                goods_number: 0,
                goods_weight: 0,
                goods_introduce: '',
                pics: [],
                attrs: []
            },
            addFormRules: {
                goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
                goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }],
                goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
                goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
                goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }]
            },
            manyTableData: [],
            onlyTableData: [],
            header: {
                Authorization: window.sessionStorage.getItem('token')
            },
            preView: '',
            picDialogVisible: false
        }
    },
    methods: {
        async getCateList() {
            const { data: res } = await this.$http.get('categories')
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品列表失败')
            }
            this.cateList = res.data
        },
        async getCateAttr() {
            if (this.stepNum === '1') {
                const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
                if (res.meta.status !== 200) {
                    return this.$message.error('获取参数失败')
                }
                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
                })
                this.manyTableData = res.data
            } else if (this.stepNum === '2') {
                const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
                if (res.meta.status !== 200) {
                    return this.$message.error('获取属性失败')
                }
                this.onlyTableData = res.data
            }
        },
        cascaderChange() {
            if (this.addForm.goods_cat.length !== 3) {
                this.addForm.goods_cat = []
            }
        },
        beforeTabLeave(activeName, oldActiveName) {
            if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
                this.$message.error('请选择商品分类')
                return false
            }
            return true
        },
        handlePreview(file) {
            this.preView = file.response.data.url
            this.picDialogVisible = true
        },
        handleRemove(file) {
            const i = this.addForm.pics.findIndex(x => {
                return x.pic === file.response.data.tmp_path
            })
            this.addForm.pics.splice(i, 1)
        },
        handleSuccess(response) {
            const picInfo = { pic: response.data.tmp_path }
            this.addForm.pics.push(picInfo)
        },
        picDialogClose() {
            this.preView = ''
            this.picDialogVisible = false
        },
        addGood() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) {
                    return this.$message.error('请填写必填选择项')
                }
                const form = _.cloneDeep(this.addForm)
                form.goods_cat = form.goods_cat.join(',')
                this.manyTableData.forEach(item => {
                    const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(',') }
                    this.addForm.attrs.push(newInfo)
                })
                this.onlyTableData.forEach(item => {
                    const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
                    this.addForm.attrs.push(newInfo)
                })
                form.attrs = this.addForm.attrs
                const { data: res } = await this.$http.post('goods', form)
                if (res.meta.status !== 201) {
                    return this.$message.error('获取属性失败')
                }
                this.$message.success('添加商品成功')
                this.$router.push('/goods')
            })
        }
    },
    created() {
        this.getCateList()
    },
    computed: {
        cateId() {
            if (this.addForm.goods_cat.length === 3) {
                return this.addForm.goods_cat[2]
            }
            return null
        }
    }
}
</script>

<style lang="less" scoped>
    .el-checkbox {
        margin: 0 5px 0 0 !important;
    }
    .preViewPic {
        width: 100%;
    }
</style>
