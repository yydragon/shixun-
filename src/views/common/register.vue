<template>
  <div id="login">
    <el-form class="container" :model="dataForm" status-icon 
               label-width="100px">
      <h3 class="title">ChosenOne1 手机后台管理系统用户注册</h3>
      <el-form-item label="用户名：" prop="userName">
        <el-input  v-model="dataForm.userName" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="密码：" prop="password">
        <el-input type="password" v-model="dataForm.userPasswd" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="手机号：" prop="mobile">
        <el-input type="phone" v-model="dataForm.mobile" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="邮箱：" prop="userEmail">
        <el-input type="email" v-model="dataForm.userEmail" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="工厂ID：" prop="factoryId">
        <el-input v-model="dataForm.factoryId" autocomplete="off"></el-input>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="register()">注册</el-button>
        
      </el-form-item>
    </el-form>
  </div>
</template>
<script>

export default {
  name: 'Login',
  data () {

    return {
      dataForm: {
          userName: '',
          userPasswd:'',
          userEmail: '',
          mobile: '',
          userJobNum:'',
          factoryId:'',
        },
      
    }
  },
  created () {

  },
  methods: {
    showMessage (message) {
      this.$message.error(message)
    },
    
    register () {
      this.$axios.post("/api/user/register",this.dataForm)
        .then(response=>{
          this.$message({
            message: '注册成功',
            type: 'success',
          })
        })
        this.$router.push({ name: 'login' })
      }
      
    },

  }

</script>



<style>
  #login {
    background:url("../../assets/img/bird.jpg") no-repeat;
    background-position: center;
    height: 100%;
    width: 100%;
    background-size: cover;
    position: fixed;
  }

  body{
    margin: 0px;
  }

  .container {
    border-radius: 15px;
    background-clip: padding-box;
    margin: 8% auto;
    width: 350px;
    padding: 35px 35px 15px 35px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
  }

  .title {
    margin: 0px auto 40px auto;
    text-align: center;
    color: #505458;
  }

  .login-code {
    display: inline-block;
    height: 40px;
    float: right;
  }

  .validCode .el-input {
    float: left;
    width: 50%;
  }

  .el-link {
    margin-left: 25px;
  }

  img {
    float: right;
  }
</style>


