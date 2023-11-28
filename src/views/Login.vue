<template>
  <div class="login-page" opacity:0.8 :style="'background-image: url(' + bgUrl + ')'">
    <div class="ceshi" style="'marginTop': '30px'">
      <el-alert
        :closable="false"
        show-icon
        title="测试账号密码"
        description="账号1：cc1218，密码1：123456 | 账号2：lt0623，密码2：123456"
        type="success"
      />
    </div>
    <transition name="fade">
      <avatar-choose v-if="showChooseAvatar" @close="setShowChooseAvatar(false)" @choose="chooseAvatar" />
    </transition>
    <div class="wrapper hor-ver-center" :style="device === 'Mobile' ? { width: '90%' } : {}">
      <el-form class="login-form" :model="loginInfo" :rules="rules" ref="ruleForm" v-if="isLoginState">
        <div class="avatar">
          <el-avatar :size="100" :src="IMG_URL + loginInfo.avatar" @error="() => true">
            <img src="https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png" />
          </el-avatar>
        </div>
        <el-form-item prop="username">
          <el-input
            autocomplete="new-password"
            v-model="loginInfo.username"
            prefix-icon="el-icon-user"
            @keydown.enter="login"
            placeholder="请输入账号"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            autocomplete="new-password"
            type="password"
            v-model="loginInfo.password"
            prefix-icon="el-icon-lock"
            placeholder="请输入密码"
          ></el-input>
        </el-form-item>
        <el-form-item class="cv-code" prop="cvCode">
          <el-input
            autocomplete="on"
            class="cv-code-inp"
            v-model="loginInfo.cvCode"
            @keydown.enter.native="login"
            prefix-icon="el-icon-circle-check"
            placeholder="验证码(不区分大小写)"
          ></el-input>
          <canvas v-show="!cvCodeIng" width="120" height="40" ref="loginCanvas" @click="getCVCode"></canvas>
          <span style="width: 200px" v-show="cvCodeIng" @click="getCVCode">获取中...</span>
        </el-form-item>
        <el-form-item>
          <el-button class="login-btn" type="primary" @click="login">登录</el-button>
          <span
            >没有账号？<span class="operation-text" style="display: inline" @click="changeState(false)"
              >注册</span
            ></span
          >
        </el-form-item>
      </el-form>
      <Register v-if="!isLoginState" @changeState="changeState"></Register>
    </div>
    <copy-right />
  </div>
</template>


<script>
import backimg from './../../static/image/bc.jpg'
import { createCanvas } from '@/utils/cvcode'
import canvasImg from './../../static/image/canvas2.jpg'
import { usernameReg, passwordReg } from '@/utils/index'
import avatarChoose from '@/components/avatarChoose'
import copyRight from '@/components/copyright'
import Register from './Register.vue'

const faceRandom = Math.ceil(Math.random() * 10)
export default {
  name: 'Login',
  components: { Register, avatarChoose, copyRight },
  data() {
    return {
      loginInfo: {
        username: '',
        password: '',
        cvCode: '',
        avatar: JSON.parse(window.localStorage.getItem('userInfo') || '{}').photo || ''
      },
      // registerInfo: {
      //   username: '',
      //   password: '',
      //   rePassword: '',
      //   cvCode: '',
      //   avatar: `face/face${faceRandom}.jpg` //默认为随机值
      // },
      cvCode: '', // 验证码
      cvCodeIng: true, // 正在获取验证码？
      isLoginState: true,
      bgUrl: backimg,
      showChooseAvatar: false,
      IMG_URL: process.env.IMG_URL,
      ruleForm: {
        username: '',
        password: '',
        cvCode: ''
      },
      rules: {
        username: [{ required: true, message: '请输入账号', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        cvCode: [{ required: true, message: '请输入验证码', trigger: 'blur' }]
      }
    }
  },
  computed: {
    device() {
      return this.$store.state.device.deviceType
    }
    // avatar() {
    //   return this.IMG_URL + this.registerInfo.avatar
    // }
  },
  methods: {
    login() {
      // if (!usernameReg.test(this.loginInfo.username)) {
      //   return this.$message.error('请输入3-6位由数字字母下划线组成的账号')
      // }
      // if (!passwordReg.test(this.loginInfo.password)) {
      //   return this.$message.error('请输入6-16位由数字字母组成的密码')
      // }
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
    // register() {
    //   if (!usernameReg.test(this.registerInfo.username)) {
    //     return this.$message.error('请输入3-6位由数字字母下划线组成的账号')
    //   }
    //   if (!passwordReg.test(this.registerInfo.password)) {
    //     return this.$message.error('请输入6-16位由数字字母组成的密码')
    //   }
    //   if (this.registerInfo.password !== this.registerInfo.rePassword) {
    //     return this.$message.error('两次输入的密码不一致')
    //   }
    //   this.$http.register(this.registerInfo).then((res) => {
    //     let { code, message } = res.data
    //     if (code !== 1005) {
    //       this.$message.error(message)
    //       code === 1007 ? this.getCVCode() : ''
    //     } else if (code === 1005) {
    //       this.$alert(`这是你的chat账号:${res.data.data.userCode}，你可以以此账号登录系统`, '注册成功')
    //       this.isLoginState = true
    //       this.getCVCode()
    //     }
    //   })
    // },
    getCVCode() {
      // 获取验证码
      this.cvCodeIng = true
      this.$http.getCVCode().then((res) => {
        this.cvCode = res.data.data.code
        this.$nextTick(() => {
          // const currCanvas = this.isLoginState ? this.$refs.loginCanvas : this.$refs.registerCanvas
          const currCanvas = this.$refs.loginCanvas
          createCanvas(this.cvCode, currCanvas, canvasImg, () => {
            this.cvCodeIng = false
          })
        })
      })
    },
    changeState(flag) {
      this.isLoginState = flag
      this.getCVCode()
    },
    setShowChooseAvatar(flag) {
      this.showChooseAvatar = flag
    }
    // chooseAvatar(item) {
    //   this.registerInfo.avatar = item
    // }
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
