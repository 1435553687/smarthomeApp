<template>
  <div>
  <div class="header">
    <div v-if="isLogin">
      <div class="header-title">登录界面</div>
      <div class="header-info"></div>
    </div>
    <div v-else>
      <div class="header-title">注册界面</div>
      <div class="header-info"></div>
    </div>
  </div>
  <div class="body">
    <div class="login-form">
      <van-field placeholder="请输入账号" :value="inputUserName" @change="onUserNameChange"/>
      <van-field type="password" placeholder="请输入密码" :value="inputPassWord" @change="onPassWordChange"/>
      <div v-if="!isLogin">
        <van-field placeholder="请输入手机号" :value="inputContact" @change="onContactChange"/>
      </div>
    </div>
    <van-button slot = "button" round block color="#ff7500" @click="onClick">{{isLogin?'登录':'新用户注册'}}</van-button>
    <div class="other-option">
      <div @click="onOptionClick">
        <span >{{isLogin?'新用户注册':'登录'}}</span>
      </div>
      <span style="margin:0 70px"></span>
      <div @click="onForgetClick"><span>忘记密码</span></div>
    </div>
    <div class="copyright-wrapper">
      <span class="copyright"></span>
    </div>
    <van-dialog
      use-slot
      title="找回密码界面"
      :show="showFindPW"
      show-cancel-button
      transition ="fade"
      @confirm = "onFindPWConfirm"
      @cancel= "onFindPWCancel"
    >
      <van-field 
      label="手机号"
      title-width="60px"
      placeholder="请输入绑定的手机号" :value="inputContact" @change="onContactChange"/>
    </van-dialog>
      <van-dialog
        use-slot
        title="重置密码"
        :show="showResetPW"
        show-cancel-button
        transition="fade"
        @confirm="onResetPWConfirm"
        @cancel="onResetPWCancel">
        <van-field 
        label="账号" 
        title-width="60px" 
        :value="inputUserName" 
        readonly 
        />
        <van-field
        label="新密码"
        type="password"
        title-width="60px"
        placeholder="请输入新密码"
        :value="inputPassWord"
        @change="onPassWordChange"
        />
      </van-dialog>
    <van-toast id="van-toast" />
  </div>
  </div>
</template>

<script>
import Toast from "vant-weapp/dist/toast/toast"
export default {


  data () {
    return {
      isLogin:true,
      inputUserName:"",
      inputPassWord:"",
      inputContact:"",
      showFindPW:false,
      showResetPW:false
    }
  },
  methods:{
    onUserNameChange(event){
      var that =this
      that.$set(that,"inputUserName",event.mp.detail)
      console.log("账号：",event.mp.detail)
    },
    onPassWordChange(event){
      var that =this
      that.$set(that,"inputPassWord",event.mp.detail)
      console.log("密码：",event.mp.detail)
    },
   onContactChange(event){
      var that =this
      that.$set(that,"inputContact",event.mp.detail)
      console.log("手机号：",event.mp.detail)
    },
    onClick(event){
      var that =this
      Toast.loading({
        duration:0,//持续展示Toast
        forbidClick:true,//禁止背景能够被点击
        message:that.isLogin?'登录中...':'注册中...'
      })
      //模拟服务器延时1000ms
      setTimeout(()=>{//箭头函数
        if(!that.isLogin){
          wx.setStorage({
          key:"user",
          data:{
            username:that.inputUserName,
            password:that.inputPassWord,
            contect:that.inputContact
          },
            success (res) {
            console.log(res);
            console.log('注册成功！')
            Toast.success('注册成功')
            },
            fail (res) {
            console.log(res)
            console.log('注册失败！')
            Toast.fail('注册失败')
            }
        });
        }
        else{
          wx.getStorage({
          key: 'user',
          success (res) {
            console.log(res.data)
            if(that.inputUserName == res.data.username &&that.inputPassWord == res.data.password)
            {
              Toast.success('登录成功')
              //500ms跳转到首页
              setTimeout(()=>{
                wx.switchTab({
                url: '/pages/index/main'
              })
              },500)
            }else{
              Toast.fail('账号或密码错误')
            }
          },
          fail (res) {
            console.log(res);
            Toast.fail('没有此用户')
          }
        });
        }
      },1000)
    },
    onOptionClick(event){
      var that = this
      that.isLogin = !that.isLogin
      console.log(`切换注册/登陆按钮被点击了！当前处于${that.isLogin ? "登陆" : "注册"}状态！`
      );
    },
    onForgetClick(event){
      var that = this;
      that.showFindPW = true

    },
    onFindPWConfirm(event){
      var that =this;
      wx.getStorage({
      key: 'user',
      success (res) {
      console.log(res.data)
      if(that.inputContact == res.data.contect)
      {
        that.inputUserName = res.data.username
        that.showResetPW = true
        console.log("找到用户：",res.data.username)
      }else{
        Toast.fail("无该用户信息");
        that.inputContact = ""
        that.showFindPW = false
      }
      },
      fail (res) {
      console.log(res);
      Toast.fail('没有此用户')
      }
      });

    },
    onFindPWCancel(event) {
    var that = this;
    that.showFindPW = false
    that.inputContect = "";
    },
    onResetPWConfirm(event) {
      var that = this;
      wx.setStorage({
        key: "user",
        data: {
          username: that.inputUserName,
          password: that.inputPassWord,
          contect: that.inputContact
        },
        success(res) {
          console.log(res);
          console.log("密码重置成功！");
          Toast.success("密码重置成功");
        },
        fail(res) {
          console.log(res);
          console.log("密码重置失败！");
          Toast.success("密码重置失败");
        }
      });
    },
    onResetPWCancel(event) {
      var that = this;
      that.showResetPW = false
      that.inputUserName = "";
      that.inputPassWord = "";
      that.inputContact = "";
    }
  }
};
</script>

<style lang="scss" scoped>
.header{
 height: 100px;
 padding: 25px ;
 text-align: center;
 background-color: #ff7500;
 .header-title{
   font-size: 28px;
   font-weight: 500;
   color: #fff;
   //margin-left: 20px;
 }
 .header-info{
   font-size: 14px;
   color: #fff;
   //margin-left: 20px;
 }
}
.body{
  padding: 20px;
  margin-top: -20px;
  border-radius: 10px 10px 0 0;
  background-color: #fff;
  .login-form{
    margin-bottom: 30px;
  }
  .other-option{
    display: flex;
    font-size: 18px;
    justify-content: center;
    margin-top: 20px;
    color: #000000;
  }
  .copyright-wrapper{
    display: flex;
    justify-content: center;
    .copyright{
    color:#9f9f9f;
    position: fixed;
    bottom: 40px;
    }
  }
}
</style>
