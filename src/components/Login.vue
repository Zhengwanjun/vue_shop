<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 登录表单区域 -->
      <el-form
        ref="loginFormRef"
        :model="loginForm"
        :rules="loginFormRules"
        class="login_form"
      >
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="iconfont icon-user"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            type="password"
            prefix-icon="iconfont icon-3702mima"
          ></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //登录表单的数据绑定对象
      loginForm: {
        username: "admin",
        password: "123456",
      },
      //表单验证规则对象
      loginFormRules: {
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          { min: 3, max: 10, message: "长度在3-10个字符", trigger: "blur" },
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, max: 15, message: "长度在6-15个字符", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    //表单重置
    resetLoginForm() {
      // console.log(this);
      this.$refs.loginFormRef.resetFields()
    },
    login() {
      this.$refs.loginFormRef.validate(async(valid) => {
        // console.log(valid);
        if(!valid) return
        //对象解构 将data重命名为res
        const {data:res} = await this.$http.post('login',this.loginForm)
        //弹窗效果 $message
        if(res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')

        //1.token 保存到客户端sessionStorage
        //  项目中出了登陆之外的其他api接口，必须在登录之后才能访问
        //  token只在当前网站打开期间生效，所以将token存在sessionStorage中
        console.log(res);
        window.sessionStorage.setItem('token',res.data.token)
        //2.通过编程式导航跳转到后台主页，路由地址是/home
        this.$router.push('/home')


      })
    }
  }
};
</script>

<style scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;

  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.login_form {
  position: absolute;
  bottom: 0;
  box-sizing: border-box;
  width: 100%;
  padding: 0 20px;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>