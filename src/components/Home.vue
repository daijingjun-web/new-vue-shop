<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button class="homebtn" type="info" @click="logout">退出登录</el-button>
    </el-header>

    <el-container>
      <el-aside :width="isCollapse? '64px': '200px'">
        <div class="toggle_btn" @click="toggleAside">|||</div>
        <el-menu :default-active="activePath" background-color="#333744" text-color="#fff" active-text-color="#409eff" unique-opened :collapse="isCollapse" :collapse-transition="false" router>
          <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">

            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>

            <el-menu-item :index="subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveSubItem(subItem.path)">
              <i class="el-icon-menu"></i>
              {{subItem.authName}}
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>

  </el-container>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      menulist: [],
      iconsObj: {
        125:'el-icon-user-solid',
        103:'el-icon-key',
        101:'el-icon-s-goods',
        102:'el-icon-s-claim',
        145:'el-icon-s-marketing',
      },
      isCollapse:false,
      activePath:''
    }
  },
  created() {
    this.getMenulist()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout(){
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenulist(){
      const {data:res} = await this.$http.get('menus')
      // console.log(res.data);
      if(res.meta.status!==200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    toggleAside(){
      this.isCollapse=!this.isCollapse
    },
    saveSubItem(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath = activePath
    }
  },
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  background-color: #373d41;
  justify-content: space-between;
  align-items: center;
  >div {
    display: flex;
    align-items: center;
    span {
      color: #fff;
      font-size: 20px;
      margin-left: 10px;
    }
  }
}
.el-aside {
  background-color: #333744;
  .toggle_btn {
    color: #fff;
    background-color: #4a5064;
    font-size: 10px;
    text-align: center;
    line-height: 24px;
    letter-spacing: 3px;
    cursor: pointer;
  }
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.homebtn {
  justify-content: flex-end;
}
</style>