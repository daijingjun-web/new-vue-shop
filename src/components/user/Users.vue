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
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary">添加新用户</el-button>
        </el-col>
      </el-row>

    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Users',
  data() {
    return {
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:2
      },
      userlist: [],
      total: 0
    }
  },
  methods: {
    async getUserList(){
      const {data:res} = await this.$http.get('users',{params:this.queryInfo})
      console.log(res);
      if(res.meta.status!==200) return this.$message.error(res.meta.msg)
      this.userlist = res.data.users
      this.total = res.data.total
    }
  },
  created() {
    this.getUserList()
  },
}
</script>

<style>

</style>