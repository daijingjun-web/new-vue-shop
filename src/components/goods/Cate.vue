<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 按钮 -->
      <el-row>
        <el-button type="primary" @click="addCateDialog">添加分类</el-button>
      </el-row>
      <!-- 表格 -->
      <el-table :data="catelist" style="width: 100%;margin-bottom: 20px;margin-top: 20px" row-key="cat_id" :indent=indent border :tree-props="{children: 'children'}">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="cat_name" label="分类名称" sortable width="230">

        </el-table-column>
        <el-table-column label="是否有效" sortable width="140" v-slot="scope">
          <i class="el-icon-error err" v-if="scope.row.cat_deleted"></i>
          <i class="el-icon-success suc" v-else></i>
        </el-table-column>
        <el-table-column label="排序" sortable v-slot="scope">
          <el-tag v-if="scope.row.cat_level==0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level==1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </el-table-column>
        <el-table-column label="操作">
          <el-button type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete">删除</el-button>
        </el-table-column>
      </el-table>
      <!-- 页码 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[3,5,10,15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateRules" ref="addCateRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader ref="cascaderRef" clearable :options="parentCateList" v-model="selectedCateArr" :props="cascaderProps" @change="parentCateChanged"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类de请求参数
      queryInfo:{
        type:3,
        pagenum:1,
        pagesize:5
      },
      // 商品分类的数据列表
      catelist:[],
      // 总数据条数
      total:0,
      indent:40,
      // 添加分类的对话框 显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的对话框 中 表单验证规则
      addCateRules:{
        cat_name: [
            { required: true, message: '请输入分类名称', trigger: 'blur' },
            { min: 1, max: 15, message: '长度在 1 到 15 个字符', trigger: 'blur' }
        ]
      },
      // 添加分类的对话框 中 表单数据
      addCateForm:{
        cat_name:'',
        // 默认分类 id 0
        cat_pid: 0,
        // 默认分类等级 0 
        cat_level: 0
      },
      // 父级分类数据
      parentCateList:[],
      // 级联选择器的 配置对象
      cascaderProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        checkStrictly:true
      },
      // 级联选择器 选择对应选项的数组
      selectedCateArr:[]
    }
  },
  methods: {
    // 获取商品列表分类
    async getCateList() {
      const {data:res} = await this.$http.get('categories',{params:this.queryInfo})
      if(res.meta.status!==200) return this.$message.error('获取商品分类数据失败')
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 监听页码组件 事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击添加分类 
    async addCateDialog(){
      const {data:res} = await this.$http.get('categories',{params:{type:2}})
      if(res.meta.status!==200) return this.$message.error('获取父级分类数据失败')
      // console.log(res.data);
      this.parentCateList = res.data
      this.addCateDialogVisible = true
    },
    // 监听 级联选择器 变化 事件
    parentCateChanged() {
      // console.log(this.selectedCateArr);
      if(this.selectedCateArr.length>0) {
        this.addCateForm.cat_pid=this.selectedCateArr[this.selectedCateArr.length-1]
        this.addCateForm.cat_level=this.selectedCateArr.length
        return
      } else {
        this.addCateForm.cat_pid=0
        this.addCateForm.cat_level=0
      }
    },
    // 添加分类对话框点击 确定按钮 操作
    addCate() {
      // console.log(this.addCateForm);
      this.$refs.addCateRef.validate( async valid => {
        if(!valid) return
        const {data:res} = await this.$http.post('categories',this.addCateForm)
        if(res.meta.status!==201) return this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听 添加分类对话框关闭 重置表单 级联选择器
    addCateDialogClosed() {
      this.$refs.addCateRef.resetFields()
      // this.$refs.cascaderRef.clearCheckedNodes()
      this.selectedCateArr=[]
      this.addCateForm.cat_pid=0
      this.addCateForm.cat_level=0
    }
  },
  created() {
    this.getCateList()
  },
}
</script>

<style lang="less" scoped>
.suc {
  color: rgb(84, 238, 84);
}
.err {
  color: rgb(241, 59, 59);
}
.el-cascader {
  width: 100%;
}
</style>