<template>
  <div class="account-pass">
  <mu-card>
    <mu-card-title title="修改密码" subTitle=""/>
  	<mu-flexbox class="mt8" orient="vertical">
      <mu-flexbox-item class="flex-demo">
        <mu-text-field v-model="registerEmail" hintText="请输入您的注册邮箱" :errorText="errorRegisterEmail" class="textInput"/>
        <mu-raised-button :label="sendButtonText" @click="sendEmailCaptcha" class="demo-raised-button" :disabled="sendButtonActive" primary/>
      </mu-flexbox-item>
      <mu-flexbox-item class="flex-demo">
        <mu-text-field v-model="emailCaptcha" hintText="请输入邮箱验证码" :errorText="errorEmailCaptcha" class="textInput"/>
      </mu-flexbox-item>
      <mu-flexbox-item class="flex-demo">
        <mu-text-field v-model="newPassword" hintText="请输入新密码" type="password" :errorText="errorPassword" class="textInput"/>
      </mu-flexbox-item>
      <mu-flexbox-item class="flex-demo">
        <mu-text-field v-model="newPasswordRe" hintText="请确认您输入的新密码" type="password" :errorText="errorPasswordRe" class="textInput"/>
      </mu-flexbox-item>
      <mu-flexbox-item class="flex-demo">
        <mu-text-field hintText="验证码" :errorText="errorImgCaptcha" v-model="imgCaptcha" @textOverflow="captchaInputOverflow" :maxLength="6" class="captchaInput textInput"/>
        <svgCaptcha></svgCaptcha>
      </mu-flexbox-item>
      <mu-flexbox-item class="flex-demo-button">
        <mu-raised-button @click="modifyPass" class="demo-raised-button" label="完成" primary/>
      </mu-flexbox-item>
    </mu-flexbox>
  </mu-card>
  </div>
</template>

<script>
import userApi from '../../api/userApi'
import captchaApi from '../../api/captchaApi'
import svgCaptcha from '../../components/captcha/svgCaptcha.vue'
export default {
  name: 'account-pass',
  components: {
    svgCaptcha
  },
  data () {
    return {
      msg: '这是用户管理密码页面',
      registerEmail: '',
      emailCaptcha: '',
      newPassword: '',
      newPasswordRe: '',
      imgCaptcha: '',
      sendButtonText: '发送',
      errorRegisterEmail: '',
      errorEmailCaptcha: '',
      errorPassword: '',
      errorPasswordRe: '',
      errorImgCaptcha: '',
      sendButtonActive: false
    }
  },
  watch: {
    emailCaptcha: function (val) {
      var _this = this
      if (val.length == 4) {
        captchaApi.vaildEmailCaptcha(this.registerEmail, this.emailCaptcha)
            .then(function (response) {
              if (response.data.code == -1) {
               _this.errorEmailCaptcha = response.data.msg
              }
            })
      } else {
        this.errorEmailCaptcha = ''
      }
    }
  },
  methods: {
    sendEmailCaptcha () {
      let _this = this
      captchaApi.getEmailCaptcha(this.registerEmail)
        .then(function (response) {
          if (response.data.code == 1) {
            _this.sendButtonActive = true
            _this.sendButtonTextTime()
          }
          if (response.data.code == -1) {
            _this.$notify.error({
              title: '错误',
              message: '发送邮件失败'
            })
          }
        })
    },
    sendButtonTextTime () {
      let _this = this
      let time = 60
      let interval = setInterval(function () {
        if (time > 0) {
          time = time - 1
          _this.sendButtonText = time + 'S'
        } else {
          _this.sendButtonText = '发送'
          _this.sendButtonActive = false
          clearInterval(interval);
        }
      }, 1000);
    },
    captchaInputOverflow (isOverflow) {
      this.errorImgCaptcha = isOverflow ? '超过啦！！！！' : ''
    },
    modifyPass () {
      let _this = this
      if (this.registerEmail == '') {
        this.errorRegisterEmail = '请填写注册时使用的邮箱'
      } else if (this.emailCaptcha == '') {
        this.errorEmailCaptcha = '请填写邮箱验证码'
      } else if (this.newPassword == '') {
        this.errorPassword = '请填写需要更改的密码'
      } else if (this.newPasswordRe == '') {
        this.errorPasswordRe = '请确认您的新密码'
      } else if (this.imgCaptcha == '') {
        this.errorImgCaptcha = '请填写验证码'
      } else if (this.newPasswordRe != this.newPassword) {
        this.errorPasswordRe = '两次密码输入不一致'
      } else {
        var id = this.$store.state.user.userInfo.uid
        userApi.modifyUserPass(id, this.newPassword, this.imgCaptcha)
          .then(function (res) {
              if (res.data.code == -1) {
                _this.errorImgCaptcha = res.data.msg
              } else {
                _this.$notify.success({
                  title: '成功',
                  message: '修改密码成功'
                })
                _this.$router.push('/userinfo/account-info')
              }
          })
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
.account-pass
  width 100%
  height 100%
  text-align left
.textInput
  margin-left 20px
.captchaInput
  width 165px
  float left
.mt8 
  margin-top 0px
.flex-demo 
  height 50px
  line-height 32px
.flex-demo-button
  height 50px
  margin-left 40px
</style>
