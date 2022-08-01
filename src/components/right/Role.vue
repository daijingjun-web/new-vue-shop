<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom:15px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRole">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolelist" stripe border>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id"  :class="['bdbottom',i1===0?'bdtop':'','vcenter']">
              <!-- 一级 -->
              <el-col :span="5">
                <el-tag  closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二三级 -->
              <el-col :span="19">
                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2===0?'':'bdtop','vcenter']">

                  <el-col :span="6">
                    <el-tag type="success"  closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <el-col :span="18">
                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="editRole(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showRightDialog(scope.row)">权限分配</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加新角色弹框 -->
    <el-dialog
  title="添加新角色"
  :visible.sync="addRoleDialogVisible"
  width="30%" @close="addRoleClose">
  <el-form :model="addRoleList" :rules="addRoleRules" ref="addRoleRef" label-width="90px">
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="addRoleList.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addRoleList.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addRoleDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addNewRole">确 定</el-button>
  </span>
</el-dialog>

  <!-- 编辑角色弹框 -->
    <el-dialog
  title="编辑角色"
  :visible.sync="editPutRoleVisible"
  width="30%" @close="editPutRoleClose">
  <el-form :model="editPutRole" :rules="addRoleRules" ref="editPutRoleRef" label-width="90px">
  <el-form-item label="角色ID">
    <el-input v-model="editPutRole.roleId" disabled></el-input>
  </el-form-item>
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="editPutRole.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="editPutRole.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editPutRoleVisible = false">取 消</el-button>
    <el-button type="primary" @click="editPutRoles">确 定</el-button>
  </span>
</el-dialog>

<!-- 权限分配弹窗  -->
<el-dialog
  title="权限分配"
  :visible.sync="showRightDialogVisible"
  width="45%" @close="setRightDialogClose">
  <el-tree :data="rightslist" :props="defaultProps" show-checkbox node-key="id" :default-expand-all="true" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="showRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>

  </div>
</template>

<script>
export default {
  name: 'Role',
  data() {
    return {
      rolelist: [],
      // 添加新角色
      addRoleList:{
        roleName:'',
        roleDesc:''
      },
      addRoleDialogVisible: false,
      editPutRoleVisible: false,
      addRoleRules:{
        roleName: [
            { required: true, message: '请输入角色名', trigger: 'blur' },
            { min: 2, max: 5, message: '长度在 2 到 8 个字符', trigger: 'blur' }
          ],
        roleDesc: [
            { required: true, message: '请输入角色描述', trigger: 'blur' },
            { min: 3, max: 25, message: '长度在 3 到 25 个字符', trigger: 'blur' }
          ],
      },
      // 编辑角色
      editPutRole:{},
      // 权限配置
      showRightDialogVisible:false,
      // 所以权限列表 树形tree
      rightslist:[],
      // 树形控件的绑定 属性
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys:[],
      // 当前即将分配权限的角色id
      roleId:''
    }
  },
  methods: {
    // 获取角色列表
    async getRoleList(){
      const {data:res} = await this.$http.get('roles')
      // console.log(res);
      if(res.meta.status!==200) return this.$message.error('获取角色列表失败')
      this.rolelist = res.data
    },
    // 添加新角色弹框
    addRole(){
      this.addRoleDialogVisible = true
    },
    // 添加角色弹窗关闭 对整个表单进行重置
    addRoleClose(){
      this.$refs.addRoleRef.resetFields()
    },
    // 添加新角色
    addNewRole(){
      this.$refs.addRoleRef.validate( async valid=>{
        if(!valid) return
        const {data:res} =await this.$http.post('roles',this.addRoleList)
        // console.log(res);
        if(res.meta.status!==201) return this.$message.error('添加角色失败')
        this.$message.success('添加角色成功')
        this.getRoleList()
        this.addRoleDialogVisible = false
      })
      
    },
    // 编辑角色弹框关闭，对整个表单进行重置
    editPutRoleClose(){
      this.$refs.editPutRoleRef.resetFields()
    },
    // 编辑提交角色弹框
    async editRole(id){
      // console.log(id);
      const {data:res} =await this.$http.get('roles/' + id)
      // console.log(res);
      if(res.meta.status!==200) return
      this.editPutRole = res.data
      this.editPutRoleVisible = true
    },
    // 编辑提交角色操作
    editPutRoles(){
      this.$refs.editPutRoleRef.validate( async valid=>{
        if(!valid) return
        const {data:res} = await this.$http.put('roles/' + this.editPutRole.roleId,{roleName:this.editPutRole.roleName,roleDesc:this.editPutRole.roleDesc})
        // console.log(res);
        if(res.meta.status!==200) return this.$message.error('更新角色信息失败')
        this.editPutRoleVisible = false
        this.getRoleList()
        this.$message.success('更新角色信息成功')
      })
    },
    // 根据id 删除角色
    removeRoleById(id){
      // console.log(id);
      this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then( async () => {
          // console.log('删除成功');
          const {data:res} = await this.$http.delete('roles/' +id)
          // console.log(res);
          if(res.meta.status!==200) return this.$message.error('删除角色失败！')
          this.getRoleList()
          
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
    // 根据角色id ，权限id 删除指定角色的指定权限
    removeRightById(role,rightId){
      // console.log('ok');
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then( async () => {
          // console.log('删除成功');
          const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          // console.log(res);
          if(res.meta.status!==200) return this.$message.error('删除权限失败！')
          // this.getRoleList()
          role.children = res.data
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
    // 权限分配弹窗 
    async showRightDialog(role) {
      this.roleId = role.id
      const {data:res} = await this.$http.get('rights/tree')
      if(res.meta.status!==200) return this.$message.error('')
      this.rightslist = res.data
      this.getLeafKeys(role,this.defKeys)
      this.showRightDialogVisible = true;
    },
    // 定义递归函数  自己调用自己
    getLeafKeys(node,arr) {
      if(!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item=>{
        this.getLeafKeys(item,arr)
      })
    },
    // 监听权限分配弹框关闭事件 清空defKeys数组
    setRightDialogClose(){
      this.defKeys=[]
    },
    // 点击确定按钮 为角色分配权限
    async allotRights(){
      const keys = [...this.$refs.treeRef.getCheckedKeys(),...this.$refs.treeRef.getHalfCheckedKeys()]
      // console.log(keys);
      const idStr = keys.join(',')
      // console.log(idStr);
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status!==200) return this.$message.error('分配权限失败')
      this.$message.success('权限分配成功')
      this.getRoleList()
      this.showRightDialogVisible = false
    }
  },
  created() {
    this.getRoleList()
  },
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>