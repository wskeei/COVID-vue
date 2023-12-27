<template>
  <div>
    <el-card class="login-form-layout">
      <el-form
        autocomplete="on"
        :model="loginForm"
        ref="loginForm"
        label-position="left"
      >
        <div style="text-align: center">
          <svg-icon icon-class="login-mall" style="width: 56px;height: 56px;color: #409EFF"></svg-icon>
        </div>
        <h2 class="login-title color-main">疫情防控管理系统登录</h2>
        <el-form-item prop="username">
          <el-input
            name="username"
            type="text"
            v-model="loginForm.username"
            autocomplete="on"
            placeholder="请输入用户名"
          >
            <span slot="prefix">
              <svg-icon icon-class="user" class="color-main"></svg-icon>
            </span>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            name="password"
            :type="pwdType"
            @keyup.enter.native="handleLogin"
            v-model="loginForm.password"
            autocomplete="on"
            placeholder="请输入密码"
            show-password
          >
            <span slot="prefix">
              <svg-icon icon-class="password" class="color-main"></svg-icon>
            </span>
            <span slot="suffix" @click="showPwd">
              <svg-icon icon-class="eye" class="color-main"></svg-icon>
            </span>
          </el-input>
        </el-form-item>

        <el-form-item>
      <!-- 添加一个包含验证码的容器，并设置样式 -->
      <div style="display: flex; align-items: center;">
        <el-input
          v-model="captchaInput"
          autocomplete="off"
          placeholder="请输入验证码"
        ></el-input>
        <canvas ref="captchaCanvas" @click="refreshCaptcha" style="cursor: pointer; margin-left: 10px; border: 1px solid #ccc;"></canvas>
      </div>
    </el-form-item>
    <el-form-item style="margin-bottom: 60px">
      <el-button
        style="width: 48%"
        type="primary"
        :loading="loading"
        @click.native.prevent="handleLogin"
      >登录</el-button>
      <el-button
        style="width: 48%"
        type="success"
        :loading="loading"
        @click="dialogFormVisible = true"
      >注册</el-button>
    </el-form-item>
  </el-form>
</el-card>
</div>
</template>

<script>
import axios from "axios";

export default {
  created() {
    axios.get("http://localhost:8080/depart/findAll").then((resp) => {
      this.options2 = resp.data;
    });
  },
  data() {
    return {
      options2: [
        {
          value: "",
          label: "",
        },
      ],
      formLabelWidth: "120px",
      LabelWidth: "40px",
      dialogFormVisible: false,
      user: {
        username: "",
        password: "",
        repassword: "",
        depart: "",
      },
      loginForm: {
        username: "",
        password: "",
      },
      loading: false,
      pwdType: "password",
      captchaInput: "",
    };
  },
  methods: {
    refreshCaptcha() {
      const canvas = this.$refs.captchaCanvas;
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      const captchaCode = this.generateRandomCode();
      this.drawCaptcha(ctx, captchaCode);
    },

    generateRandomCode() {
      const characters = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";      
      const length = 6;
      let captchaCode = "";
      for (let i = 0; i < length; i++) {
        const randomIndex = Math.floor(Math.random() * characters.length);
        captchaCode += characters.charAt(randomIndex);
      }
      return captchaCode;
    },

    drawCaptcha(ctx, captchaCode) {
      ctx.font = "70px Arial";
      ctx.fillText(captchaCode, 25, 85);
      this.captchaCode = captchaCode;
    },

    register() {
      this.dialogFormVisible = false;
      if (this.user.username == "" || this.user.password == "") {
        this.$alert("注册用户名或密码不能为空");
        this.loading = false;
      } else if (this.user.password == this.user.repassword) {
        axios.post('http://localhost:8080/userlogin/register', this.user).then((resp) => {
          console.log(resp);
          this.loading = true;
          if (resp.data == 'success') {
            this.loading = false;
            this.$alert("注册成功!请登录");
          } else if (resp.data == 'repeat') {
            this.loading = false;
            this.$alert('用户名已存在!请重新注册');
          } else {
            this.loading = false;
            this.$alert("注册失败!请重新注册");
            window.location.reload();
          }
        });
      } else {
        this.$alert("两次输入的密码不一致!");
      }
    },

    showPwd() {
      if (this.pwdType === "password") {
        this.pwdType = "";
      } else {
        this.pwdType = "password";
      }
    },

    handleLogin() {
      // 在这里添加验证码验证逻辑
      if (this.captchaInput === this.captchaCode) {
        // 验证码正确，执行登录逻辑
        axios.post('http://localhost:8080/userlogin/user', this.loginForm).then((resp) => {
          console.log(resp);
          this.loading = true;
          if (this.loginForm.username == "" || this.loginForm.password == "") {
            this.$alert("用户名或密码不能为空");
            this.loading = false;
          } else {
            if (resp.data == 'success') {
              this.loading = false;
              this.$message({
                showClose: true,
                message: '登录成功',
                type: 'success'
              });
              this.$router.push('/index');
            } else {
              this.loading = false;
              this.$alert("用户名或密码错误");
            }
          }
        });
      } else {
        // 验证码错误，给出提示或阻止登录
        this.$message.error("验证码错误");
      }
    },
  },
};
</script>

<style scoped>
.login-form-layout {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-top: 10px solid #409eff;
  background-image: url('../assets/1.jpg'); /* 相对路径 */
  background-size: cover; /* 调整背景尺寸 */
  background-repeat: no-repeat; /* 不重复显示背景 */
  background-position: center; /* 居中显示背景 */
}

.login-title {
  text-align: center;
}

.login-center-layout {
  background: #409eff;
  width: 360px; /* 或您想要的宽度 */
  padding: 20px; /* 或您想要的内边距 */
  border-radius: 8px; /* 或其他您想要的边框半径 */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* 添加一些阴影效果 */
}

/* 添加或调整验证码框的样式 */
.login-center-layout .verification-code {
  margin-bottom: 15px; /* 调整验证码框底部的间距 */
}

.login-center-layout input[type="text"].verification-code {
  width: 100%; /* 设置验证码输入框宽度为100% */
  /* 添加其他样式，如边框、内边距等 */
}
</style>