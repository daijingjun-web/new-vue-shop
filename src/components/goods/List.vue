<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- Card 视图区 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table 表格区 -->
      <el-table :data="goodslist" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格（元）" width="100px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="110px"></el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="180px">
          <template v-slot="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot="scope">
            <!-- {{scope.row}} -->
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="editGoodsById(scope.row.goods_id)"></el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeGoodsById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区 -->
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
 data() {
  return {
    // 商品列表数据 的 查询参数
    queryInfo: {
      query: '',
      pagenum: 1,
      pagesize: 10
    },
    // 商品列表数据
    goodslist: [],
    // 商品列表 总条数
    total: 0
  }
 },
 methods: {
  // 获取商品列表数据
  async getGoodsList() {
    const {data:res} = await this.$http.get('goods',{params:this.queryInfo})
    // console.log(res);
    this.goodslist = res.data.goods
    this.total = res.data.total
  },
  // 每页条数 改变 触发
  handleSizeChange(newSize) {
    this.queryInfo.pagesize = newSize
    this.getGoodsList()
  },
  // 当前页 改变 触发
  handleCurrentChange(newPage) {
    this.queryInfo.pagenum = newPage
    this.getGoodsList()
  },
  // 
  removeGoodsById(id) {
    this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then( async () => {
          // console.log(id);
          const {data:res} = await this.$http.delete('goods/'+ id)
          // console.log(res);
          if(res.meta.status!==200) return this.$message.error('删除商品失败')
          this.$message.success('删除商品成功')
          this.getGoodsList()
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
  goAddPage() {
    this.$router.push('add')
  }
 },
 created() {
  this.getGoodsList()
 },
}
</script>

<style lang="less" scoped>
.el-table {
  margin: 20px 0;
}
</style>