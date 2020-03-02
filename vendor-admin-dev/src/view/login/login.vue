<template>
  <div class='loginIndex'>
    <img :src="require('../../assets/images/loginImg/load_img.png')" class='bigImg'>
    <!-- 登陆框 -->
    <div class="login-con">
      <Tabs @on-click="tabsChange">
        <TabPane label="密码登录" name="name1">
          <div class="form-con">
            <login-form
              ref="loginForm"
              @on-success-valid="handleSubmit"
              :autoLogin.sync="autoLogin"
            ></login-form>
            <div class="login-tip">
              <span>
                若无账号，请先
                <a href="javascript:;" @click="register">注册</a>
              </span>
            </div>
          </div>
        </TabPane>
        <TabPane label="短信登录" name="name2">
          <div class="form-con">
            <login-message ref="loginMessage" @on-message-valid="handleMessageSubmit"></login-message>
            <div class="login-tip">
              <span>
                若无账号，请先
                <a href="javascript:;" @click="register">注册</a>
              </span>
            </div>
          </div>
        </TabPane>
      </Tabs>
        <div class='ewmCode'>
          <img :src="require('../../assets/images/loginImg/load_img.png')">
          <p>下载商户APP</p>
        </div>
    </div>
    <p class='cName'>深圳市自购猫科技有限公司</p>
    <p class='beian' @click='toLink'>粤CIP备20003838号</p>
  </div>
</template>
<script>
import LoginForm from "_c/login-form";
import LoginMessage from "_c/login-message";
import { mapActions, mapMutations } from "vuex";
import { getMenuByRouter, localSave, localRead } from "@/libs/util";
import { netWorkHttp } from "@/api/data";
import { matchingRoute } from "@/router/matching";
import Cookies from "js-cookie";
import { home } from "@/router/routers.js";
import CryptoJS from "crypto-js";
export default {
  name: "login",
  components: {
    LoginForm,
    LoginMessage
  },
  data() {
    return {
      autoLogin: true, //是否记住密码
      channelId: null, //渠道id
      userId: null, //用户id
      userType: null, //用户类型
      auditStatus: "", //审核状态
    };
  },
  methods: {
    ...mapActions([
      "handleLogin",
      "handlePhoneLogin",
      "setAsyncRouter",
      "getlogo"
    ]),
    toLink(){
      window.open("http://www.beian.miit.gov.cn/state/outPortal/loginPortal.action", '_blank');
    },
    // 子组件传参的事件
    handleSubmit({ userName, password }) {
      // 调用审核账号的接口
      netWorkHttp(
        `/channelApply/queryChannelApplyByUserNameOrPhone?userName=${userName}&password=${password}`,
        null,
        "get"
      ).then(res => {
        // 渠道商信息保存到本地
        if (
          res.result !== undefined &&
          res.result.auditStatus != 4 &&
          res.result.channelId == null
        ) {
          window.sessionStorage.setItem("info", JSON.stringify(res.result));
          this.auditStatus = res.result.auditStatus;
          //展示审核通知,跳转页面
          this.$router.push({
            path: "auditSteps",
            query: { auditStatus: this.auditStatus }
          });
        } else {
          //账号登录
          this.handleLogin({ userName, password }).then(res => {
            this.setCookie("uesrName", userName, this.autoLogin);
            this.setCookie("password", password, this.autoLogin);
            this.channelId = res.data.result.userVo.channelId;
            this.userId = res.data.result.userVo.id;
            this.userType = res.data.result.userVo.type;
            this.getTreeData();
            this.getlogo(this.channelId);
            this.$router.push({
              name: this.$config.homeName
            });
          });
        }
      });
    },
    // 短信验证登录
    handleMessageSubmit({ phone, smsCode }) {
      this.handlePhoneLogin({ phone, smsCode }).then(res => {
        this.channelId = res.data.result.userVo.channelId;
        this.userId = res.data.result.userVo.id;
        this.userType = res.data.result.userVo.type;
        this.getTreeData();
        this.getlogo(this.channelId);
        this.$router.push({
          name: this.$config.homeName
        });
      }).catch((err)=>{
        if(this.isPhone())alert(err)
      });
    },
    isPhone() {
      let mobile = navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i)
      return mobile!= null
    },
    setCookie(key, value, is) {
      if (is) {
        var cipherText = CryptoJS.AES.encrypt(
          value + "",
          "secretkey123"
        ).toString();
        Cookies.set(key, cipherText, { expires: 7 });
      } else {
        Cookies.set(key, "", { expires: 7 });
      }
    },
    // 跳转注册
    register() {
      this.$router.push("register");
    },
    // 获取权限树
    getTreeData() {
      let data = {
        channelId: this.channelId,
        permissionType: 1,
        userType: this.userType,
        userId: this.userId
      };
      netWorkHttp("/permission/queryUserMenuOrPermission", data).then(res => {
        let data = res.result;
        const routers = getMenuByRouter(data);
        const originRouteNames = this.$router.options.routes.map(r => r.name);
        // // 需要解决重复加入问题
        if (
          routers &&
          routers.length &&
          originRouteNames.indexOf(routers[0].name) < 0
        ) {
          data = [home, ...data];
          const AsyncRouter = this.filterAsyncRouter(data);
          this.$router.addRoutes(AsyncRouter);
          this.setAsyncRouter(AsyncRouter);
        }
      });
    },
    // 将后台传来的字符串替换为组件
    filterAsyncRouter(asyncRouterMap) {
      const accessedRouters = asyncRouterMap.filter(route => {
        if (route.component) {
          route.component = matchingRoute(route.component); // 匹配路由
        }
        if (route.children && route.children.length) {
          route.children = this.filterAsyncRouter(route.children);
        }
        return true;
      });
      return accessedRouters;
    },
    tabsChange() {
      this.$refs.loginForm.$refs.loginForm.resetFields();
      this.$refs.loginMessage.$refs.loginMessage.resetFields();
      this.$refs.loginMessage.clearTimeID();
    }
  }
};
</script>
<style lang="less" scoped>
  .loginIndex{
    background-image: linear-gradient(to top left, #54AAFF, #0048D0);
    height: 100%;
    .login-con{
      position: absolute;
      top:50%;
      right:18%;
      transform: translateY(-50%);
      .ivu-tabs{
        background: transparent;
        width: 250px;
        border:none;
        /deep/.ivu-tabs-bar{
          border-bottom:none;
          margin-bottom:25px;
          .ivu-tabs-nav{
            color:rgba(255,255,255,0.5);
            font-size: 13px;
          }
        }
        /deep/ .ivu-tabs-nav .ivu-tabs-tab-active{
          color:white;
          font-size: 16px;
        }
        /deep/.ivu-tabs-ink-bar{
          background: white
        }
      }
      .login-tip{
        text-align: right;
        margin-top: -25px;
        >span{
          color:rgba(255,255,255,0.3);
          font-size: 12px;
          >a{
            color:rgba(255,255,255,0.6);
          }
        }
      }
      .ewmCode{
        margin-top: 25px;
        >img{
          width: 57px;
          height: 57px;
          vertical-align: top;
        }
        >p{
          font-size: 12px;
          color:rgba(255,255,255,1);
          margin-top: 5px;
        }
      }
    }
    .bigImg{
      position: absolute;
      top:50%;
      left:15%;
      transform: translateY(-50%);
      width: 490px;
    }
    .cName,.beian{
      position: fixed;
      left: 50%;
      bottom: 50px;
      color:rgba(255,255,255,1);
      transform: translateX(-50%)
    }
    .beian{
      bottom: 20px;
      cursor: pointer;
      &:hover{
        text-decoration: underline;
      }
    }
  }
</style>
