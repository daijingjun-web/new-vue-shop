<template>
  <div class="login_container">
    <div class="login_box">
      <!-- logo图片 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!-- 登录表单区域 -->
      <el-form :model="Loginform" :rules="loginRules" ref="loginForm" label-width="0px">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input prefix-icon="el-icon-user-solid" v-model="Loginform.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input prefix-icon="el-icon-lock" v-model="Loginform.password" type="password"></el-input>
        </el-form-item>
        <!-- 登录、重置按钮 -->
        <el-form-item class="btns">
          <el-button type="info" @click="resetForm">重置</el-button>
          <el-button type="primary" @click="login">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name:'Login',
  data() {
    return {
      Loginform:{
        username:'admin',
        password:'123456'
      },
      loginRules:{
        username: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
        password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, max: 16, message: '长度在 3 到 16 个字符', trigger: 'blur' }
          ],
      }
    }
  },
  methods: {
    // loginForm
    resetForm(){
      console.log(this.$refs);
      this.$refs.loginForm.resetFields()
    },
    login() {
      this.$refs.loginForm.validate( async valid=>{
        // console.log(valid);
        if(!valid) return
        const {data:res} = await this.$http.post('login',this.Loginform)
        // console.log(res);
        if(res.meta.status!==200) return this.$message.error('登录失败')
        // console.log('登录成功');
        this.$message.success('登录成功')
        window.sessionStorage.setItem('token',res.data.token)
        this.$router.push('/home')
      })
    }
  },
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 450px;
  height: 300px;
  background-color: #eee;
  border-radius: 3px;
}
.avatar_box {
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 130px;
  height: 130px;
  background-color: #eee;
  border: 1px solid rgb(215, 215, 215);
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #fff;
  img{
    width: 100%;
    height: 100%;
    background-color: rgb(224, 224, 224);
    border-radius: 50%;
  }
}
.el-form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>