<template>
  <Form ref="loginForm" :model="form" :rules="rules" @keydown.enter.native="handleSubmit" class='loginForm'>
    <FormItem prop="userName" style="margin-top:20px">
      <Input :class='focusUser?"focusInput":"blueInput"' @on-focus='focusUser=true' @on-blur='focusUser=false'  v-model="form.userName" placeholder="请输入用户名" autofocus>
        <img class='inputImg' :src="focusUser||form.userName?require('../../assets/images/loginImg/user_select.png'):require('../../assets/images/loginImg/user_normal.png')" slot="prefix" >
      </Input>
    </FormItem>
    <FormItem prop="password" style="margin-top:20px">
      <Input :type="formpswd?'text':'password'" :class='focusPswd?"focusInput":"blueInput"' :readonly='pswdReadonly' @on-focus='focusPswd=true;' @on-blur='focusPswd=false'  v-model="form.password" placeholder="请输入密码">
        <img class='inputImg' :src="focusPswd||form.password?require('../../assets/images/loginImg/password_select.png'):require('../../assets/images/loginImg/password_normal.png')" slot="prefix" >
        <img class='inputImgShow' :src="formpswd?require('../../assets/images/loginImg/showPswd.png'):require('../../assets/images/loginImg/hidePswd.png')"  @click="formpswd=!formpswd" slot="suffix">
      </Input>
    </FormItem>
    <FormItem  style="margin-top:-10px">
      <div @click="loginChange(!autoLogin)" class='autoLogin'>
        <img class='inputImg' :src="autoLogin?require('../../assets/images/loginImg/picked.png'):require('../../assets/images/loginImg/pick.png')" >
        <span>记住密码</span>
      </div>
    </FormItem>
    <FormItem style="margin-top:10px">
      <Button :class='{btnActive:form.password&&form.userName}' @click="handleSubmit('loginForm')" type="primary" long>登录</Button>
    </FormItem>
  </Form>
</template>
<script>
import CryptoJS from "crypto-js";
export default {
  name: "LoginForm",
  props: {
    userNameRules: {
      type: Array,
      default: () => {
        return [{ required: true, message: "账号不能为空", trigger: "blur" }];
      }
    },
    passwordRules: {
      type: Array,
      default: () => {
        return [{ required: true, message: "密码不能为空", trigger: "blur" }];
      }
    },
    autoLogin: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      form: {
        userName: "",
        password: ""
      },
      focusUser:false,
      focusPswd:false,
      formpswd: false,
      pswdReadonly:true,
    };
  },
  computed: {
    rules() {
      return {
        userName: this.userNameRules,
        password: this.passwordRules
      };
    }
  },
  methods: {
    //预读取cookie中用户信息
    getCookie() {
      //cookie有账号信息
      if (document.cookie.length>0) {
        var arr = document.cookie.split("; "); 
        arr.forEach(item=>{
          let arr2=item.split("=")
          if (arr2[0] == "uesrName") {
            // Decrypt，将解密后的内容赋值给账号
            var bytes = CryptoJS.AES.decrypt(arr2[1], "secretkey123");
            this.form.userName = bytes.toString(CryptoJS.enc.Utf8);
          } else if (arr2[0] == "password") {
            // Decrypt，将解密后的内容赋值给密码
            var bytes = CryptoJS.AES.decrypt(arr2[1], "secretkey123");
            this.form.password = bytes.toString(CryptoJS.enc.Utf8);
          }
        })
      }
    },
    loginChange(value) {
      this.$emit("update:autoLogin",value);
    },
    handleSubmit() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.$emit("on-success-valid", {
            userName: this.form.userName,
            password: this.form.password
          });
        }
      });
    }
  },
  mounted() {
    this.getCookie();
    this.$nextTick(()=>{
      setTimeout(()=>{
        this.pswdReadonly = false;
      },1000)
    })
  }
};
</script>
<style lang="less" scoped>
  .loginForm{
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
      >.inputImgShow{
        width: 20px;
        height: 20px;
        margin-top:3px;
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
    .btnActive{
      background:rgba(255,255,255,1);
    }
    .autoLogin{
      cursor: pointer;
      >img{
        width: 12px;
        height: 13px;
        vertical-align:-3px;
      }
      >span{
        color:rgba(255,255,255,0.6);
        margin-left: 5px;
        vertical-align:middle
      }
    }
  }
  
</style>
