<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card 卡片视图 -->
    <el-card>
      <!-- 提示框 -->
      <el-alert title="注意：只允许第三级分类设置相关参数！" type="warning" :closable="false" show-icon>
      </el-alert>
      <!-- 级联选择框 -->
      <span>选择商品分类：</span>
      <el-cascader v-model="paramsArr" :options="catelist" :props="cateProps" @change="handleChange"></el-cascader>
      <el-row>

      </el-row>
      <!-- Tabs  标签页 -->
      <!-- {{cateId}} -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <!-- 动态参数 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" stripe border>
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- {{scope.row}} -->
                <el-tag closable v-for="(item,i) in scope.row.attr_vals" :key="item.attr_id" @close="handleClose(scope.row,i)">{{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                <!-- 124.2.35 -->
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>

            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" stripe border>
            <el-table-column type="expand">
              <template v-slot="scope">
                <!-- {{scope.row.attr_vals.split(',')}} -->
                <el-tag closable v-for="(item,i) in scope.row.attr_vals" :key="item.attr_id" @close="handleClose(scope.row,i)">{{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                <!-- 124.2.35 -->
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="静态属性" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>

            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数 属性 对话框 -->
    <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
      <el-form :model="addForm" :rules="addRules" ref="addRuleFormRef" label-width="100px">
        <el-form-item label="活动名称" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑 参数 属性 对话框 -->
    <el-dialog :title="'添加'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClose">
      <el-form :model="editForm" :rules="addRules" ref="editRuleFormRef" label-width="100px">
        <el-form-item label="活动名称" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editPutParams">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data() {
    return {
       // 商品分类的数据列表
      catelist: [],
      // 级联选择框的配置对象
      cateProps: {
        value:'cat_id',
        label:'cat_name',
        expandTrigger: 'hover'
      },
      paramsArr:[],
      // Tabs  标签页 默认选中项
      activeName: 'many',
      // 动态参数
      manyTableData:[],
      // 静态属性
      onlyTableData:[],
      //  添加参数 属性 对话框 de 显示与隐藏
      addDialogVisible: false,
      //  添加动态参数或者静态属性 de 配置对象
      addForm:{
        // 参数名称
        attr_name: ''
      },
      // 验证规则
      addRules: {
        attr_name: [
            { required: true, message: '请输入活动名称', trigger: 'blur' },
            { min: 1, max: 15, message: '长度在 1 到 15 个字符', trigger: 'blur' }
          ],
      },
      // 编辑 参数 属性 对话框de 显示与隐藏
      editDialogVisible: false,
      // 编辑提交参数 配置对象
      editForm: {},
      // 展开行数组
      manyTableDataArr:[],
      onlyTableDataArr: [], 
      
    }
  },
  methods: {
    // 获取商品列表分类
    async getCateList() {
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status!==200) return this.$message.error('获取商品分类数据失败')
      // console.log(res);
      this.catelist = res.data
    },
    // 级联选择框 变化 触发
    async handleChange() {
      // console.log(this.paramsArr);
      if(this.paramsArr.length!==3) {
        this.manyTableData = []
        this.onlyTableData = []
        this.paramsArr = []
        return
      }
      this.getParamsData()
    },
    // Tabs  标签页  改变触发
    handleClick() {
      // console.log(this.activeName);
      if(this.paramsArr.length!==3) {
        return
      }
      this.getParamsData()
      
    },
    // 获取参数列表函数
    async getParamsData() {
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}})
      if(res.meta.status!==200) return this.$message.error('获取参数列表失败')
      // console.log(res);
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals?item.attr_vals.split(','):[]
        item.inputVisible = false
        item.inputValue = ''
      })
      if(this.activeName==='many') {
        this.manyTableData = res.data
        // this.manyTableDataArr = res.data.attr_vals.split(',')
        // console.log(res.data);
      } else {
        this.onlyTableData = res.data
        // this.onlyTableDataArr = res.data.attr_vals.split(',')
      }
    },
    // 监听 添加对话框的关闭事件
    addDialogClose(){
      this.$refs.addRuleFormRef.resetFields()
    },
    // 点击按钮 添加参数
    addParams(){
      this.$refs.addRuleFormRef.validate( async valid => {
        if(!valid) return
        const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{attr_name:this.addForm.attr_name,attr_sel:this.activeName})
        if(res.meta.status!==201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    // 点击 编辑按钮 弹出编辑对话框
    async showEditDialog(id){
      // console.log(id);
      // 查询当前参数 显示在对话框中
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${id}`,{params:{attr_sel:this.activeName}})
      if(res.meta.status!==200) return this.$message.error('获取参数失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 提交编辑
    editPutParams(){
      this.$refs.editRuleFormRef.validate( async valid => {
        if(!valid) return
        const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{attr_name:this.editForm.attr_name,attr_sel:this.activeName})
        if(res.meta.status!==200) return this.$message.error('修改属性失败')
        this.$message.success('修改属性成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // editDialogClose 重置编辑表单
    editDialogClose() {
      this.$refs.editRuleFormRef.resetFields()
    },
    // 根据id 删除参数
    removeParams(id) {
      this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then( async () => {
          const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
          if(res.meta.status!==200) return this.$message.error('删除失败')
          this.getParamsData()
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
    },
    // 失去焦点活 按下 enter 
    async handleInputConfirm(row) {
      // console.log('ok');
      if(row.inputValue.trim().length===0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // console.log(row.attr_vals);
      // console.log(row.attr_vals.join(' '));
      const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
        
      })
      console.log(row.attr_vals.join(','));
      if(res.meta.status!==200) return this.$message.error('修改参数失败')
      this.$message.success('修改参数项成功')
    },
    // 展示输入框 获得焦点
    showInput(row){
      row.inputVisible = true
      this.$nextTick(()=>{
        this.$refs.saveTagInput.focus()
      })
      
    },
    // 监听Tag close 关闭事件
    async handleClose(row,i){
      row.attr_vals.splice(i,1)
      // console.log(i);
      // console.log('ok');
      const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
        
      })
      console.log(row.attr_vals.join(','));
      if(res.meta.status!==200) return this.$message.error('修改参数失败')
      this.$message.success('修改参数项成功')
    }
  },
  // created 生命周期函数
  created() {
    this.getCateList()
  },
  // 计算属性
  computed: {
    // 按钮是否禁用
    isBtnDisabled() {
      if (this.paramsArr.length===3) {
        return false
      } else {
        return true
      }
    },
    // 动态计算 分类id
    cateId() {
      return this.paramsArr[this.paramsArr.length-1]
    },
    // 动态计算 添加属性 参数 对话框的 标题
    titleText() {
      if(this.activeName==='many') {
        return '动态参数'
      }
      return '静态属性'
    }
  },
}
</script>

<style lang="less" scoped>
.el-alert {
  margin-bottom: 15px;
}
.el-table {
  margin-top: 15px;
}
.el-tag {
  margin: 0 10px;
}
.input-new-tag {
  width: 150px;
}
</style>