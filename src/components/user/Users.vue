<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true">添加新用户</el-button>
        </el-col>
      </el-row>
    

    <el-table :data="userlist" border stripe>
      <el-table-column label="姓名" prop="username"></el-table-column>
      <el-table-column label="邮箱" prop="email"></el-table-column>
      <el-table-column label="电话" prop="mobile"></el-table-column>
      <el-table-column label="角色" prop="role_name"></el-table-column>
      <el-table-column label="状态" prop="mg_state">
        <template v-slot="scope">
          <!-- {{scope.row.mg_state}} -->
          <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
          <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
    </el-card>
<!-- 添加用户对话框 -->
    <el-dialog
  title="添加新用户"
  :visible.sync="dialogVisible"
  width="40%" @close="addDialogClosed">

  <el-form :model="addForm" :rules="addFormRules" ref="addruleForm" label-width="70px" class="demo-ruleForm">
  <el-form-item label="用户名" prop="username">
    <el-input v-model="addForm.username"></el-input>
  </el-form-item>
  <el-form-item label="密码" prop="password">
    <el-input v-model="addForm.password"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="addForm.email"></el-input>
  </el-form-item>
  <el-form-item label="手机号" prop="mobile">
    <el-input v-model="addForm.mobile"></el-input>
  </el-form-item>
  </el-form>

  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
</el-dialog>
<!-- 修改用户对话框 -->
<el-dialog
  title="修改用户信息"
  :visible.sync="editDialogVisible"
  width="30%" @close="editDialogClose">
  <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">

   <el-form-item label="用户名">
    <el-input v-model="editForm.username" disabled></el-input>
   </el-form-item>
   <el-form-item label="邮箱" prop="email">
    <el-input v-model="editForm.email"></el-input>
   </el-form-item>
   <el-form-item label="手机号" prop="mobile">
    <el-input v-model="editForm.mobile"></el-input>
   </el-form-item>

  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Users',
  data() {
    // 验证邮箱正则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/g
        if (regEmail.test(value)) {
          return callback()
        }
        callback(new Error('请输入合法的邮箱'))
    }
    // 验证手机号正则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/g
        if (regMobile.test(value)) {
          return callback()
        }
        callback(new Error('请输入合法的手机号'))
    }
    // 数据
    return {
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:2
      },
      userlist: [],
      total: 0,
      dialogVisible: false,
      addForm: {
        username:'',
        password:'',
        email: '',
        mobile:''
      },
      editForm:{},
      editDialogVisible:false,
      addFormRules: {
        username: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
        password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
          ],
        email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { validator: checkEmail, trigger: 'blur' }
          ],
        mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ],
      }
    }
  },
  methods: {
    async getUserList(){
      const {data:res} = await this.$http.get('users',{params:this.queryInfo})
      if(res.meta.status!==200) return this.$message.error(res.meta.msg)
      this.userlist = res.data.users
      this.total = res.data.total
      // console.log(res.data);
    },
    // 页码大小
    handleSizeChange(newSize) {
      // console.log(newSize);
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 当前页变动时候触发
    handleCurrentChange(newPage){
      // console.log(newPage);
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChanged(userinfo) {
      // console.log(userinfo);
      const {data:res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      console.log(res);
      if(res.meta.status!==200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('操作失败')
      }
      this.$message.success('更新用户状态成功')
    },
    addDialogClosed() {
      this.$refs.addruleForm.resetFields()
    },
    addUser() {
      this.$refs.addruleForm.validate( async valid => {
        if(!valid) return
        const {data:res} = await this.$http.post('users',this.addForm)
        console.log(res);
        if(res.meta.status!==201) return this.$message.error('添加新用户失败')
        this.$message.success('添加新用户成功')
        this.dialogVisible = false
        this.getUserList()
      })
    },
    async showEditDialog(id){
      // console.log(id);
      const {data:res} = await this.$http.get('users/'+id)
      // console.log(res);
      if(res.meta.status!==200) return this.$message.error('查询用户信息失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 修改表单重置操作
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editUserInfo() {
      this.$refs.editFormRef.validate( async valid=>{
        if(!valid) return
        const {data:res} =await this.$http.put('users/'+this.editForm.id,{
          email:this.editForm.email,
          mobile:this.editForm.mobile
        })
        // console.log(res);
        if(res.meta.status!==200) return this.$message.error('更新用户信息失败！')
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('更新用户信息成功！')
      })
    },
    removeUserById(id) {
      // console.log(id);
       this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then( async () => {
          // console.log('删除成功');
          const {data:res} = await this.$http.delete('users/' + id)
          console.log(res);
          if(res.meta.status!==200) return this.$message.error('删除用户失败！')
          this.getUserList()
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
    }
  },
  created() {
    this.getUserList()
  },
}
</script>

<style lang="less" scoped>
.el-pagination {
  margin-top: 15px;
}
.el-table {
  margin-top: 15px;
}
</style>