<template>
  <div>
    <el-form class="register-form" :model="registerInfo" :rules="rules" ref="ruleForm">
      <div class="avatar" @click="setShowChooseAvatar(true)">
        <img :src="avatar" alt="" srcset="" width="100" height="100" style="border-radius: 50%" />
        <span class="secondary-font" style="display: inline-block; margin-bottom: 5px"> 点击头像切换头像 </span>
      </div>
      <el-form-item prop="username">
        <el-input
          type="text"
          autocomplete="new-password"
          v-model="registerInfo.username"
          prefix-icon="el-icon-user"
          placeholder="请输入账号"
        ></el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          type="text"
          autocomplete="new-password"
          onfocus="this.type = 'password'"
          v-model="registerInfo.password"
          prefix-icon="el-icon-lock"
          placeholder="请输入密码"
        ></el-input>
      </el-form-item>
      <el-form-item prop="rePassword">
        <el-input
          type="text"
          autocomplete="new-password"
          onfocus="this.type = 'password'"
          v-model="registerInfo.rePassword"
          prefix-icon="el-icon-lock"
          placeholder="请确认密码"
        ></el-input>
        <!-- <span class="password-errinfo">{{ errInfo.password }}</span> -->
      </el-form-item>
      <el-form-item class="cv-code" prop="cvCode">
        <el-input
          class="cv-code-inp"
          v-model="registerInfo.cvCode"
          prefix-icon="el-icon-circle-check"
          placeholder="验证码(不区分大小写)"
        ></el-input>
        <canvas width="120" height="40" ref="registerCanvas" @click="getCVCode"></canvas>
      </el-form-item>
      <el-form-item class="oper">
        <el-button class="login-btn" type="primary" @click="register">注册</el-button>
        <span
          >已有账号？<span class="operation-text" style="display: inline" @click="changeState(true)">登录</span></span
        >
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import backimg from './../../static/image/bc.jpg'
import { createCanvas } from '@/utils/cvcode'
import canvasImg from './../../static/image/canvas2.jpg'
import { usernameReg, passwordReg } from '@/utils/index'
import avatarChoose from '@/components/avatarChoose'
import copyRight from '@/components/copyright'

const faceRandom = Math.ceil(Math.random() * 10)
export default {
  name: 'Register',
  props: {},
  data() {
    return {
      registerInfo: {
        username: '',
        password: '',
        rePassword: '',
        cvCode: '',
        avatar: `face/face${faceRandom}.jpg` //默认为随机值
      },
      cvCode: '', // 验证码
      cvCodeIng: true, // 正在获取验证码？
      isLoginState: true,
      bgUrl: backimg,
      showChooseAvatar: false,
      IMG_URL: process.env.IMG_URL,
      ruleForm: {
        username: '',
        password: '',
        rePassword: '',
        cvCode: ''
      },
      rules: {
        username: [{ required: true, message: '请输入账号', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        rePassword: [{ required: true, message: '请确认密码', trigger: 'blur' }],
        cvCode: [{ required: true, message: '请输入验证码', trigger: 'blur' }]
      }
    }
  },
  computed: {
    device() {
      return this.$store.state.device.deviceType
    },
    avatar() {
      return this.IMG_URL + this.registerInfo.avatar
    }
  },
  components: {
    avatarChoose,
    copyRight
  },
  methods: {
    login() {
      if (!usernameReg.test(this.loginInfo.username)) {
        return this.$message.error('请输入3-6位由数字字母下划线组成的账号')
      }
      if (!passwordReg.test(this.loginInfo.password)) {
        return this.$message.error('请输入6-16位由数字字母组成的密码')
      }
      //获取ip和国家名
      const returnCitySN = window.returnCitySN ? window.returnCitySN : {}
      const params = {
        ...this.loginInfo,
        setting: {
          os: window.OSInfo,
          browser: window.Browser,
          ip: returnCitySN['cip'],
          country: returnCitySN['cname']
        }
      }
      // console.log("登录的请求参数为：", params)
      this.$http.login(params).then((res) => {
        // console.log("登录返回结果为：", res.data)
        let { code, message, data } = res.data
        // 验证码错误或失效重新获取验证码
        if (code === 1007) {
          this.loginInfo.cvCode = ''
          this.getCVCode()
          return
        }
        if (res.status === 200 && code === 1000) {
          this.$message.success('登录成功！')
          this.$store.dispatch('user/LOGIN', data.userInfo)
          const redirect = this.$router.currentRoute.query.redirect
          const next = redirect ? redirect : '/chat/home'
          this.$router.replace(next)
        } else {
          this.$message.error(message)
          if (code === 1200) {
            this.$confirm(`你的${message}，如需恢复请联系管理员：dxm3342@163.com`, `通知：${message}`, {
              // confirmButtonText: '确定',
              // cancelButtonText: '取消',
              type: 'error'
            })
          }
        }
      })
    },
    register() {
      if (!usernameReg.test(this.registerInfo.username)) {
        return this.$message.error('请输入3-6位由数字字母下划线组成的账号')
      }
      if (!passwordReg.test(this.registerInfo.password)) {
        return this.$message.error('请输入6-16位由数字字母组成的密码')
      }
      if (this.registerInfo.password !== this.registerInfo.rePassword) {
        return this.$message.error('两次输入的密码不一致')
      }
      this.$http.register(this.registerInfo).then((res) => {
        let { code, message } = res.data
        if (code !== 1005) {
          this.$message.error(message)
          code === 1007 ? this.getCVCode() : ''
        } else if (code === 1005) {
          this.$alert(`这是你的chat账号:${res.data.data.userCode}，你可以以此账号登录系统`, '注册成功')
          this.isLoginState = true
          this.getCVCode()
        }
      })
    },
    getCVCode() {
      // 获取验证码
      this.cvCodeIng = true
      this.$http.getCVCode().then((res) => {
        this.cvCode = res.data.data.code
        this.$nextTick(() => {
          //  const currCanvas = this.isLoginState ? this.$refs.loginCanvas : this.$refs.registerCanvas
          const currCanvas = this.$refs.registerCanvas
          createCanvas(this.cvCode, currCanvas, canvasImg, () => {
            this.cvCodeIng = false
          })
        })
      })
    },
    changeState(flag) {
      this.$emit('changeState', flag)
      this.getCVCode()
    },
    setShowChooseAvatar(flag) {
      this.showChooseAvatar = flag
    },
    chooseAvatar(item) {
      this.registerInfo.avatar = item
    }
  },
  async mounted() {
    this.getCVCode()
  }
}
</script>

<style lang="scss">
.login-page {
  @import './../../static/css/animation.scss';
  @import './../../static/css/var.scss';
  height: 100vh;
  background-repeat: no-repeat;
  background-size: cover;
  transition: all 0.8s ease;

  .ceshi {
    display: none;
    position: absolute;
    top: 50px;
    left: 50%;
    transform: translateX(-50%);
    margin: 0 auto;
    width: 60%;
  }

  .wrapper {
    background-color: #fff;
    width: 400px;
    padding: 35px 20px 0;
    border-radius: 5px;

    .login-form,
    .register-form {
      position: relative;

      .avatar {
        position: absolute;
        z-index: 1001;
        top: -110px;
        text-align: center;
        margin-bottom: 10px;

        .el-avatar {
          box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
        }
      }
    }

    .cv-code {
      .el-form-item__content {
        display: flex;
        justify-content: space-between;

        .cv-code-inp {
          margin-right: 20px;
        }
      }
    }

    .login-btn {
      width: 100%;
    }
  }
}
</style>

