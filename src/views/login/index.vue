<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" autocomplete="on" label-position="left">

      <div class="title-container">
        <h3 class="title">iCode题库</h3>
      </div>

      <el-form-item prop="username">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <!-- 添加ref属性，后面可以用this.$refs.username获取此输入框对象，进而调用属性和相关方法 -->
        <el-input
          ref="username"
          v-model="loginForm.username"
          placeholder="用户名"
          name="username"
          type="text"
          tabindex="1"
          autocomplete="on"
        />
      </el-form-item>

      <el-tooltip v-model="capsTooltip" content="大写已打开" placement="right" manual>
        <el-form-item prop="password">
          <span class="svg-container">
            <svg-icon icon-class="password" />
          </span>
          <el-input
            :key="passwordType"
            ref="password"
            v-model="loginForm.password"
            :type="passwordType"
            placeholder="Password"
            name="password"
            tabindex="2"
            autocomplete="on"
            @keyup.native="checkCapslock"
            @blur="capsTooltip = false"
            @keyup.enter.native="handleLogin"
          />
          <span class="show-pwd" @click="showPwd">
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
          </span>
        </el-form-item>
      </el-tooltip>

      <el-checkbox v-model="loginForm.remember" style="margin-bottom: 20px; margin-left: 5px">记住密码</el-checkbox>

      <!--登录按钮-->
      <!--不是原生标签点击需要添加.native修饰，避免点击失效；.prevent提交事件不会重载页面-->
      <el-button
        :loading="loading"
        type="primary"
        style="width:100%;margin-bottom:30px;"
        @click.native.prevent="handleLogin"
      >登录</el-button>

      <div style="position:relative">
        <div class="tips">
          <span>管理员用户: admin</span>
          <span>密码: 111111</span>
        </div>
        <div class="tips">
          <span style="margin-right:18px;">学生用户: student</span>
          <span>密码: 111111</span>
        </div>
      </div>
    </el-form>

    <div class="foot-copyright">
      <span>Copyright © 2021 湖南软件职业技术大学 讯飞人工智能学院 版权所有</span>
    </div>
  </div>
</template>

<script>
import { validUsername } from '@/utils/validate' // 导入校验用户名函数

export default {
  name: 'Login',
  components: { },
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!validUsername(value)) { // 判断名字是否是admin或者student
        callback(new Error('请输入正确的用户名')) // 在输入框下方进行错误提示
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('密码位数不能少于6位'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: 'admin',
        password: '111111',
        remember: false
      },
      loginRules: { // 输入校验规则
        username: [{ required: true, type: 'string', message: '请输入用户名', trigger: 'blur', validator: validateUsername }],
        password: [{ required: true, type: 'string', message: '请输入密码', trigger: 'blur', validator: validatePassword }]
      },
      passwordType: 'password',
      capsTooltip: false,
      loading: false, // 点击登录按钮上显示的转圈圈
      redirect: undefined,
      otherQuery: {}
    }
  },
  watch: {
    $route: { // 监听路由变化
      handler: function(route) { // 有两个参数：oldValue, newValue，相当于路由中的to和from的概念
        const query = route.query // 获取页面跳转参数
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  created() {
    // window.addEventListener('storage', this.afterQRScan)
  },
  mounted() {
    if (this.loginForm.username === '') {
      this.$refs.username.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  destroyed() {
    // window.removeEventListener('storage', this.afterQRScan)
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => { // 调用输入框的验证方法
        if (valid) { // 验证通过
          this.loading = true
          // user/login是调用store/modules/user.js中的全局login函数 -> api/user.js/login函数
          this.$store.dispatch('user/login', this.loginForm)
            .then(() => {
              // 页面跳转到path，携带get类型参数query；也可以使用name+params发起post方式的请求
              this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
  }
}
</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg:#283443;
$light_gray:#fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style lang="scss" scoped>
$bg:#2d3a4b;
$dark_gray:#889aa4;
$light_gray:#eee;

.foot-copyright {
  position: fixed;
  bottom: 0;
  left: 50%;
  text-align: center;
  transform: translateX(-50%);

  span {
    line-height: 20px;
    text-align: center;
    color: #666;
    margin: 0px 5px;
    font-size: 14px;
  }
}

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 30px 50px 10px 50px;
    margin: 120px auto auto auto;
    overflow: hidden;
    background: rgba(252, 254, 255, 0.11);
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}
</style>
