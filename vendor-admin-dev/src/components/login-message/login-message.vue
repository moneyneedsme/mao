<template>
  <div id="loginMessage">
    <Form ref="loginMessage" :model="form" :rules="rules" @keydown.enter.native="handleSubmit">
      <FormItem prop="phone" style="margin-top:20px">
        <Input :maxlength="11"  :class='focusUser?"focusInput":"blueInput"' @on-focus='focusUser=true' @on-blur='focusUser=false'  v-model="form.phone" placeholder="请输入手机号码" autofocus>
          <img class='inputImg' :src="focusUser||form.phone?require('../../assets/images/loginImg/phone2.png'):require('../../assets/images/loginImg/phone1.png')" slot="prefix" >
        </Input>
      </FormItem>
      <FormItem prop="smsCode">
        <Button
          class="yzmBtn"
          :class='{"BtnActive":form.phone}'
          type="primary"
          :disabled="!canClick"
          @click="countDown"
        >
          {{content}}
        </Button>
        <Input :class='focusPswd?"focusInput":"blueInput"' @on-focus='focusPswd=true' @on-blur='focusPswd=false'  v-model="form.smsCode" placeholder="请输入短信验证码">
          <img class='inputImg' :src="focusPswd||form.smsCode?require('../../assets/images/loginImg/password_select.png'):require('../../assets/images/loginImg/password_normal.png')" slot="prefix" >
        </Input>
      </FormItem>
      <FormItem style="margin-top:40px">
        <Button :class='{BtnActive:form.smsCode&&form.phone}' @click="handleSubmit('loginMessage')" type="primary" long>登录</Button>
      </FormItem>
    </Form>
  </div>
</template>
<script>
import { getsmsCode } from "@/api/user";
export default {
  name: "LoginMessage",
  data() {
    const validatePhone = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("输入不能为空"));
      } else if (!/^1[3456789]\d{9}$/.test(value)) {
        callback(new Error("电话号码格式错误"));
      } else {
        callback();
      }
    };
    return {
      form: {
        phone: "",
        smsCode: ""
      },
      focusUser:false,
      focusPswd:false,
      rules: {
        phone: [{ required: true, validator: validatePhone, trigger: "blur" }],
        smsCode: [
          {
            required: true,
            message: "输入不能为空",
            trigger: "blur"
          }
        ]
      },
      content: "发送验证码", // 按钮里显示的内容
      totalTime: 60, // 记录具体倒计时时间
      canClick: true, // 添加canClick
      clock:null
    };
  },
  methods: {
    handleSubmit() {
      this.$refs.loginMessage.validate(valid => {
        if (valid) {
          this.$emit("on-message-valid", {
            phone: this.form.phone,
            smsCode: this.form.smsCode
          });
        }
      });
    },

    countDown() {
      getsmsCode(this.form.phone).then(res => {
        if (res.data.code == 200) {
          if (!this.canClick) return;
          this.canClick = false;
          this.content = this.totalTime + "s后重新发送";
          this.clock = window.setInterval(() => {
            this.totalTime--;
            this.content = this.totalTime + "s后重新发送";
            if (this.totalTime < 0) {
              window.clearInterval(this.clock);
              this.content = "重新发送验证码";
              this.totalTime = 60;
              this.canClick = true; // 这里重新开启
            }
          }, 1000);
        }
      });
    },
    clearTimeID(){
      window.clearInterval(this.clock);
      this.content="发送验证码";
      this.canClick=true;
    }
  }
};
</script>
<style lang="less" scoped>
  #loginMessage{
    /deep/.ivu-input-wrapper{
      width: 100%!important;
      overflow: hidden;
      .ivu-input{
        background: transparent;
        border-width:0 0 1px 0; 
        border-radius: 0px;
      }
    }
    .focusInput{
      /deep/.ivu-input{
        border-color:rgba(255,255,255,1);
        color:white;
        &::placeholder{
          color:rgba(255,255,255,1);
        }
      }
    }
    .blueInput{
      /deep/.ivu-input{
        border-color:rgba(255,255,255,0.5);
        color:white;
        &::placeholder{
          color:rgba(255,255,255,0.5);
        }
      }
    }
    /deep/ .ivu-input-prefix, .ivu-input-suffix{
      width: 20px;
      top:4px;
      >.inputImg{
        width: 20px;
        height: 20px;
        vertical-align: middle
      }
    }
    /deep/.ivu-form-item-content{
      overflow: hidden;
      padding-bottom: 20px;
      margin-top: -20px!important
    }
    /deep/.ivu-form-item-error-tip{
      top:inherit;
      bottom: 0px;
    }
    /deep/ .ivu-btn{
      border-radius:14px;
      background:rgba(255,255,255,0.5);
      font-size:12px;
      color:rgba(32,110,226,1);
      &:hover{
      background:rgba(255,255,255,1);
      }
    }
    .yzmBtn{
      position: absolute;
      right: 0px;
      top:0px;
      width:89px;
      height:25px;
      line-height:25px;
      padding:0px;
      border-radius:2px;
      background:rgba(255,255,255,0.2);
      font-size: 12px;
      color:rgba(255,255,255,0.3);
      border-radius:4px;
      z-index: 999;
      &:hover{
        background:rgba(255,255,255,1);
        color:rgba(32,110,226,1);
      }
    }
    .BtnActive{
      background:rgba(255,255,255,1);
      color:rgba(32,110,226,1);
    }
  }
</style>
