<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-table :data="rightsList" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template v-slot="scope">
            <el-tag v-if="scope.row.level==0">一级</el-tag>
            <el-tag type="success" v-else-if="scope.row.level==1">二级</el-tag>
            <el-tag type="warning" v-else>三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Rights',
  data() {
    return {
      rightsList:[]
    }
  },
  methods: {
    async getRightsList(){
      const {data:res} = await this.$http.get('rights/list')
      // console.log(res);
      if(res.meta.status!==200) return this.$message.error('获取权限列表失败！')
      this.rightsList = res.data
    }
  },
  created() {
    this.getRightsList()
  },
}
</script>

<style>

</style>