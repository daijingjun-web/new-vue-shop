<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card 视图区 -->
    <el-card>

      <!-- Alert 警告区 -->
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
      </el-alert>

      <!-- 步骤条 -->
      <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- Tabs 标签页 -->
      <el-form :model="addForm" :rules="addRules" ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" tab-position="left" :before-leave="beforeTabLeave" @tab-click="tabClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price" type="number">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat" :options="catelist" :props="cateaProps" @change="handleChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的item项 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="it" v-for="(it,i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload action="http://127.0.0.1:8888/api/private/v1/upload" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :headers="headersObj" :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="addBtn" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>

    </el-card>

    <!-- 预览dialog对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewDialogVisible" width="50%">
      <img :src="previewPath" alt="" class="preview">
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 步骤条 激活项
      activeIndex: '0',
      // form 表单绑定的 数据对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属的 分类 数组
        goods_cat: [],
        // 上传的图片临时路径
        pics: [],
        // 介绍
        goods_introduce: '',
        // 商品的参数（数组），包含 `动态参数` 和 `静态属性`
        attrs:[]
      },
      goodscat:'',
      // 表单验证规则
      addRules: {
         goods_name: [
            { required: true, message: '请输入商品名称', trigger: 'blur' },
          ],
         goods_price: [
            { required: true, message: '请输入商品价格', trigger: 'blur' },
          ],
         goods_weight: [
            { required: true, message: '请输入商品重量', trigger: 'blur' },
          ],
         goods_number: [
            { required: true, message: '请输入商品数量', trigger: 'blur' },
          ],
         goods_cat: [
            { required: true, message: '请选择商品分类', trigger: 'blur' },
          ],
      },
      // 商品分类列表
      catelist: [],
      // 级联选择器 配置对象
      cateaProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
      },
      // 动态参数列表数据
      manyTableData:[],
      onlyTableData:[],
      // 设置上传图片的请求头部
      headersObj:{
        Authorization: window.sessionStorage.getItem('token')
      },
      // 预览图片地址
      previewPath: '',
      // 预览dialog对话框
      previewDialogVisible: false
    }
  },
  methods: {
    async getCateList() {
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status!==200) return this.$message.error('获取商品分类数据失败')
      this.catelist = res.data
      // console.log(res);
    },
    // 级联选择器选项改变 触发 只能选定3级
    handleChange() {
      // console.log('ok');
      // console.log(this.addForm.goods_cat);
      if(this.addForm.goods_cat.length!==3) {
        this.addForm.goods_cat = []
      }
    },
    // 限制 tab 切换
    beforeTabLeave(activeName, oldActiveName){
      // console.log(oldActiveName);
      if(oldActiveName==='0' && this.addForm.goods_cat.length!==3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // tab 切换到 商品参数 项
    async tabClicked(){
      // console.log(this.activeIndex);
      if(this.activeIndex==='1') {
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params: {sel:'many'}})
        if(res.meta.status!==200) return this.$message.error('获取参数列表失败')
        // console.log(res.data);
        res.data.forEach(item=>{
          item.attr_vals=item.attr_vals.length?item.attr_vals.split(' '):[]
        })
        this.manyTableData = res.data
      } else if(this.activeIndex==='2'){
        // console.log('2');
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params: {sel:'only'}})
        if(res.meta.status!==200) return this.$message.error('获取参数列表失败')
        // console.log(res.data);
        // res.data.forEach(item=>{
        //   item.attr_vals=item.attr_vals.length?item.attr_vals.split(' '):[]
        // })
        console.log(res.data);
        this.onlyTableData = res.data
      }
    },
    // 图片预览
    handlePreview(file) {
      // console.log(file);
      this.previewPath = file.response.data.url
      this.previewDialogVisible = true
    },
    // 移除图片
    handleRemove(file) {
      // console.log(file);
      const filePath = file.response.data.tmp_path
      // 找到对应地址在数组中的索引 findIdex
      const i = this.addForm.pics.findIndex(x=>{
        x.pic == filePath
      })
      this.addForm.pics.splice(i,1)

    },
    // 监听图片上传事件
    handleSuccess(reponse) {
      const picInfo = {pic:reponse.data.tmp_path}
      this.addForm.pics.push(picInfo)
    },
    // 添加商品事件
    addGoods() {
      // console.log('ok');
      this.$refs.addFormRef.validate( async valid => {
        if (!valid) return this.$message.error('请填写必要的商品参数')
        this.goodscat=this.addForm.goods_cat.join(',')
        // console.log(this.goodscat);
        this.manyTableData.forEach( item=>{
          const newInfo = {attr_id:item.attr_id,attr_value:item.attr_vals.join('')}
          this.addForm.attrs.push(newInfo)
        })
        // 
        this.onlyTableData.forEach( item=>{
          const newInfo = {attr_id:item.attr_id,attr_value:item.attr_vals}
          this.addForm.attrs.push(newInfo)
        })
        // console.log(this.addForm.attrs);
        this.addForm.goods_cat=this.goodscat
        // 添加商品
        const {data:res} = await this.$http.post('goods',this.addForm)
        if(res.meta.status!==201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        this.$router.push('goods')
      })
    }
  },
  // 计算属性
  computed:{
    cateId(){
      if(this.addForm.goods_cat.length==3){
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },
  created() {
    this.getCateList()
  },
}
</script>

<style lang="less" scoped>
.el-steps{
  margin: 15px 0;
}
.el-step__title {
  font-size: 13px;
}
.el-checkbox {
  margin: 0 8px 0 0 !important;
}
.preview {
  width: 100%;
}
.ql-editor {
  min-height: 300px!important;
}
.addBtn {
  margin-top: 15px;
}
</style>